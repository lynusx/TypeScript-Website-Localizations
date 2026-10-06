---
title: 高级类型
layout: docs
permalink: /zh/docs/handbook/advanced-types.html
oneline: TypeScript 中关于类型的高级概念
deprecated_by: /docs/handbook/2/types-from-types.html

# prettier-ignore
deprecation_redirects: [
  type-guards-and-differentiating-types, /docs/handbook/2/narrowing.html,
  user-defined-type-guards, /docs/handbook/2/narrowing.html#using-type-predicates,
  typeof-type-guards, "/docs/handbook/2/narrowing.html#typeof-type-guards",
  instanceof-type-guards, /docs/handbook/2/narrowing.html#instanceof-narrowing,
  nullable-types, /docs/handbook/2/everyday-types.html#null-and-undefined,
  type-aliases, /docs/handbook/2/everyday-types.html#type-aliases,
  interfaces-vs-type-aliases, /docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces,
  enum-member-types, /docs/handbook/enums.html,
  polymorphic-this-types, /docs/handbook/2/classes.html,
  index-types, /docs/handbook/2/objects.html#index-signatures,
  index-types-and-index-signatures, /docs/handbook/2/indexed-access-types.html,
  mapped-types, /docs/handbook/2/mapped-types.html,
  inference-from-mapped-types, /docs/handbook/2/mapped-types.html,
  conditional-types, /docs/handbook/2/conditional-types.html,
  distributive-conditional-types, /docs/handbook/2/conditional-types.html#distributive-conditional-types,
  type-inference-in-conditional-types, /docs/handbook/2/conditional-types.html#inferring-within-conditional-types,
  predefined-conditional-types, /docs/handbook/utility-types.html,
  using-the-in-operator, "/docs/handbook/2/narrowing.html#the-in-operator-narrowing",
  using-type-predicates, "/docs/handbook/2/narrowing.html#using-type-predicates"
]
---

本页面列举了一些更高级的类型建模方式。它与 [Utility Types](/docs/handbook/utility-types.html) 文档相辅相成，后者涵盖了 TypeScript 中内置且全局可用的实用工具类型。

## 类型守卫与类型区分

当值可能属于重叠的多种类型时，联合类型在为这类场景建模时非常有用。
但当我们明确需要知道当前拥有的是不是 `Fish` 时，该怎么办？
在 JavaScript 中，区分两个可能值的常用做法是检查某个成员是否存在。
正如前文所述，你只能访问那些保证存在于联合类型所有组成成员中的成员。

```ts twoslash
// @errors: 2339
type Fish = { swim: () => void }
type Bird = { fly: () => void }
declare function getSmallPet(): Fish | Bird
// ---cut---
let pet = getSmallPet()

// You can use the 'in' operator to check
if ('swim' in pet) {
  pet.swim()
}
// However, you cannot use property access
if (pet.fly) {
  pet.fly()
}
```

为了让上述代码通过属性访问器正常工作，我们需要使用类型断言：

```ts twoslash
type Fish = { swim: () => void }
type Bird = { fly: () => void }
declare function getSmallPet(): Fish | Bird
// ---cut---
let pet = getSmallPet()
let fishPet = pet as Fish
let birdPet = pet as Bird

if (fishPet.swim) {
  fishPet.swim()
} else if (birdPet.fly) {
  birdPet.fly()
}
```

然而，这并不是你希望在代码库中看到的代码风格。

## 用户自定义类型守卫

如果在执行检查后，我们就能在每个分支中直接获知 `pet` 的确切类型，那就好多了。

恰好，TypeScript 提供了名为*类型守卫*（type guard）的机制。
类型守卫是一种在运行时执行检查的表达式，用于确保在某个作用域内的类型安全。

### 使用类型谓词

要定义一个类型守卫，我们只需要定义一个返回值类型为*类型谓词*（type predicate）的函数：

```ts twoslash
type Fish = { swim: () => void }
type Bird = { fly: () => void }
declare function getSmallPet(): Fish | Bird
// ---cut---
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined
}
```

在这个例子中，`pet is Fish` 就是我们的类型谓词。
谓词的形式为 `parameterName is Type`，其中 `parameterName` 必须是当前函数签名中的参数名。

每当使用某个变量调用 `isFish` 时，如果原始类型与目标类型兼容，TypeScript 就会将该变量*收窄*（narrow）为具体的类型。

```ts twoslash
type Fish = { swim: () => void }
type Bird = { fly: () => void }
declare function getSmallPet(): Fish | Bird
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined
}
// ---cut---
// Both calls to 'swim' and 'fly' are now okay.
let pet = getSmallPet()

if (isFish(pet)) {
  pet.swim()
} else {
  pet.fly()
}
```

