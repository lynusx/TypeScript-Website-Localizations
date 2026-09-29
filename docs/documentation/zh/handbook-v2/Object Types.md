---
title: 对象类型
layout: docs
permalink: /zh/docs/handbook/2/objects.html
oneline: 'TypeScript 如何描述 JavaScript 对象的结构。'
---

在 JavaScript 中，对象是我们对数据进行分组和传递的基础方式。在 TypeScript 中，我们通过 _对象类型_ 来表示这些对象。

正如我们所见，对象类型可以是匿名的：

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

在上述三个例子中，我们编写的函数都接受包含 `name` 属性（必须是 `string`）和 `age` 属性（必须是 `number`）的对象作为参数。

## 快速参考

如果你想快速浏览重要的常用语法，我们为你准备了 [`type` 和 `interface`](https://www.typescriptlang.org/cheatsheets) 的速查表。

## 属性修饰符

对象类型中的每个属性都可以指定几个方面的信息：类型、该属性是否可选，以及该属性是否可写。

### 可选属性

很多时候，我们会发现自己要处理的是 _可能_ 设置了某个属性的对象。在这些情况下，我们可以在属性名末尾加上问号（`?`），将这些属性标记为 _可选的_。

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

在这个例子中，`xPos` 和 `yPos` 都被视为可选的。我们可以选择提供其中任意一个，因此上面对 `paintShape` 的每次调用都是合法的。可选性真正表达的含义只是：如果该属性 _被_ 设置了，它最好具有特定的类型。

我们也可以读取这些属性 —— 但在启用了 [`strictNullChecks`](/tsconfig#strictNullChecks) 时，TypeScript 会告诉我们它们可能是 `undefined`。

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

在 JavaScript 中，即使某个属性从未被设置过，我们仍然可以访问它 —— 只是会得到值 `undefined`。我们可以通过检查来专门处理 `undefined`。

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

请注意，这种为未指定的值设置默认值的模式太常见了，以至于 JavaScript 专门有语法来支持它。

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

这里我们对 `paintShape` 的参数使用了[解构模式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)，并为 `xPos` 和 `yPos` 提供了[默认值](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Default_values)。现在 `xPos` 和 `yPos` 在 `paintShape` 的函数体内一定存在，但对 `paintShape` 的调用者来说仍是可选的。

> 注意，目前无法在解构模式中放置类型注解。这是因为下面的语法在 JavaScript 中已经有了不同的含义。
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
> 在对象解构模式中，`shape: Shape` 的意思是"取出属性 `shape`，并在局部将其重新定义为名为 `Shape` 的变量"。同样，`xPos: number` 创建了一个名为 `number` 的变量，其值基于参数的 `xPos`。

### `readonly` 属性

属性也可以在 TypeScript 中被标记为 `readonly`。虽然它不会改变运行时的任何行为，但标记为 `readonly` 的属性在类型检查期间不能被写入。

```ts twoslash
// @errors: 2540
interface SomeType {
  readonly prop: string
}

function doSomething(obj: SomeType) {
  // 我们可以读取 'obj.prop'。
  console.log(`prop has the value '${obj.prop}'.`)

  // 但我们不能重新赋值。
  obj.prop = 'hello'
}
```

使用 `readonly` 修饰符并不一定意味着值是完全不可变的 —— 换句话说，并不意味着其内部内容不能被更改。它只意味着该属性本身不能被重新写入。

```ts twoslash
// @errors: 2540
interface Home {
  readonly resident: { name: string; age: number }
}

function visitForBirthday(home: Home) {
  // 我们可以读取和更新 'home.resident' 中的属性。
  console.log(`Happy birthday ${home.resident.name}!`)
  home.resident.age++
}

function evict(home: Home) {
  // 但我们不能直接修改 'Home' 上的 'resident' 属性本身。
  home.resident = {
    name: 'Victor the Evictor',
    age: 42,
  }
}
```

正确理解 `readonly` 的含义很重要。在开发阶段，它可以用来向 TypeScript 传达意图：对象应该被如何使用。TypeScript 在检查两个类型是否兼容时不会考虑属性上的 `readonly`，因此 `readonly` 属性也可以通过别名被改变。

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

使用 [映射修饰符](/docs/handbook/2/mapped-types.html#mapping-modifiers)，你可以移除 `readonly` 属性。

### 索引签名

有时你事先不知道一个类型所有属性的名称，但你知道值的形状。

在这些情况下，你可以使用索引签名来描述可能值的类型，例如：

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

上面我们有一个带有索引签名的 `StringArray` 接口。这个索引签名声明：当用 `number` 对 `StringArray` 进行索引时，将返回一个 `string`。

索引签名属性只允许使用某些类型：`string`、`number`、`symbol`、模板字符串模式，以及仅由这些类型组成的联合类型。

<details>
    <summary>可以支持多种类型的索引器……</summary>
    <summary>可以支持多种类型的索引器……</summary>
    <p>可以支持多种类型的索引器。注意，当同时使用 `number` 和 `string` 索引器时，数字索引器返回的类型必须是字符串索引器返回类型的子类型。这是因为当用 <code>number</code> 索引时，JavaScript 实际上会先将其转换为 <code>string</code>，然后再对对象进行索引。这意味着用 <code>100</code>（一个 <code>number</code>）索引与用 <code>"100"</code>（一个 <code>string</code>）索引是同一回事，所以两者需要保持一致。</p>

```ts twoslash
// @errors: 2413
// @strictPropertyInitialization: false
interface Animal {
  name: string
}

interface Dog extends Animal {
  breed: string
}

// 错误：使用数字字符串进行索引可能会让你得到一个完全不同类型的 Animal！
interface NotOkay {
  [x: number]: Animal
  [x: string]: Dog
}
```

</details>

虽然字符串索引签名是描述"字典"模式的强大方式，但它们也会强制所有属性匹配其返回类型。这是因为字符串索引声明了 `obj.property` 也可以作为 `obj["property"]` 来访问。在下面的例子中，`name` 的类型与字符串索引的类型不匹配，类型检查器给出了错误：

```ts twoslash
// @errors: 2411
// @errors: 2411
interface NumberDictionary {
  [index: string]: number

  length: number // ok
  name: string
}
```

不过，如果索引签名是属性类型的联合，不同类型的属性也是可以接受的：

```ts twoslash
interface NumberOrStringDictionary {
  [index: string]: number | string
  length: number // ok，length 是一个 number
  name: string // ok，name 是一个 string
}
```

最后，你可以将索引签名设为 `readonly`，以防止对其索引进行赋值：

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

你不能设置 `myArray[2]`，因为索引签名是 `readonly` 的。

## 多余属性检查

对象在何处以及如何被赋予类型，在类型系统中会产生差别。其中一个关键例子就是多余属性检查：当对象字面量在创建时被赋值给一个对象类型时，它会更彻底地校验这个对象。

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

注意传给 `createSquare` 的参数拼写成了 _`colour`_ 而不是 `color`。在普通 JavaScript 中，这类错误会悄无声息地失败。

你可能会争辩说这个程序的类型是正确的，因为 `width` 属性是兼容的，没有 `color` 属性存在，而且额外的 `colour` 属性无关紧要。

然而，TypeScript 的立场是这段代码很可能有 bug。对象字面量在赋值给其他变量或作为参数传递时会得到特殊对待，经历 _多余属性检查_。如果对象字面量具有"目标类型"没有的任何属性，你就会得到错误：

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

绕过这些检查其实很简单。最简单的方法是直接使用类型断言：

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

不过，如果你确定对象可以有一些以特殊方式使用的额外属性，更好的办法可能是添加字符串索引签名。如果 `SquareConfig` 可以有上述类型的 `color` 和 `width` 属性，但 _还_ 可以有任意数量的其他属性，那么我们可以这样定义它：

```ts twoslash
interface SquareConfig {
  color?: string
  width?: number
  [propName: string]: unknown
}
```

这里我们表示 `SquareConfig` 可以有任意数量的属性，只要它们不是 `color` 或 `width`，它们的类型就无所谓。

最后一种绕过这些检查的方法可能有点出乎意料：把对象赋给另一个变量。因为赋值 `squareOptions` 不会经历多余属性检查，编译器不会报错：

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

只要 `squareOptions` 和 `SquareConfig` 之间有共同的属性，上面的变通方法就有效。在这个例子中，共同属性是 `width`。但是，如果变量没有任何共同的对象属性，它就会失败。例如：

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

请记住，对于上面这样的简单代码，你可能不应该试图"绕过"这些检查。对于有方法并持有状态的更复杂的对象字面量，你可能需要记住这些技巧，但大多数多余属性错误实际上都是 bug。

这意味着，如果在处理选项包（option bags）等内容时遇到多余属性检查问题，你可能需要修改一些类型声明。在这个例子中，如果允许向 `createSquare` 传递带有 `color` 或 `colour` 属性的对象，你应该修正 `SquareConfig` 的定义以反映这一点。

## 扩展类型

有些类型可能是其他类型的更具体版本，这很常见。例如，我们可能有一个 `BasicAddress` 类型，描述在美国寄送信件和包裹所必需的字段。

```ts twoslash
interface BasicAddress {
  name?: string
  street: string
  city: string
  country: string
  postalCode: string
}
```

在某些情况下这就够了，但如果某个地址的建筑物有多个单元，地址通常还会关联一个单元号。这时我们可以描述一个 `AddressWithUnit`。

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

这样也能用，但缺点在于：尽管我们的改动纯粹是新增字段，却不得不重复 `BasicAddress` 中的所有其他字段。相反，我们可以扩展原始的 `BasicAddress` 类型，只添加 `AddressWithUnit` 独有的新字段。

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

`interface` 上的 `extends` 关键字允许我们有效地从其他命名类型复制成员，并添加任何我们想要的新成员。这有助于减少我们必须编写的类型声明样板代码，并传达这样的意图：同一属性的几个不同声明之间可能是有关联的。例如，`AddressWithUnit` 不需要重复 `street` 属性，而且因为 `street` 来自 `BasicAddress`，读者会知道这两个类型之间存在某种关联。

`interface` 也可以从多个类型扩展。

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

`interface` 允许我们通过扩展其他类型来构建新类型。TypeScript 还提供了另一种称为 _交叉类型_ 的结构，主要用于组合现有的对象类型。

交叉类型使用 `&` 运算符定义。

```ts twoslash
interface Colorful {
  color: string
}
interface Circle {
  radius: number
}

type ColorfulCircle = Colorful & Circle
```

在这里，我们把 `Colorful` 和 `Circle` 交叉起来，产生一个同时拥有 `Colorful` _和_ `Circle` 所有成员的新类型。

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

## 接口扩展 vs 交叉类型

我们刚刚看了两种组合类型的方式，它们相似，但实际上有微妙的区别。使用接口时，我们可以用 `extends` 子句从其他类型扩展；而使用交叉类型时我们也能做类似的事，并用类型别名为结果命名。两者的主要区别在于冲突的处理方式，而这一区别通常就是你在接口和交叉类型的类型别名之间二选一的主要原因之一。

如果定义了同名的接口，TypeScript 会在属性兼容时尝试合并它们。如果属性不兼容（即属性名相同但类型不同），TypeScript 会报错。

而对于交叉类型，类型不同的属性会被自动合并。当该类型随后被使用时，TypeScript 会期望该属性同时满足两种类型，这可能产生意想不到的结果。

例如，下面的代码会因为属性不兼容而报错：

```ts
interface Person {
  name: string
}

interface Person {
  name: number
}
```

相比之下，下面的代码可以编译，但结果是 `never` 类型：

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

在这种情况下，`Staff` 要求 `name` 属性同时是 `string` 和 `number`，这导致该属性的类型变为 `never`。

## 泛型对象类型

让我们想象一个可以包含任何值的 `Box` 类型 —— `string`、`number`、`Giraffe`，什么都行。

```ts twoslash
interface Box {
  contents: any
}
```

目前，`contents` 属性的类型是 `any`，这能用，但可能在日后引发意外。

我们也可以改用 `unknown`，但那意味着在我们已经知道 `contents` 类型的情况下，仍需做预防性检查，或者使用容易出错的类型断言。

```ts twoslash
interface Box {
  contents: unknown
}

let x: Box = {
  contents: 'hello world',
}

// 我们可以检查 'x.contents'
if (typeof x.contents === 'string') {
  console.log(x.contents.toLowerCase())
}

// 或者我们也可以使用类型断言
console.log((x.contents as string).toLowerCase())
```

一种类型安全的做法是为每种 `contents` 类型分别搭建不同的 `Box` 类型。

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

这就有大量样板代码了。而且，我们以后可能还需要引入新的类型和重载。这很令人沮丧，因为我们的 box 类型和重载实际上都是一回事。

相反，我们可以创建一个声明了 _类型参数_ 的 _泛型_ `Box` 类型。

```ts twoslash
interface Box<Type> {
  contents: Type
}
```

你可以把它读作"`Type` 的 `Box`，就是 `contents` 类型为 `Type` 的东西"。之后，当我们引用 `Box` 时，必须在 `Type` 的位置给出一个 _类型实参_。

```ts twoslash
interface Box<Type> {
  contents: Type
}
// ---cut---
let box: Box<string>
```

可以把 `Box` 看作真实类型的模板，其中 `Type` 是将被替换为其他类型的占位符。当 TypeScript 看到 `Box<string>` 时，它会把 `Box<Type>` 中的每个 `Type` 替换为 `string`，最终得到类似 `{ contents: string }` 的东西。换句话说，`Box<string>` 和我们之前的 `StringBox` 的工作方式完全相同。

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

`Box` 是可复用的，因为 `Type` 可以替换为任何东西。这意味着当我们需要为新类型准备一个盒子时，根本不需要声明新的 `Box` 类型（不过如果想这么做当然也完全可以）。

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

这也意味着我们可以通过使用[泛型函数](/docs/handbook/2/functions.html#generic-functions)来完全避免重载。

```ts twoslash
interface Box<Type> {
  contents: Type
}

// ---cut---
function setContents<Type>(box: Box<Type>, newContents: Type) {
  box.contents = newContents
}
```

值得注意的是，类型别名也可以是泛型的。我们刚才定义的新 `Box<Type>` 接口是：

```ts twoslash
interface Box<Type> {
  contents: Type
}
```

也可以改用类型别名来定义：

```ts twoslash
type Box<Type> = {
  contents: Type
}
```

由于类型别名与接口不同，不仅可以描述对象类型，我们还可以用它们编写其他种类的泛型辅助类型。

```ts twoslash
// @errors: 2575
type OrNull<Type> = Type | null

type OneOrMany<Type> = Type | Type[]

type OneOrManyOrNull<Type> = OrNull<OneOrMany<Type>>
//   ^?

type OneOrManyOrNullStrings = OneOrManyOrNull<string>
//   ^?
```

我们稍后会再回到类型别名的话题。

### `Array` 类型

泛型对象类型通常是某种容器类型，其工作方式与它们所包含元素的类型无关。数据结构以这种方式工作是理想的，这样它们就能在不同数据类型之间复用。

其实我们在本手册中一直在使用这样一个类型：`Array` 类型。每当我们写出 `number[]` 或 `string[]` 这样的类型时，那实际上只是 `Array<number>` 和 `Array<string>` 的简写。

```ts twoslash
function doSomething(value: Array<string>) {
  // ...
}

let myArray: string[] = ['hello', 'world']

// 以下两种方式都有效！
doSomething(myArray)
doSomething(new Array('hello', 'world'))
```

和上面的 `Box` 类型很像，`Array` 本身就是一个泛型类型。

```ts twoslash
// @noLib: true
interface Number {}
interface String {}
interface Boolean {}
interface Symbol {}
// ---cut---
interface Array<Type> {
  /**
   * 获取或设置数组的长度。
   */
  length: number

  /**
   * 从数组中移除最后一个元素并返回该元素。
   */
  pop(): Type | undefined

  /**
   * 向数组中追加新元素，并返回数组的新长度。
   */
  push(...items: Type[]): number

  // ...
}
```

现代 JavaScript 也提供了其他泛型数据结构，比如 `Map<K, V>`、`Set<T>` 和 `Promise<T>`。
这一切的核心在于，基于 `Map`、`Set` 和 `Promise` 的行为特征，它们可以适配任何类型的集合。

### `ReadonlyArray` 类型

`ReadonlyArray` 是一个特殊类型，用于描述不应被更改的数组。

```ts twoslash
// @errors: 2339
function doStuff(values: ReadonlyArray<string>) {
  // 我们可以从 'values' 读取...
  const copy = values.slice()
  console.log(`The first value is ${values[0]}`)

  // ...但我们不能改变 'values'。
  values.push('hello!')
}
```

与属性的 `readonly` 修饰符很像，它主要是一种表达意图的工具。当我们看到一个返回 `ReadonlyArray` 的函数时，它告诉我们完全不应当更改其内容；当我们看到一个接受 `ReadonlyArray` 的函数时，它告诉我们可以把任何数组传给该函数，而不用担心它会改变数组内容。

与 `Array` 不同，没有可供我们使用的 `ReadonlyArray` 构造函数。

```ts twoslash
// @errors: 2693
new ReadonlyArray('red', 'green', 'blue')
```

不过，我们可以把普通的 `Array` 赋值给 `ReadonlyArray`。

```ts twoslash
const roArray: ReadonlyArray<string> = ['red', 'green', 'blue']
```

正如 TypeScript 为 `Array<Type>` 提供了 `Type[]` 的简写语法一样，它也为 `ReadonlyArray<Type>` 提供了 `readonly Type[]` 的简写语法。

```ts twoslash
// @errors: 2339
function doStuff(values: readonly string[]) {
  //                     ^^^^^^^^^^^^^^^^^
  // 我们可以从 'values' 读取...
  const copy = values.slice()
  console.log(`The first value is ${values[0]}`)

  // ...但我们不能改变 'values'。
  values.push('hello!')
}
```

最后要注意的是：与 `readonly` 属性修饰符不同，普通 `Array` 和 `ReadonlyArray` 之间的可赋值性不是双向的。

```ts twoslash
// @errors: 4104
let x: readonly string[] = []
let y: string[] = []

x = y
y = x
```

### 元组类型

_元组类型_ 是另一种 `Array` 类型，它确切地知道自己包含多少个元素，以及在特定位置上包含哪些类型。

```ts twoslash
type StringNumberPair = [string, number]
//                      ^^^^^^^^^^^^^^^^
```

在这里，`StringNumberPair` 是 `string` 和 `number` 的元组类型。与 `ReadonlyArray` 一样，它在运行时没有对应表示，但对 TypeScript 来说意义重大。对类型系统来说，`StringNumberPair` 描述的是索引 `0` 处包含 `string`、索引 `1` 处包含 `number` 的数组。

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

如果我们试图索引超出元素数量的位置，就会得到错误。

```ts twoslash
// @errors: 2493
function doSomething(pair: [string, number]) {
  // ...

  const c = pair[2]
}
```

我们还可以使用 JavaScript 的数组解构来[解构元组](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Array_destructuring)。

```ts twoslash
function doSomething(stringHash: [string, number]) {
  const [inputString, hash] = stringHash

  console.log(inputString)
  //          ^?

  console.log(hash)
  //          ^?
}
```

> 元组类型在重度依赖约定的 API 中很有用，因为在这种 API 中每个元素的含义都是“显而易见的”。这让我们在解构时可以灵活地为变量起任何想要的名字。在上面的例子中，我们可以随意命名元素 `0` 和 `1`。
>
> 但是，由于并非每个用户对"显而易见"的看法都一致，因此也许值得重新考虑：对你的 API 来说，使用带有描述性属性名的对象是否更好。

除了这些长度检查之外，像这样的简单元组类型等价于这样的 `Array` 版本：它为特定索引声明了属性，并用数字字面量类型声明了 `length`。

```ts twoslash
interface StringNumberPair {
  // 特定的属性
  length: 2
  0: string
  1: number

  // 'Array<string | number>' 的其他成员...
  slice(start?: number, end?: number): Array<string | number>
}
```

你可能感兴趣的另一点是：元组可以有可选元素，写法是在元素类型后面加上问号（`?`）。可选的元组元素只能出现在末尾，并且也会影响 `length` 的类型。

```ts twoslash
type Either2dOr3d = [number, number, number?]

function setCoordinate(coord: Either2dOr3d) {
  const [x, y, z] = coord
  //           ^?

  console.log(`Provided coordinates had ${coord.length} dimensions`)
  //                                            ^?
}
```

元组也可以有剩余元素，剩余元素必须是 数组/元组 类型。

```ts twoslash
type StringNumberBooleans = [string, number, ...boolean[]]
type StringBooleansNumber = [string, ...boolean[], number]
type BooleansStringNumber = [...boolean[], string, number]
```

- `StringNumberBooleans` 描述的是前两个元素分别为 `string` 和 `number`、但后面可以有任意数量 `boolean` 的元组。
- `StringBooleansNumber` 描述的是第一个元素为 `string`、然后是任意数量的 `boolean`、最后以 `number` 结尾的元组。
- `BooleansStringNumber` 描述的是开头为任意数量的 `boolean`、最后依次以 `string` 和 `number` 结尾的元组。

带有剩余元素的元组没有固定的“长度” —— 它只有一组位于不同位置的已知元素。

```ts twoslash
type StringNumberBooleans = [string, number, ...boolean[]]
// ---cut---
const a: StringNumberBooleans = ['hello', 1]
const b: StringNumberBooleans = ['beautiful', 2, true]
const c: StringNumberBooleans = ['world', 3, true, false, true, false, true]
```

为什么可选元素和剩余元素可能有用呢？嗯，它使得 TypeScript 能够把元组与参数列表对应起来。
元组类型可以用于[剩余参数和展开语法](/docs/handbook/2/functions.html#rest-parameters-and-arguments)，因此下面的写法：

```ts twoslash
function readButtonInput(...args: [string, number, ...boolean[]]) {
  const [name, version, ...input] = args
  // ...
}
```

基本上等价于：

```ts twoslash
function readButtonInput(name: string, version: number, ...input: boolean[]) {
  // ...
}
```

当你想用剩余参数接受可变数量的实参、需要保证最少数量的元素，但又不想引入中间变量时，这就很方便。

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

关于元组类型的最后一点说明：元组类型有 `readonly` 变体，可以通过在前面加上 `readonly` 修饰符来指定 —— 就像数组的简写语法一样。

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

在大多数代码中，元组创建后往往不会再被修改，因此尽可能将类型注解为 `readonly` 元组是一个好的默认选择。这一点也很重要，因为带有 `const` 断言的数组字面量会被推断为 `readonly` 元组类型。

```ts twoslash
// @errors: 2345
let point = [3, 4] as const

function distanceFromOrigin([x, y]: [number, number]) {
  return Math.sqrt(x ** 2 + y ** 2)
}

distanceFromOrigin(point)
```

在这里，`distanceFromOrigin` 从不修改其元素，但它期望一个可变元组。由于 `point` 的类型被推断为 `readonly [3, 4]`，它与 `[number, number]` 不兼容，因为后者无法保证 `point` 的元素不会被修改。

<!-- ## Other Kinds of Object Members

Most of the declarations in object types:

### Method Syntax

### Call Signatures

### Construct Signatures

### Index Signatures -->
