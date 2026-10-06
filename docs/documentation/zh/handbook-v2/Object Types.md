---
title: 对象类型
layout: docs
permalink: /zh/docs/handbook/2/objects.html
oneline: 'TypeScript 如何描述 JavaScript 对象的形状。'
---

在 JavaScript 中，我们将数据组织和传递的基本方式是通过对象。
而在 TypeScript 中，我们通过*对象类型*（object types）来表示它们。

正如前面所见，对象类型可以是匿名的：

```ts twoslash
function greet(person: { name: string; age: number }) {
  //                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  return 'Hello ' + person.name
}
```

也可以通过接口来命名：

```ts twoslash
interface Person {
  //      ^^^^^^
  name: string
  age: number
}

function greet(person: Person) {
  return 'Hello ' + person.name
}
```

或者使用类型别名：

```ts twoslash
type Person = {
  // ^^^^^^
  name: string
  age: number
}

function greet(person: Person) {
  return 'Hello ' + person.name
}
```

在上述三个示例中，我们编写的函数都接收包含 `name` 属性（必须是 `string`）和 `age` 属性（必须是 `number`）的对象。

## 快速参考

如果你想快速浏览日常开发中的重要语法，我们提供了关于 [`type` 和 `interface`](https://www.typescriptlang.org/cheatsheets) 的速查表。

## 属性修饰符

对象类型中的每个属性都可以指定若干特性：属性的类型、属性是否可选，以及属性是否可写。

### 可选属性

很多时候，我们会遇到*可能*设置了某个属性的对象。
在这些情况下，我们可以在属性名末尾添加问号（`?`），将其标记为*可选*（optional）属性。

```ts twoslash
interface Shape {}
declare function getShape(): Shape

// ---cut---
interface PaintOptions {
  shape: Shape
  xPos?: number
  //  ^
  yPos?: number
  //  ^
}

function paintShape(opts: PaintOptions) {
  // ...
}

const shape = getShape()
paintShape({ shape })
paintShape({ shape, xPos: 100 })
paintShape({ shape, yPos: 100 })
paintShape({ shape, xPos: 100, yPos: 100 })
```

在这个示例中，`xPos` 和 `yPos` 都被视为可选属性。
我们可以选择传入其中的任意一个，因此上面对 `paintShape` 的每次调用都是合法的。
可选性真正表达的含义是：如果该属性*确实*被设置了，那么它最好符合指定的类型。

我们也可以读取这些属性——但在启用了 [`strictNullChecks`](/tsconfig#strictNullChecks) 时，TypeScript 会提示它们的值可能是 `undefined`。

```ts twoslash
interface Shape {}
declare function getShape(): Shape

interface PaintOptions {
  shape: Shape
  xPos?: number
  yPos?: number
}

// ---cut---
function paintShape(opts: PaintOptions) {
  let xPos = opts.xPos
  //              ^?
  let yPos = opts.yPos
  //              ^?
  // ...
}
```

在 JavaScript 中，即使某个属性从未被设置过，我们依然可以访问它——只是会得到 `undefined` 值。
我们可以通过显式检查 `undefined` 来专门处理这种情况。

```ts twoslash
interface Shape {}
declare function getShape(): Shape

interface PaintOptions {
  shape: Shape
  xPos?: number
  yPos?: number
}

// ---cut---
function paintShape(opts: PaintOptions) {
  let xPos = opts.xPos === undefined ? 0 : opts.xPos
  //  ^?
  let yPos = opts.yPos === undefined ? 0 : opts.yPos
  //  ^?
  // ...
}
```

注意，为未指定的值设置默认值这种模式非常普遍，因此 JavaScript 提供了专门的语法来支持它。

```ts twoslash
interface Shape {}
declare function getShape(): Shape

interface PaintOptions {
  shape: Shape
  xPos?: number
  yPos?: number
}

// ---cut---
function paintShape({ shape, xPos = 0, yPos = 0 }: PaintOptions) {
  console.log('x coordinate at', xPos)
  //                             ^?
  console.log('y coordinate at', yPos)
  //                             ^?
  // ...
}
```

这里我们在 `paintShape` 的参数中使用了[解构模式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)，并为 `xPos` 和 `yPos` 提供了[默认值](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Default_values)。
现在，`xPos` 和 `yPos` 在 `paintShape` 的函数体内必定存在，但对于 `paintShape` 的任何调用方而言仍然是可选的。

> 请注意，目前无法在解构模式内部直接添加类型注解。
> 这是因为在 JavaScript 中，以下语法已经具有完全不同的含义。
>
> ```ts twoslash
> // @noImplicitAny: false
> // @errors: 2552 2304
> interface Shape {}
> declare function render(x: unknown)
> // ---cut---
> function draw({ shape: Shape, xPos: number = 100 /*...*/ }) {
>   render(shape)
>   render(xPos)
> }
> ```
>
> 在对象解构模式中，`shape: Shape` 的含义是“提取属性 `shape` 并将其在局部重新定义为名为 `Shape` 的变量”。
> 同样，`xPos: number` 会创建一个名为 `number` 的变量，其值取决于参数的 `xPos`。

### `readonly` 属性

在 TypeScript 中，属性还可以被标记为 `readonly`（只读）。
虽然这不会改变运行时的任何行为，但在类型检查期间，标记为 `readonly` 的属性无法被写入。

```ts twoslash
// @errors: 2540
interface SomeType {
  readonly prop: string
}

function doSomething(obj: SomeType) {
  // We can read from 'obj.prop'.
  console.log(`prop has the value '${obj.prop}'.`)

  // But we can't re-assign it.
  obj.prop = 'hello'
}
```

使用 `readonly` 修饰符并不一定意味着值是完全不可变的——换句话说，并不代表其内部内容不能被更改。
它仅表示该属性本身不能被重新赋值。

```ts twoslash
// @errors: 2540
interface Home {
  readonly resident: { name: string; age: number }
}

function visitForBirthday(home: Home) {
  // We can read and update properties from 'home.resident'.
  console.log(`Happy birthday ${home.resident.name}!`)
  home.resident.age++
}

function evict(home: Home) {
  // But we can't write to the 'resident' property itself on a 'Home'.
  home.resident = {
    name: 'Victor the Evictor',
    age: 42,
  }
}
```

合理理解 `readonly` 的内涵非常重要。
在开发阶段，它有助于向 TypeScript 表明该对象应当如何被使用的意图。
但在检查两个类型是否兼容时，TypeScript 并不会考虑这两个类型上的属性是否为 `readonly`，因此 `readonly` 属性也可能通过别名引用发生改变。

```ts twoslash
interface Person {
  name: string
  age: number
}

interface ReadonlyPerson {
  readonly name: string
  readonly age: number
}

let writablePerson: Person = {
  name: 'Person McPersonface',
  age: 42,
}

// works
let readonlyPerson: ReadonlyPerson = writablePerson

console.log(readonlyPerson.age) // prints '42'
writablePerson.age++
console.log(readonlyPerson.age) // prints '43'
```

通过使用[映射修饰符](/docs/handbook/2/mapped-types.html#mapping-modifiers)，你可以移除 `readonly` 特性。

### 索引签名

有时你无法提前预知某个类型所有属性的名称，但你清楚这些属性值的形状。

在这些情况下，你可以使用索引签名（index signature）来描述可能出现的值的类型，例如：

```ts twoslash
declare function getStringArray(): StringArray
// ---cut---
interface StringArray {
  [index: number]: string
}

const myArray: StringArray = getStringArray()
const secondItem = myArray[1]
//     ^?
```

在上面，我们定义了一个带有索引签名的 `StringArray` 接口。
该索引签名表明：当使用 `number` 对 `StringArray` 进行索引时，它将返回一个 `string`。

索引签名属性只允许使用某些特定类型：`string`、`number`、`symbol`、模板字符串模式，以及仅由这些类型组成的联合类型。

<details>
    <summary>支持多种类型的索引器……</summary>
    <p>同时支持多种类型的索引器是可行的。请注意，当同时使用 <code>number</code> 和 <code>string</code> 索引器时，数值索引器返回的类型必须是字符串索引器返回类型的子类型。这是因为当使用 <code>number</code> 进行索引时，JavaScript 实际上会在索引对象之前将其转换为 <code>string</code>。这意味着使用 <code>100</code>（<code>number</code>）进行索引与使用 <code>"100"</code>（<code>string</code>）进行索引是完全相同的，因此两者需要保持一致。</p>
```ts twoslash
// @errors: 2413
// @strictPropertyInitialization: false
interface Animal {
  name: string
}

interface Dog extends Animal {
breed: string
}

// Error: indexing with a numeric string might get you a completely separate type of Animal!
interface NotOkay {
[x: number]: Animal
[x: string]: Dog
}

````

</details>

虽然字符串索引签名是描述“字典”模式的强大方式，但它们也会强制要求所有属性都要与它们的返回类型相匹配。
这是因为字符串索引声明了 `obj.property` 同样可以通过 `obj["property"]` 进行访问。
在以下示例中，`name` 的类型与字符串索引的类型不匹配，类型检查器会报错：
```ts twoslash
// @errors: 2411
// @errors: 2411
interface NumberDictionary {
  [index: string]: number

  length: number // ok
  name: string
}
````

然而，如果索引签名是属性类型的联合类型，那么不同类型的属性是可以接受的：

```ts twoslash
interface NumberOrStringDictionary {
  [index: string]: number | string
  length: number // ok, length is a number
  name: string // ok, name is a string
}
```

最后，你可以将索引签名设置为 `readonly`，以防止对其索引进行赋值：

```ts twoslash
declare function getReadOnlyStringArray(): ReadonlyStringArray
// ---cut---
// @errors: 2542
interface ReadonlyStringArray {
  readonly [index: number]: string
}

let myArray: ReadonlyStringArray = getReadOnlyStringArray()
myArray[2] = 'Mallory'
```

你无法设置 `myArray[2]`，因为该索引签名是 `readonly` 的。

## 额外属性检查

对象在何处以及如何被赋予类型，在类型系统中会产生不同的效果。
其中一个关键示例就是额外属性检查（excess property checking），当对象被创建并在创建过程中赋值给对象类型时，该检查会对对象进行更彻底的校验。

```ts twoslash
// @errors: 2345 2739
interface SquareConfig {
  color?: string
  width?: number
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  return {
    color: config.color || 'red',
    area: config.width ? config.width * config.width : 20,
  }
}

let mySquare = createSquare({ colour: 'red', width: 100 })
```

注意，传入 `createSquare` 的实参拼写是 _`colour`_ 而非 `color`。
在原生 JavaScript 中，这类问题通常会静默失败。

你可能会认为这段程序的类型检查应该通过，因为 `width` 属性是兼容的，没有提供 `color` 属性，而多余的 `colour` 属性也无关紧要。

然而，TypeScript 认为这段代码中很可能存在 bug。
对象字面量在被赋值给其他变量或作为实参传递时，会受到特殊对待并经历*额外属性检查*。
如果对象字面量包含“目标类型”所没有的任何属性，你就会收到一个错误：

```ts twoslash
// @errors: 2345 2739
interface SquareConfig {
  color?: string
  width?: number
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  return {
    color: config.color || 'red',
    area: config.width ? config.width * config.width : 20,
  }
}
// ---cut---
let mySquare = createSquare({ colour: 'red', width: 100 })
```

绕过这些检查实际上非常简单。
最简便的方法就是使用类型断言：

```ts twoslash
// @errors: 2345 2739
interface SquareConfig {
  color?: string
  width?: number
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  return {
    color: config.color || 'red',
    area: config.width ? config.width * config.width : 20,
  }
}
// ---cut---
let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig)
```

不过，如果你确定该对象可以拥有某些以特殊方式使用的额外属性，更好的方法可能是添加一个字符串索引签名。
如果 `SquareConfig` 可以拥有上述类型的 `color` 和 `width` 属性，但*同时*也可以拥有任意数量的其他属性，我们可以这样定义它：

```ts twoslash
interface SquareConfig {
  color?: string
  width?: number
  [propName: string]: unknown
}
```

这里我们表达的是，`SquareConfig` 可以拥有任意数量的属性，只要它们不是 `color` 或 `width`，它们的类型并不重要。

最后一种绕过这些检查的方法可能有些出人意料，那就是将对象赋值给另一个变量：
由于将对象赋值给 `squareOptions` 时不会经历额外属性检查，因此编译器不会报错：

```ts twoslash
interface SquareConfig {
  color?: string
  width?: number
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  return {
    color: config.color || 'red',
    area: config.width ? config.width * config.width : 20,
  }
}
// ---cut---
let squareOptions = { colour: 'red', width: 100 }
let mySquare = createSquare(squareOptions)
```

上述变通方法之所以有效，前提是 `squareOptions` 和 `SquareConfig` 之间存在公共属性。
在此示例中是 `width` 属性。但是，如果该变量不包含任何公共的对象属性，则仍然会报错。例如：

```ts twoslash
// @errors: 2559
interface SquareConfig {
  color?: string
  width?: number
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  return {
    color: config.color || 'red',
    area: config.width ? config.width * config.width : 20,
  }
}
// ---cut---
let squareOptions = { colour: 'red' }
let mySquare = createSquare(squareOptions)
```

请记住，对于像上面这样的简单代码，通常不应该试图“绕过”这些检查。
对于拥有方法并保持状态的更复杂的对象字面量，你可能需要牢记这些技巧，但绝大多数额外属性检查错误实际上都是 bug。

这意味着，如果你在选项包（option bags）等场景中遇到了额外属性检查问题，你可能需要修改某些类型声明。
在当前情况下，如果允许向 `createSquare` 传递同时包含 `color` 或 `colour` 属性的对象，你应该修改 `SquareConfig` 的定义来明确反映这一点。

## 扩展类型

在日常开发中，某些类型经常会是其他类型的更具体版本。
例如，我们可能有一个 `BasicAddress` 类型，用于描述在美国寄送信件和包裹所需的必要字段。

```ts twoslash
interface BasicAddress {
  name?: string
  street: string
  city: string
  country: string
  postalCode: string
}
```

在某些情况下这已经足够了，但如果某个地址所在的建筑物有多个单元，地址通常还会关联一个单元号。
接着我们可以描述一个 `AddressWithUnit`。

<!-- prettier-ignore -->
```ts twoslash
interface AddressWithUnit {
  name?: string;
  unit: string;
//^^^^^^^^^^^^^
  street: string;
  city: string;
  country: string;
  postalCode: string;
}
```

这样写能够满足需求，但缺点在于，尽管我们的修改仅仅是新增字段，却不得不重复书写来自 `BasicAddress` 的所有其他字段。
相反，我们可以扩展原有的 `BasicAddress` 类型，只添加属于 `AddressWithUnit` 的新增字段。

```ts twoslash
interface BasicAddress {
  name?: string
  street: string
  city: string
  country: string
  postalCode: string
}

interface AddressWithUnit extends BasicAddress {
  unit: string
}
```

在 `interface` 上使用 `extends` 关键字，使我们能够有效地从其他命名类型中复制成员，并随意添加新的成员。
这有助于减少编写类型声明时的样板代码，并明确表明同一属性的若干不同声明之间存在关联。
例如，`AddressWithUnit` 无需重复声明 `street` 属性，并且由于 `street` 源自 `BasicAddress`，读者就会明白这两个类型在某种程度上是相关的。

`interface` 还可以同时继承自多个类型。

```ts twoslash
interface Colorful {
  color: string
}

interface Circle {
  radius: number
}

interface ColorfulCircle extends Colorful, Circle {}

const cc: ColorfulCircle = {
  color: 'red',
  radius: 42,
}
```

## 交叉类型

`interface` 允许我们通过扩展已有类型来构建新类型。
TypeScript 还提供了另一种称为*交叉类型*（intersection types）的构造，主要用于组合现有的对象类型。

交叉类型使用 `&` 运算符进行定义。

```ts twoslash
interface Colorful {
  color: string
}
interface Circle {
  radius: number
}

type ColorfulCircle = Colorful & Circle
```

在这里，我们将 `Colorful` 与 `Circle` 进行了交叉，生成了一个同时拥有 `Colorful` _和_ `Circle` 中所有成员的新类型。

```ts twoslash
// @errors: 2345
interface Colorful {
  color: string
}
interface Circle {
  radius: number
}
// ---cut---
function draw(circle: Colorful & Circle) {
  console.log(`Color was ${circle.color}`)
  console.log(`Radius was ${circle.radius}`)
}

// okay
draw({ color: 'blue', radius: 42 })

// oops
draw({ color: 'red', raidus: 42 })
```

## 接口扩展与交叉类型

我们刚刚了解了两种组合类型的途径，它们看似相似，但实际上存在微妙的差异。
对于接口，我们可以使用 `extends` 子句来继承其他类型；而对于交叉类型，我们也可以实现类似的效果，并使用类型别名为结果命名。
两者之间的主要区别在于冲突的处理方式，这一差异通常也是在接口和交叉类型别名之间做出选择的核心原因之一。

如果定义了同名接口，只要属性相互兼容，TypeScript 就会尝试将它们合并。如果属性不兼容（即属性名相同但类型不同），TypeScript 则会抛出错误。

对于交叉类型，类型不同的属性会自动合并。随后使用该类型时，TypeScript 会要求该属性同时满足这两种类型，这可能会产生出人意料的结果。

例如，以下代码会抛出错误，因为属性之间不兼容：

```ts
interface Person {
  name: string
}

interface Person {
  name: number
}
```

相比之下，以下代码能够编译通过，但其结果是一个 `never` 类型：

```ts twoslash
interface Person1 {
  name: string
}

interface Person2 {
  name: number
}

type Staff = Person1 & Person2

declare const staffer: Staff
staffer.name
//       ^?
```

在这种情况下，`Staff` 要求 `name` 属性同时是 `string` 和 `number`，从而导致该属性的类型变成了 `never`。

## 泛型对象类型

设想一个可以包含任意值的 `Box` 类型——无论是 `string`、`number`、`Giraffe`，还是其他任何类型。

```ts twoslash
interface Box {
  contents: any
}
```

目前，`contents` 属性的类型被指定为 `any`，这虽然可行，但可能在后续开发中引发隐患。

我们可以改用 `unknown`，但这意味着在已经确切知道 `contents` 类型的情况下，我们必须进行预防性检查，或者使用容易出错的类型断言。

```ts twoslash
interface Box {
  contents: unknown
}

let x: Box = {
  contents: 'hello world',
}

// we could check 'x.contents'
if (typeof x.contents === 'string') {
  console.log(x.contents.toLowerCase())
}

// or we could use a type assertion
console.log((x.contents as string).toLowerCase())
```

一种类型安全的方法是为每种 `contents` 类型分别搭建不同的 `Box` 类型。

```ts twoslash
// @errors: 2322
interface NumberBox {
  contents: number
}

interface StringBox {
  contents: string
}

interface BooleanBox {
  contents: boolean
}
```

但这意味着我们必须创建不同的函数或函数重载来操作这些类型。

```ts twoslash
interface NumberBox {
  contents: number
}

interface StringBox {
  contents: string
}

interface BooleanBox {
  contents: boolean
}
// ---cut---
function setContents(box: StringBox, newContents: string): void
function setContents(box: NumberBox, newContents: number): void
function setContents(box: BooleanBox, newContents: boolean): void
function setContents(box: { contents: any }, newContents: any) {
  box.contents = newContents
}
```

这会产生大量的样板代码。而且，我们后续可能还需要引入新的类型和重载。
这令人十分沮丧，因为所有的 box 类型和重载在本质上都是相同的。

相反，我们可以创建一个声明了*类型参数*（type parameter）的*泛型*（generic）`Box` 类型。

```ts twoslash
interface Box<Type> {
  contents: Type
}
```

你可以将其理解为：“`Type` 的 `Box` 是一个其 `contents` 类型为 `Type` 的对象”。
后续在引用 `Box` 时，我们需要提供一个*类型实参*（type argument）来替换 `Type`。

```ts twoslash
interface Box<Type> {
  contents: Type
}
// ---cut---
let box: Box<string>
```

可以将 `Box` 视为某种真实类型的模板，其中 `Type` 是一个占位符，后续会被替换为其他类型。
当 TypeScript 看到 `Box<string>` 时，它会将 `Box<Type>` 中出现的所有 `Type` 替换为 `string`，最终其工作机制类似于 `{ contents: string }`。
换句话说，`Box<string>` 与我们之前定义的 `StringBox` 效果完全相同。

```ts twoslash
interface Box<Type> {
  contents: Type
}
interface StringBox {
  contents: string
}

let boxA: Box<string> = { contents: 'hello' }
boxA.contents
//   ^?

let boxB: StringBox = { contents: 'world' }
boxB.contents
//   ^?
```

`Box` 是可复用的，因为 `Type` 可以被替换为任何类型。这意味着当我们为某种新类型需要一个 box 时，完全不需要重新声明新的 `Box` 类型（尽管如果有意愿的话也可以这么做）。

```ts twoslash
interface Box<Type> {
  contents: Type
}

interface Apple {
  // ....
}

// Same as '{ contents: Apple }'.
type AppleBox = Box<Apple>
```

这也意味着我们可以通过改用[泛型函数](/docs/handbook/2/functions.html#generic-functions)来彻底避免重载。

```ts twoslash
interface Box<Type> {
  contents: Type
}

// ---cut---
function setContents<Type>(box: Box<Type>, newContents: Type) {
  box.contents = newContents
}
```

值得注意的是，类型别名也可以是泛型的。我们原本定义的 `Box<Type>` 接口：

```ts twoslash
interface Box<Type> {
  contents: Type
}
```

也可以通过类型别名来定义：

```ts twoslash
type Box<Type> = {
  contents: Type
}
```

与接口不同，类型别名不仅可以描述对象类型，因此我们还可以使用它们来编写其他类型的泛型辅助工具：

```ts twoslash
// @errors: 2575
type OrNull<Type> = Type | null

type OneOrMany<Type> = Type | Type[]

type OneOrManyOrNull<Type> = OrNull<OneOrMany<Type>>
//   ^?

type OneOrManyOrNullStrings = OneOrManyOrNull<string>
//   ^?
```

我们稍后还会回过头来进一步讨论类型别名。

### `Array` 类型

泛型对象类型通常充当某种容器类型，其工作机制独立于所包含元素的具体类型。
数据结构以这种方式运作是非常理想的，这样它们便能够在不同的数据类型之间复用。

事实证明，我们在本手册中一直在使用这样的类型：`Array` 类型。
每当我们写出像 `number[]` 或 `string[]` 这样的类型时，它们实际上只是 `Array<number>` 和 `Array<string>` 的简写形式。

```ts twoslash
function doSomething(value: Array<string>) {
  // ...
}

let myArray: string[] = ['hello', 'world']

// either of these work!
doSomething(myArray)
doSomething(new Array('hello', 'world'))
```

与前面的 `Box` 类型类似，`Array` 本身也是一个泛型类型。

```ts twoslash
// @noLib: true
interface Number {}
interface String {}
interface Boolean {}
interface Symbol {}
// ---cut---
interface Array<Type> {
  /**
   * Gets or sets the length of the array.
   */
  length: number

  /**
   * Removes the last element from an array and returns it.
   */
  pop(): Type | undefined

  /**
   * Appends new elements to an array, and returns the new length of the array.
   */
  push(...items: Type[]): number

  // ...
}
```

现代 JavaScript 还提供了其他泛型数据结构，例如 `Map<K, V>`、`Set<T>` 和 `Promise<T>`。
这仅仅意味着由于 `Map`、`Set` 和 `Promise` 的行为特性，它们可以适用于任意类型集合。

### `ReadonlyArray` 类型

`ReadonlyArray` 是一种用于描述不应被修改的数组的特殊类型。

```ts twoslash
// @errors: 2339
function doStuff(values: ReadonlyArray<string>) {
  // We can read from 'values'...
  const copy = values.slice()
  console.log(`The first value is ${values[0]}`)

  // ...but we can't mutate 'values'.
  values.push('hello!')
}
```

与属性的 `readonly` 修饰符非常相似，它主要是一个用于表达意图的工具。
当我们看到一个返回 `ReadonlyArray` 的函数时，它表明我们完全不应该修改其内容；而当我们看到一个接收 `ReadonlyArray` 的函数时，它表明我们可以将任何数组传入该函数，而无需担心其内容会被修改。

与 `Array` 不同的是，并没有可供使用的 `ReadonlyArray` 构造函数。

```ts twoslash
// @errors: 2693
new ReadonlyArray('red', 'green', 'blue')
```

相反，我们可以将常规的 `Array` 赋值给 `ReadonlyArray`。

```ts twoslash
const roArray: ReadonlyArray<string> = ['red', 'green', 'blue']
```

正如 TypeScript 为 `Array<Type>` 提供了简写语法 `Type[]` 一样，它也为 `ReadonlyArray<Type>` 提供了简写语法 `readonly Type[]`。

```ts twoslash
// @errors: 2339
function doStuff(values: readonly string[]) {
  //                     ^^^^^^^^^^^^^^^^^
  // We can read from 'values'...
  const copy = values.slice()
  console.log(`The first value is ${values[0]}`)

  // ...but we can't mutate 'values'.
  values.push('hello!')
}
```

最后需要注意的一点是，与 `readonly` 属性修饰符不同，常规 `Array` 和 `ReadonlyArray` 之间的可赋值性并不是双向的。

```ts twoslash
// @errors: 4104
let x: readonly string[] = []
let y: string[] = []

x = y
y = x
```

### 元组类型

_元组类型_（tuple type）是另一种 `Array` 类型，它确切地知道自己包含多少个元素，以及特定位置上所包含的具体类型。

```ts twoslash
type StringNumberPair = [string, number]
//                      ^^^^^^^^^^^^^^^^
```

在这里，`StringNumberPair` 是由 `string` 和 `number` 组成的元组类型。
与 `ReadonlyArray` 类似，它在运行时没有对应体现，但对 TypeScript 而言非常重要。
在类型系统中，`StringNumberPair` 描述了索引 `0` 处为 `string`、索引 `1` 处为 `number` 的数组。

```ts twoslash
function doSomething(pair: [string, number]) {
  const a = pair[0]
  //    ^?
  const b = pair[1]
  //    ^?
  // ...
}

doSomething(['hello', 42])
```

如果我们尝试访问超出元素数量的索引，将会得到一个错误。

```ts twoslash
// @errors: 2493
function doSomething(pair: [string, number]) {
  // ...

  const c = pair[2]
}
```

我们还可以使用 JavaScript 的数组解构来[解构元组](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Array_destructuring)。

```ts twoslash
function doSomething(stringHash: [string, number]) {
  const [inputString, hash] = stringHash

  console.log(inputString)
  //          ^?

  console.log(hash)
  //          ^?
}
```

> 元组类型在高度依赖约定的 API 中非常有用，因为在这些场景中每个元素的含义都是“显而易见”的。
> 这为我们在解构时随意命名变量提供了灵活性。
> 在上面的示例中，我们可以将索引 `0` 和 `1` 处的元素命名为任何我们需要的名称。
>
> 然而，由于并非每位使用者对“显而易见”都有相同的认知，因此值得重新考虑：对于你的 API 而言，使用具有明确描述性属性名称的对象是否会是更好的选择。

除了长度检查之外，像这样简单的元组类型，等价于为特定索引声明了属性、并使用数字字面量类型声明了 `length` 的特殊 `Array` 类型。

```ts twoslash
interface StringNumberPair {
  // specialized properties
  length: 2
  0: string
  1: number

  // Other 'Array<string | number>' members...
  slice(start?: number, end?: number): Array<string | number>
}
```

你可能感兴趣的另一点是：元组可以通过书写问号（在元素类型后添加 `?`）来拥有可选属性。
可选的元组元素只能放在末尾，并且也会影响 `length` 的类型。

```ts twoslash
type Either2dOr3d = [number, number, number?]

function setCoordinate(coord: Either2dOr3d) {
  const [x, y, z] = coord
  //           ^?

  console.log(`Provided coordinates had ${coord.length} dimensions`)
  //                                            ^?
}
```

元组还可以包含剩余元素（rest elements），其必须是数组/元组类型。

```ts twoslash
type StringNumberBooleans = [string, number, ...boolean[]]
type StringBooleansNumber = [string, ...boolean[], number]
type BooleansStringNumber = [...boolean[], string, number]
```

- `StringNumberBooleans` 描述了一个前两个元素分别为 `string` 和 `number`，但后面可以跟任意数量 `boolean` 的元组。
- `StringBooleansNumber` 描述了一个首元素为 `string`，随后跟任意数量 `boolean`，并以 `number` 结尾的元组。
- `BooleansStringNumber` 描述了一个起始元素为任意数量 `boolean`，并以 `string` 和 `number` 结尾的元组。

包含剩余元素的元组没有固定的“长度”（length）——它只在不同位置上具有一组已知的确定元素。

```ts twoslash
type StringNumberBooleans = [string, number, ...boolean[]]
// ---cut---
const a: StringNumberBooleans = ['hello', 1]
const b: StringNumberBooleans = ['beautiful', 2, true]
const c: StringNumberBooleans = ['world', 3, true, false, true, false, true]
```

为什么可选元素和剩余元素很有用呢？
这是因为它们能让 TypeScript 将元组与参数列表对应起来。
元组类型可以用于[剩余形参和剩余实参](/docs/handbook/2/functions.html#rest-parameters-and-arguments)，因此以下代码：

```ts twoslash
function readButtonInput(...args: [string, number, ...boolean[]]) {
  const [name, version, ...input] = args
  // ...
}
```

基本等价于：

```ts twoslash
function readButtonInput(name: string, version: number, ...input: boolean[]) {
  // ...
}
```

当你想要通过剩余参数接收可变数量的实参，且需要满足最少元素数量的要求，同时又不想引入中间变量时，这种写法非常方便。

<!--
TODO do we need this example?

For example, imagine we need to write a function that adds up `number`s based on arguments that get passed in.
```ts twoslash
function sum(...args: number[]) {
    // ...
}
```

We might feel like it makes little sense to take any fewer than 2 elements, so we want to require callers to provide at least 2 arguments.
A first attempt might be
```ts twoslash
function foo(a: number, b: number, ...args: number[]) {
    args.unshift(a, b);

    let result = 0;
    for (const value of args) {
        result += value;
    }
    return result;
}
```

-->

### `readonly` 元组类型

关于元组类型的最后一点说明——元组类型具有 `readonly` 变体，可以通过在其前面加上 `readonly` 修饰符来指定——就像数组简写语法一样。

```ts twoslash
function doSomething(pair: readonly [string, number]) {
  //                       ^^^^^^^^^^^^^^^^^^^^^^^^^
  // ...
}
```

正如你所预料的，在 TypeScript 中不允许写入 `readonly` 元组的任何属性。

```ts twoslash
// @errors: 2540
function doSomething(pair: readonly [string, number]) {
  pair[0] = 'hello!'
}
```

在大多数代码中，元组被创建后通常不会被修改，因此尽可能将类型标注为 `readonly` 元组是一个不错的默认选择。
这一点也很重要，因为带有 `const` 断言的数组字面量会被推断为 `readonly` 元组类型。

```ts twoslash
// @errors: 2345
let point = [3, 4] as const

function distanceFromOrigin([x, y]: [number, number]) {
  return Math.sqrt(x ** 2 + y ** 2)
}

distanceFromOrigin(point)
```

在此处，`distanceFromOrigin` 虽然从未修改其元素，但它期望接收一个可变元组。
由于 `point` 的类型被推断为 `readonly [3, 4]`，它与 `[number, number]` 不兼容，因为该类型无法保证 `point` 的元素不会被修改。

<!-- ## Other Kinds of Object Members

Most of the declarations in object types:

### Method Syntax

### Call Signatures

### Construct Signatures

### Index Signatures -->