请注意，TypeScript 不仅知道在 `if` 分支中 `pet` 是 `Fish`；
它还知道在 `else` 分支中它*不是* `Fish`，因此必定是 `Bird`。

你还可以使用类型守卫 `isFish` 来过滤 `Fish | Bird` 类型的数组，从而获取 `Fish` 数组：

```ts twoslash
// @errors: 2345
type Fish = { swim: () => void }
type Bird = { fly: () => void }
declare function getSmallPet(): Fish | Bird
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined
}
// ---cut---
const zoo: (Fish | Bird)[] = [getSmallPet(), getSmallPet(), getSmallPet()]
const underWater1: Fish[] = zoo.filter(isFish)
// or, equivalently
const underWater2: Fish[] = zoo.filter<Fish>(isFish)
const underWater3: Fish[] = zoo.filter<Fish>((pet) => isFish(pet))
```

### 使用 `in` 操作符

`in` 操作符同样可以用作收窄类型的表达式。

对于形如 `n in x` 的表达式（其中 `n` 是字符串字面量或字符串字面量类型，`x` 是联合类型），在为 true 的分支中，类型会收窄为具有可选或必选属性 `n` 的类型；而在为 false 的分支中，类型会收窄为具有可选或缺失属性 `n` 的类型。

```ts twoslash
type Fish = { swim: () => void }
type Bird = { fly: () => void }
// ---cut---
function move(pet: Fish | Bird) {
  if ('swim' in pet) {
    return pet.swim()
  }
  return pet.fly()
}
```

## `typeof` 类型守卫

我们回过头来，使用联合类型编写一个 `padLeft` 的版本。
我们可以使用类型谓词将其编写如下：

```ts twoslash
function isNumber(x: any): x is number {
  return typeof x === 'number'
}

function isString(x: any): x is string {
  return typeof x === 'string'
}

function padLeft(value: string, padding: string | number) {
  if (isNumber(padding)) {
    return Array(padding + 1).join(' ') + value
  }
  if (isString(padding)) {
    return padding + value
  }
  throw new Error(`Expected string or number, got '${padding}'.`)
}
```

然而，仅仅为了判断一个类型是否为原始类型而专门定义一个函数未免过于麻烦。
幸运的是，你不需要将 `typeof x === "number"` 抽象为单独的函数，因为 TypeScript 本身就能将其识别为一个类型守卫。
这意味着我们可以直接内联编写这些检查。

```ts twoslash
function padLeft(value: string, padding: string | number) {
  if (typeof padding === 'number') {
    return Array(padding + 1).join(' ') + value
  }
  if (typeof padding === 'string') {
    return padding + value
  }
  throw new Error(`Expected string or number, got '${padding}'.`)
}
```

这类 _`typeof` 类型守卫_ 可以被识别为两种不同的形式：`typeof v === "typename"` 和 `typeof v !== "typename"`，其中 `"typename"` 可以是 [`typeof` 操作符的返回值](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof#Description) 之一（`"undefined"`、`"number"`、`"string"`、`"boolean"`、`"bigint"`、`"symbol"`、`"object"` 或 `"function"`）。
虽然 TypeScript 不会阻止你将其与其他字符串进行比较，但编译器不会将那些表达式识别为类型守卫。

## `instanceof` 类型守卫

如果你阅读过 `typeof` 类型守卫，并且熟悉 JavaScript 中的 `instanceof` 操作符，那么大概就能猜到本节要探讨的内容了。

_`instanceof` 类型守卫_ 是一种利用构造函数来收窄类型的方式。
例如，我们借用之前那个工业级强度的字符串填充器示例：

```ts twoslash
interface Padder {
  getPaddingString(): string
}

class SpaceRepeatingPadder implements Padder {
  constructor(private numSpaces: number) {}
  getPaddingString() {
    return Array(this.numSpaces + 1).join(' ')
  }
}

class StringPadder implements Padder {
  constructor(private value: string) {}
  getPaddingString() {
    return this.value
  }
}

function getRandomPadder() {
  return Math.random() < 0.5
    ? new SpaceRepeatingPadder(4)
    : new StringPadder('  ')
}

let padder: Padder = getRandomPadder()
//       ^?

if (padder instanceof SpaceRepeatingPadder) {
  padder
  //   ^?
}
if (padder instanceof StringPadder) {
  padder
  //   ^?
}
```

`instanceof` 的右侧必须是一个构造函数，TypeScript 将按如下顺序依次进行类型收窄：

1. 如果该函数的 `prototype` 属性类型不是 `any`，则收窄为其 `prototype` 属性的类型
2. 由该类型的构造签名所返回类型的联合类型

## 可空类型

TypeScript 拥有两个特殊的类型：`null` 和 `undefined`，它们分别对应值 null 和 undefined。
我们在[基础类型](/docs/handbook/basic-types.html)章节中曾简要介绍过它们。

默认情况下，类型检查器认为 `null` 和 `undefined` 可以赋值给任何类型。
实际上，`null` 和 `undefined` 是每种类型的有效值。
这意味着你无法*阻止*它们被赋值给任何类型，即便你希望加以防范。
`null` 的发明者 Tony Hoare 将其称为他[“价值十亿美元的错误”](https://wikipedia.org/wiki/Null_pointer#History)。

[`strictNullChecks`](/tsconfig#strictNullChecks) 标志修复了这个问题：当声明一个变量时，它不会自动包含 `null` 或 `undefined`。
你可以通过联合类型显式地将它们包含进来：

```ts twoslash
// @errors: 2322
let exampleString = 'foo'
exampleString = null

let stringOrNull: string | null = 'bar'
stringOrNull = null

stringOrNull = undefined
```

注意，为了契合 JavaScript 语义，TypeScript 会对 `null` 和 `undefined` 作区分处理。
`string | null` 与 `string | undefined` 以及 `string | undefined | null` 分别属于不同的类型。

从 TypeScript 3.7 开始，你可以使用[可选链](/docs/handbook/release-notes/typescript-3-7.html#optional-chaining)（optional chaining）来简化可空类型的处理。

### 可选参数与属性

在启用了 [`strictNullChecks`](/tsconfig#strictNullChecks) 的情况下，可选参数会自动添加 `| undefined`：

```ts twoslash
// @errors: 2345
function f(x: number, y?: number) {
  return x + (y ?? 0)
}

f(1, 2)
f(1)
f(1, undefined)
f(1, null)
```

可选属性同样如此：

```ts twoslash
// @strict: false
// @strictNullChecks: true
// @errors: 2322
class C {
  a: number
  b?: number
}

let c = new C()

c.a = 12
c.a = undefined
c.b = 13
c.b = undefined
c.b = null
```

### 类型守卫与类型断言

由于可空类型是通过联合类型实现的，因此你需要使用类型守卫来去除 `null`。
幸运的是，这与你在 JavaScript 中编写的代码完全相同：

```ts twoslash
function f(stringOrNull: string | null): string {
  if (stringOrNull === null) {
    return 'default'
  } else {
    return stringOrNull
  }
}
```

这里的 `null` 排除逻辑非常显而易见，但你也可以使用更简短的操作符：

```ts twoslash
function f(stringOrNull: string | null): string {
  return stringOrNull ?? 'default'
}
```

当编译器无法自动消除 `null` 或 `undefined` 时，你可以使用类型断言操作符来手动移除它们。
其语法为后置 `!`：`identifier!` 会从 `identifier` 的类型中移除 `null` 和 `undefined`：

```ts twoslash
// @errors: 2532 18048
function getUser(id: string): UserAccount | undefined {
  return {} as any
}
// ---cut---
interface UserAccount {
  id: number
  email?: string
}

const user = getUser('admin')
user.id

if (user) {
  user.email.length
}

// Instead if you are sure that these objects or fields exist, the
// postfix ! lets you short circuit the nullability
user!.email!.length
```

## 类型别名

类型别名会为类型创建一个新名称。
类型别名有时与接口类似，但它可以为原始类型、联合类型、元组以及任何其他需要手动编写的类型进行命名。

```ts twoslash
type Second = number

let timeInSecond: number = 10
let time: Second = 10
```

定义别名并不会真正创建一个新类型——它只是创建了一个引用该类型的新*名称*。
为原始类型定义别名并没有太大实际用处，不过可以用作一种代码文档形式。

与接口一样，类型别名也可以是泛型的——我们只需添加类型参数，并在别名声明的右侧使用它们：

```ts
type Container<T> = { value: T }
```

我们还可以让类型别名在属性中引用自身：

```ts
type Tree<T> = {
  value: T
  left?: Tree<T>
  right?: Tree<T>
}
```

结合[交叉类型](/docs/handbook/unions-and-intersections.html)，我们还可以构建出一些非常精妙复杂的类型：

```ts twoslash
declare function getDriversLicenseQueue(): LinkedList<Person>
// ---cut---
type LinkedList<Type> = Type & { next: LinkedList<Type> }

interface Person {
  name: string
}

let people = getDriversLicenseQueue()
people.name
people.next.name
people.next.next.name
people.next.next.next.name
//                  ^?
```

## 接口与类型别名

如前所述，类型别名的作用与接口十分相似；然而，它们之间仍存在一些细微差别。

`interface` 的绝大多数特性在 `type` 中均可使用，二者的核心区别在于：类型别名创建后无法再次打开以添加新属性，而接口始终是可扩展的。

<div class='table-container'>
<table class='full-width-table'>
  <tbody>
    <tr>
      <th><code>Interface</code></th>
      <th><code>Type</code></th>
    </tr>
    <tr>
      <td>
        <p>扩展接口</p>
        <code><pre>
interface Animal {
  name: string
}<br/>
interface Bear extends Animal {
  honey: boolean
}<br/>
const bear = getBear() 
bear.name
bear.honey
        </pre></code>
      </td>
      <td>
        <p>通过交叉类型扩展类型别名</p>
        <code><pre>
type Animal = {
  name: string
}<br/>
type Bear = Animal & { 
  honey: Boolean 
}<br/>
const bear = getBear();
bear.name;
bear.honey;
        </pre></code>
      </td>
    </tr>
    <tr>
      <td>
        <p>向已有接口添加新字段</p>
        <code><pre>
interface Window {
  title: string
}<br/>
interface Window {
  ts: import("typescript")
}<br/>
const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
        </pre></code>
      </td>
      <td>
        <p>类型别名一旦创建便无法更改</p>
        <code><pre>
type Window = {
  title: string
}<br/>
type Window = {
  ts: import("typescript")
}<br/>
// Error: Duplicate identifier 'Window'.<br/>
        </pre></code>
      </td>
    </tr>
    </tbody>
</table>
</div>

由于接口[对扩展开放](https://wikipedia.org/wiki/Open/closed_principle)的特性更贴合 JavaScript 对象的运作方式，因此在可行的情况下，我们建议优先使用接口而非类型别名。

另一方面，如果你无法使用接口来表达某种类型形状，并且需要使用联合类型或元组类型，那么类型别名通常是更好的选择。

## 枚举成员类型

正如在[枚举章节](./enums.html#union-enums-and-enum-member-types)中所述，当枚举的每个成员都通过字面量初始化时，枚举成员便拥有各自独立的类型。

在大多数情况下，当我们提到“单例类型”（singleton types）时，通常指的既是枚举成员类型，也是数值/字符串字面量类型；不过许多用户会把“单例类型”和“字面量类型”混用互称。

## 多态 `this` 类型

多态 `this` 类型表示其包含类或接口的*子类型*（subtype）。
这被称为 _F_ 有界多态（_F_-bounded polymorphism），许多人更熟悉将其称为 [fluent API](https://en.wikipedia.org/wiki/Fluent_interface) 模式。
例如，这使得具有层级继承关系的流式接口表达起来更加容易。
以一个在每次操作后都返回 `this` 的简单计算器为例：

```ts twoslash
class BasicCalculator {
  public constructor(protected value: number = 0) {}
  public currentValue(): number {
    return this.value
  }
  public add(operand: number): this {
    this.value += operand
    return this
  }
  public multiply(operand: number): this {
    this.value *= operand
    return this
  }
  // ... other operations go here ...
}

let v = new BasicCalculator(2).multiply(5).add(1).currentValue()
```

由于该类使用了 `this` 类型，你可以对它进行继承扩展，派生出的新类无需任何修改就能直接沿用原有的方法：

```ts twoslash
class BasicCalculator {
  public constructor(protected value: number = 0) {}
  public currentValue(): number {
    return this.value
  }
  public add(operand: number): this {
    this.value += operand
    return this
  }
  public multiply(operand: number): this {
    this.value *= operand
    return this
  }
  // ... other operations go here ...
}
// ---cut---
class ScientificCalculator extends BasicCalculator {
  public constructor(value = 0) {
    super(value)
  }
  public sin() {
    this.value = Math.sin(this.value)
    return this
  }
  // ... other operations go here ...
}

let v = new ScientificCalculator(2).multiply(5).sin().add(1).currentValue()
```

如果没有 `this` 类型，`ScientificCalculator` 将无法在继承 `BasicCalculator` 的同时保持流式接口。
此时 `multiply` 会返回 `BasicCalculator`，而它并没有 `sin` 方法。
然而，在使用 `this` 类型后，`multiply` 返回的是 `this`，在此处其类型正是 `ScientificCalculator`。

## 索引类型

借助索引类型，你可以让编译器对使用动态属性名的代码进行类型检查。
例如，在 JavaScript 中有一种常见模式是从对象中选取属性子集：

```js
function pluck(o, propertyNames) {
  return propertyNames.map((n) => o[n])
}
```

以下是在 TypeScript 中使用**索引类型查询**（index type query）和**索引访问**（indexed access）操作符来编写与使用该函数的方式：

```ts twoslash
function pluck<T, K extends keyof T>(o: T, propertyNames: K[]): T[K][] {
  return propertyNames.map((n) => o[n])
}

interface Car {
  manufacturer: string
  model: string
  year: number
}

let taxi: Car = {
  manufacturer: 'Toyota',
  model: 'Camry',
  year: 2014,
}

// Manufacturer and model are both of type string,
// so we can pluck them both into a typed string array
let makeAndModel: string[] = pluck(taxi, ['manufacturer', 'model'])

// If we try to pluck model and year, we get an
// array of a union type: (string | number)[]
let modelYear = pluck(taxi, ['model', 'year'])
```

编译器会检查 `manufacturer` 和 `model` 是否确实为 `Car` 上的属性。
上面的示例引入了两个全新的类型操作符。
首先是 `keyof T`，即**索引类型查询操作符**。
对于任何类型 `T`，`keyof T` 是 `T` 的已知公共属性名组成的联合类型。
例如：

```ts twoslash
interface Car {
  manufacturer: string
  model: string
  year: number
}
// ---cut---
let carProps: keyof Car
//         ^?
```

`keyof Car` 与 `"manufacturer" | "model" | "year"` 完全可以互换。
两者的区别在于，如果你向 `Car` 中添加另一个属性，比如 `ownersAddress: string`，那么 `keyof Car` 会自动更新为 `"manufacturer" | "model" | "year" | "ownersAddress"`。
此外，你还可以在像 `pluck` 这样的泛型上下文中使用 `keyof`，因为在这些场景中你无法提前预知具体的属性名。
这意味着编译器会校验你传入 `pluck` 的属性名集合是否正确：

```ts
// error, Type '"unknown"' is not assignable to type '"manufacturer" | "model" | "year"'
pluck(taxi, ['year', 'unknown'])
```

第二个操作符是 `T[K]`，即**索引访问操作符**。
在这里，类型语法反映了表达式语法。
这意味着 `taxi["manufacturer"]` 的类型为 `Car["manufacturer"]` &mdash; 在本例中就是 `string`。
然而，正如索引类型查询一样，你也可以在泛型上下文中使用 `T[K]`，而这正是其真正威力所在。
你只需要确保类型变量满足 `K extends keyof T` 即可。
下面是另一个名为 `getProperty` 的函数示例：

```ts
function getProperty<T, K extends keyof T>(o: T, propertyName: K): T[K] {
  return o[propertyName] // o[propertyName] is of type T[K]
}
```

在 `getProperty` 中，由于 `o: T` 且 `propertyName: K`，这意味着 `o[propertyName]: T[K]`。
一旦返回 `T[K]` 的计算结果，编译器就会实例化该键的实际类型，因此 `getProperty` 的返回值类型会根据你所请求的属性不同而动态变化。

```ts twoslash
// @errors: 2345
function getProperty<T, K extends keyof T>(o: T, propertyName: K): T[K] {
  return o[propertyName] // o[propertyName] is of type T[K]
}
interface Car {
  manufacturer: string
  model: string
  year: number
}
let taxi: Car = {
  manufacturer: 'Toyota',
  model: 'Camry',
  year: 2014,
}
// ---cut---
let manufacturer: string = getProperty(taxi, 'manufacturer')
let year: number = getProperty(taxi, 'year')

let unknown = getProperty(taxi, 'unknown')
```

## 索引类型与索引签名

`keyof` 与 `T[K]` 会与索引签名相互作用。索引签名的参数类型必须是 'string' 或 'number'。
如果一个类型带有字符串索引签名，那么 `keyof T` 将是 `string | number`
（而不仅仅是 `string`，因为在 JavaScript 中，访问对象属性既可以使用字符串（`object["42"]`），也可以使用数字（`object[42]`））。
并且 `T[string]` 就是该索引签名的类型：

```ts twoslash
interface Dictionary<T> {
  [key: string]: T
}
let keys: keyof Dictionary<number>
//     ^?
let value: Dictionary<number>['foo']
//      ^?
```

如果一个类型带有数字索引签名，那么 `keyof T` 将仅为 `number`。

```ts twoslash
// @errors: 2339
interface Dictionary<T> {
  [key: number]: T
}

let keys: keyof Dictionary<number>
//     ^?
let numberValue: Dictionary<number>[42]
//     ^?
let value: Dictionary<number>['foo']
```

## 映射类型

一个常见的需求是基于某个已有类型，将其所有属性都变为可选属性：

```ts
interface PersonSubset {
  name?: string
  age?: number
}
```

或者我们可能需要一个只读版本：

```ts
interface PersonReadonly {
  readonly name: string
  readonly age: number
}
```

这种情况在 JavaScript 中非常普遍，因此 TypeScript 提供了一种基于旧类型创建新类型的方式 &mdash; **映射类型**。
在映射类型中，新类型以相同的方式转换旧类型中的每一个属性。
例如，你可以将所有属性都设为可选，或者设为 `readonly` 类型。
以下是几个示例：

```ts twoslash
type Partial<T> = {
  [P in keyof T]?: T[P]
}

// @noErrors
type Readonly<T> = {
  readonly [P in keyof T]: T[P]
}
```

使用方式如下：

```ts twoslash
type Person = {
  name: string
  age: number
}
// ---cut---
type PersonPartial = Partial<Person>
//   ^?
type ReadonlyPerson = Readonly<Person>
//   ^?
```

请注意，这种语法描述的是整个类型而非单个成员。
如果你希望添加新成员，可以使用交叉类型：

```ts twoslash
// @errors: 2693 1005 1128 7061
// Use this:
type PartialWithNewMember<T> = {
  [P in keyof T]?: T[P];
} & { newMember: boolean }

// This is an error!
type WrongPartialWithNewMember<T> = {
  [P in keyof T]?: T[P];
  newMember: boolean;
}
```

让我们来看一下最简单的映射类型及其组成部分：

```ts twoslash
type Keys = 'option1' | 'option2'
type Flags = { [K in Keys]: boolean }
```

该语法与内部带有 `for .. in` 的索引签名语法十分相似。
它包含三个部分：

1. 类型变量 `K`，它会依次绑定到每个属性名。
2. 字符串字面量联合类型 `Keys`，包含要遍历的属性名集合。
3. 属性的目标类型。

在这个简单示例中，`Keys` 是硬编码的属性名列表，而属性类型始终是 `boolean`，因此该映射类型等价于：

```ts twoslash
type Flags = {
  option1: boolean
  option2: boolean
}
```

然而在实际应用中，映射类型通常类似于前文提到的 `Readonly` 或 `Partial`。
它们基于某个已有类型，并以某种方式对这些属性进行转换。
这正是 `keyof` 与索引访问类型发挥作用的地方：

```ts twoslash
type Person = {
  name: string
  age: number
}
// ---cut---
type NullablePerson = { [P in keyof Person]: Person[P] | null }
//   ^?
type PartialPerson = { [P in keyof Person]?: Person[P] }
//   ^?
```

但定义一个通用泛型版本会更加实用：

```ts
type Nullable<T> = { [P in keyof T]: T[P] | null }
type Partial<T> = { [P in keyof T]?: T[P] }
```

在这些示例中，属性列表为 `keyof T`，而得到的目标类型则是 `T[P]` 的某种变体。
这是通用的映射类型模板。
因为这种转换是[同态的](https://wikipedia.org/wiki/Homomorphism)，这意味着该映射仅作用于 `T` 的属性，而不会引入其他属性。
编译器知道在添加任何新修饰符之前，可以完整保留并复制所有现有的属性修饰符。
例如，如果 `Person.name` 原本是只读的，那么 `Partial<Person>.name` 将会同时是只读和可选的。

再来看一个示例，其中 `T[P]` 被包装在 `Proxy<T>` 类中：

```ts twoslash
// @noErrors
type Proxy<T> = {
  get(): T
  set(value: T): void
}

type Proxify<T> = {
  [P in keyof T]: Proxy<T[P]>
}

function proxify<T>(o: T): Proxify<T> {
  // ... wrap proxies ...
}

let props = { rooms: 4 }
let proxyProps = proxify(props)
//  ^?
```

由于 `Readonly<T>` 和 `Partial<T>` 非常实用，它们与 `Pick` 以及 `Record` 一起被直接收录在 TypeScript 的标准库中：

```ts
type Pick<T, K extends keyof T> = {
  [P in K]: T[P]
}

type Record<K extends keyof any, T> = {
  [P in K]: T
}
```

`Readonly`、`Partial` 和 `Pick` 是同态的，而 `Record` 则不是。
`Record` 不是同态的一个线索是，它并没有接收一个用于从中复制属性的输入类型：

```ts twoslash
type ThreeStringProps = Record<'prop1' | 'prop2' | 'prop3', string>
```

非同态类型本质上是在创建新属性，因此它们无法从任何地方复制已有的属性修饰符。

需要注意的是，`keyof any` 表示可以用作对象索引的任意值的类型。换句话说，`keyof any` 目前等同于 `string | number | symbol`。

## 由映射类型进行推断

既然已经掌握了如何对类型的属性进行包装，接下来很自然的一步就是将它们解包。
幸运的是，这非常简单：

```ts twoslash
type Proxy<T> = {
  get(): T
  set(value: T): void
}

type Proxify<T> = {
  [P in keyof T]: Proxy<T[P]>
}

function proxify<T>(o: T): Proxify<T> {
  return {} as any
}

let props = { rooms: 4 }
let proxyProps = proxify(props)
// ---cut---
function unproxify<T>(t: Proxify<T>): T {
  let result = {} as T
  for (const k in t) {
    result[k] = t[k].get()
  }
  return result
}

let originalProps = unproxify(proxyProps)
//  ^?
```

需要注意的是，这种解包推断仅适用于同态映射类型。
如果映射类型不是同态的，你必须为解包函数显式指定类型参数。

## 条件类型

条件类型会根据由类型关系测试所表达的条件，在两种可能的类型中选择其一：

```ts
T extends U ? X : Y
```

上述类型表示：当 `T` 可以赋值给 `U` 时，其类型为 `X`，否则为 `Y`。

条件类型 `T extends U ? X : Y` 要么被直接*解析（resolved）*为 `X` 或 `Y`，要么由于该条件依赖于一个或多个类型变量而被*延迟（deferred）*解析。
当 `T` 或 `U` 包含类型变量时，究竟是解析为 `X` 或 `Y`，还是进行延迟解析，取决于类型系统是否有足够的信息来断定 `T` 始终可以赋值给 `U`。

作为会被立即解析的类型示例，我们可以看下面这段代码：

```ts twoslash
declare function f<T extends boolean>(x: T): T extends true ? string : number

// Type is 'string | number'
let x = f(Math.random() < 0.5)
//  ^?
```

另一个示例是使用了嵌套条件类型的 `TypeName` 类型别名：

```ts twoslash
type TypeName<T> = T extends string
  ? 'string'
  : T extends number
    ? 'number'
    : T extends boolean
      ? 'boolean'
      : T extends undefined
        ? 'undefined'
        : T extends Function
          ? 'function'
          : 'object'

type T0 = TypeName<string>
//   ^?
type T1 = TypeName<'a'>
//   ^?
type T2 = TypeName<true>
//   ^?
type T3 = TypeName<() => void>
//   ^?
type T4 = TypeName<string[]>
//   ^?
```

而对于条件类型被延迟解析（即保持原样而未立即选取某一分支）的情况，可以参考如下示例：

```ts twoslash
interface Foo {
  propA: boolean
  propB: boolean
}

declare function f<T>(x: T): T extends Foo ? string : number

function foo<U>(x: U) {
  // Has type 'U extends Foo ? string : number'
  let a = f(x)

  // This assignment is allowed though!
  let b: string | number = a
}
```

在上面的代码中，变量 `a` 的类型是一个尚未选取具体分支的条件类型。
当其他代码调用 `foo` 时，会用具体类型替换 `U`，此时 TypeScript 将重新计算该条件类型，以决定是否能确定具体的分支。

与此同时，只要条件类型的每个分支都能赋值给某个目标类型，我们就可以将该条件类型赋值给该目标类型。
因此在上面的示例中，我们可以将 `U extends Foo ? string : number` 赋值给 `string | number`，因为无论该条件最终的计算结果是什么，它必定是 `string` 或 `number` 之一。

## 分布式条件类型

如果被检查的类型是裸类型参数（naked type parameter），那么此类条件类型被称为*分布式条件类型（distributive conditional types）*。
分布式条件类型在实例化期间会自动分发到联合类型上。
例如，若用类型实参 `A | B | C` 实例化 `T extends U ? X : Y` 中的 `T`，结果会被解析为 `(A extends U ? X : Y) | (B extends U ? X : Y) | (C extends U ? X : Y)`。

#### 示例

```ts twoslash
type TypeName<T> = T extends string
  ? 'string'
  : T extends number
    ? 'number'
    : T extends boolean
      ? 'boolean'
      : T extends undefined
        ? 'undefined'
        : T extends Function
          ? 'function'
          : 'object'
// ---cut---
type T5 = TypeName<string | (() => void)>
//   ^?
type T6 = TypeName<string | string[] | undefined>
//   ^?
type T7 = TypeName<string[] | number[]>
//   ^?
```

在分布式条件类型 `T extends U ? X : Y` 的实例化中，条件类型内部对 `T` 的引用会被解析为联合类型的各个独立组成成员（即在条件类型分发到联合类型*之后*，`T` 指代这些独立的成员）。
此外，在 `X` 分支中对 `T` 的引用还具有额外的类型参数约束 `U`（即在 `X` 中，`T` 被视作可以赋值给 `U`）。

#### 示例

```ts twoslash
type BoxedValue<T> = { value: T }
type BoxedArray<T> = { array: T[] }
type Boxed<T> = T extends any[] ? BoxedArray<T[number]> : BoxedValue<T>

type T1 = Boxed<string>
//   ^?
type T2 = Boxed<number[]>
//   ^?
type T3 = Boxed<string | number[]>
//   ^?
```

注意在 `Boxed<T>` 的真值（true）分支中，`T` 具有额外的约束 `any[]`，因此可以直接使用 `T[number]` 来引用数组的元素类型。另外，请注意在最后一个示例中，条件类型是如何分发到联合类型上的。

利用条件类型的分发特性，可以很方便地对联合类型进行*过滤（filter）*：

```ts twoslash
// @errors: 2300 2322
// Remove types from T that are assignable to U
type Diff<T, U> = T extends U ? never : T
// Remove types from T that are not assignable to U
type Filter<T, U> = T extends U ? T : never

type T1 = Diff<'a' | 'b' | 'c' | 'd', 'a' | 'c' | 'f'>
//   ^?
type T2 = Filter<'a' | 'b' | 'c' | 'd', 'a' | 'c' | 'f'> // "a" | "c"
//   ^?
type T3 = Diff<string | number | (() => void), Function> // string | number
//   ^?
type T4 = Filter<string | number | (() => void), Function> // () => void
//   ^?

// Remove null and undefined from T
type NotNullable<T> = Diff<T, null | undefined>

type T5 = NotNullable<string | number | undefined>
//   ^?
type T6 = NotNullable<string | string[] | null | undefined>
//   ^?

function f1<T>(x: T, y: NotNullable<T>) {
  x = y
  y = x
}

function f2<T extends string | undefined>(x: T, y: NotNullable<T>) {
  x = y
  y = x
  let s1: string = x
  let s2: string = y
}
```

当条件类型与映射类型结合使用时尤其实用：

```ts twoslash
type FunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? K : never
}[keyof T]
type FunctionProperties<T> = Pick<T, FunctionPropertyNames<T>>

type NonFunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? never : K
}[keyof T]
type NonFunctionProperties<T> = Pick<T, NonFunctionPropertyNames<T>>

interface Part {
  id: number
  name: string
  subparts: Part[]
  updatePart(newName: string): void
}

type T1 = FunctionPropertyNames<Part>
//   ^?
type T2 = NonFunctionPropertyNames<Part>
//   ^?
type T3 = FunctionProperties<Part>
//   ^?
type T4 = NonFunctionProperties<Part>
//   ^?
```

注意，条件类型不允许递归引用自身。例如，以下写法会报错。

#### 示例

```ts twoslash
// @errors: 2456 2315
type ElementType<T> = T extends any[] ? ElementType<T[number]> : T // Error
```

## 条件类型中的类型推断

在条件类型的 `extends` 子句中，可以使用 `infer` 声明来引入一个待推断的类型变量。
这些被推断的类型变量可以在条件类型的真值分支中被引用。
同一个类型变量可以出现在多个 `infer` 位置上。

例如，以下代码提取了一个函数类型的返回值类型：

```ts twoslash
// @noErrors
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any
```

条件类型可以嵌套使用，以形成按顺序求值的模式匹配序列：

```ts twoslash
type Unpacked<T> = T extends (infer U)[]
  ? U
  : T extends (...args: any[]) => infer U
    ? U
    : T extends Promise<infer U>
      ? U
      : T

type T0 = Unpacked<string>
//   ^?
type T1 = Unpacked<string[]>
//   ^?
type T2 = Unpacked<() => string>
//   ^?
type T3 = Unpacked<Promise<string>>
//   ^?
type T4 = Unpacked<Promise<string>[]>
//   ^?
type T5 = Unpacked<Unpacked<Promise<string>[]>>
//   ^?
```

以下示例演示了：当同一个类型变量在协变（co-variant）位置存在多个候选类型时，推断出的结果为联合类型：

```ts twoslash
type Foo<T> = T extends { a: infer U; b: infer U } ? U : never

type T1 = Foo<{ a: string; b: string }>
//   ^?
type T2 = Foo<{ a: string; b: number }>
//   ^?
```

同样地，当同一个类型变量在逆变（contra-variant）位置存在多个候选类型时，推断出的结果为交叉类型：

```ts twoslash
type Bar<T> = T extends { a: (x: infer U) => void; b: (x: infer U) => void }
  ? U
  : never

type T1 = Bar<{ a: (x: string) => void; b: (x: string) => void }>
//   ^?
type T2 = Bar<{ a: (x: string) => void; b: (x: number) => void }>
//   ^?
```

当从具有多个调用签名的类型（例如重载函数的类型）进行推断时，推断会基于*最后一个*签名进行（这通常是最宽松的兜底分支）。
无法根据参数类型列表来进行重载解析。

```ts twoslash
declare function foo(x: string): number
declare function foo(x: number): string
declare function foo(x: string | number): string | number

type T1 = ReturnType<typeof foo>
//   ^?
```

不能在常规类型参数的约束子句中使用 `infer` 声明：

```ts twoslash
// @errors: 1338 2304
type ReturnedType<T extends (...args: any[]) => infer R> = R
```

不过，可以通过抹去约束中的类型变量，转而指定条件类型来达到大致相同的效果：

```ts twoslash
// @noErrors
type AnyFunction = (...args: any[]) => any
type ReturnType<T extends AnyFunction> = T extends (...args: any[]) => infer R
  ? R
  : any
```

## 预定义的条件类型

TypeScript 添加了若干预定义的条件类型，你可以在 [Utility Types](/docs/handbook/utility-types.html) 中查看完整列表及示例。
