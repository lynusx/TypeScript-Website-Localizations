---
title: 针对 JavaScript 开发者的 TypeScript 指南
short: 针对 JS 开发者的 TS 指南
layout: docs
permalink: /zh/docs/handbook/typescript-in-5-minutes.html
oneline: 了解 TypeScript 如何扩展 JavaScript
---

TypeScript 与 JavaScript 之间存在着一种特殊的关系。TypeScript 提供了 JavaScript 的所有特性，并在其之上增加了一层：TypeScript 的类型系统。

例如，JavaScript 提供了诸如 `string` 和 `number` 等语言原始类型，但它不会检查你对这些类型的赋值是否前后一致。而 TypeScript 会进行检查。

这意味着你现有的、能够正常运行的 JavaScript 代码同样也是 TypeScript 代码。TypeScript 的主要优势在于它可以指出代码中出人意料的行为，从而降低出现 bug 的几率。

本教程简要概述了 TypeScript，重点介绍其类型系统。

## 类型推断

TypeScript 能够理解 JavaScript 语言，并且在许多情况下会自动为你生成类型。
例如在创建变量并为其赋予特定值时，TypeScript 会将该值作为其类型推断的依据。

```ts twoslash
let helloWorld = 'Hello World'
//  ^?
```

通过理解 JavaScript 的运作方式，TypeScript 构建了一套能够接受 JavaScript 代码且具备类型的类型系统。这使你无需在代码中添加额外的字符显式声明类型，就能享受到类型系统的好处。上面的示例正是 TypeScript 由此推断出 `helloWorld` 为 `string` 类型的原因。

你可能在 Visual Studio Code 中编写过 JavaScript 代码并体验过编辑器的自动补全功能。Visual Studio Code 在底层正是借助 TypeScript 来改善 JavaScript 开发体验的。

## 定义类型

你可以在 JavaScript 中使用各种各样的设计模式。然而，某些设计模式会导致类型难以被自动推断（例如使用动态特性的模式）。为了应对这些情况，TypeScript 提供了对 JavaScript 语言的扩展，允许你在特定位置显式告知 TypeScript 应该使用什么类型。

例如，若要创建一个推断类型包含 `name: string` 和 `id: number` 的对象，你可以这样写：

```ts twoslash
const user = {
  name: 'Hayes',
  id: 0,
}
```

你可以使用 `interface` 声明显式描述该对象的形状：

```ts twoslash
interface User {
  name: string
  id: number
}
```

然后，你可以通过在变量声明后使用类似 `: TypeName` 的语法，来声明某个 JavaScript 对象符合新建 `interface` 的形状：

```ts twoslash
interface User {
  name: string
  id: number
}
// ---cut---
const user: User = {
  name: 'Hayes',
  id: 0,
}
```

如果你提供的对象与指定的接口不匹配，TypeScript 会发出警告：

```ts twoslash
// @errors: 2322
interface User {
  name: string
  id: number
}

const user: User = {
  username: 'Hayes',
  id: 0,
}
```

既然 JavaScript 支持类和面向对象编程，TypeScript 亦是如此。你可以在类中使用接口声明：

```ts twoslash
interface User {
  name: string
  id: number
}

class UserAccount {
  name: string
  id: number

  constructor(name: string, id: number) {
    this.name = name
    this.id = id
  }
}

const user: User = new UserAccount('Murphy', 1)
```

你可以使用接口来标注函数的参数和返回值：

```ts twoslash
// @noErrors
interface User {
  name: string
  id: number
}
// ---cut---
function deleteUser(user: User) {
  // ...
}

function getAdminUser(): User {
  //...
}
```

JavaScript 中已经内置了一组原始类型：`boolean`、`bigint`、`null`、`number`、`string`、`symbol` 和 `undefined`，你可以在接口中使用它们。TypeScript 在此基础上进行了扩充，增加了诸如 `any`（允许任何值）、[`unknown`](/play#example/unknown-and-never)（确保使用者必须声明其具体类型）、[`never`](/play#example/unknown-and-never)（表示不可能出现的值的类型）以及 `void`（返回 `undefined` 或没有返回值的函数）等类型。

你会发现构建类型有两种语法：[接口与类型别名（Interfaces and Types）](/play/?e=83#example/types-vs-interfaces)。通常建议优先使用 `interface`；当需要特定特性时，再使用 `type`。

## 组合类型

在 TypeScript 中，你可以通过组合简单类型来创建复杂类型。有两种常见的方式：联合类型和泛型。

### 联合类型

通过联合类型，你可以声明一个类型可以是多种类型之一。例如，你可以将 `boolean` 类型描述为 `true` 或 `false`：

```ts twoslash
type MyBool = true | false
```

_注意：_如果你将鼠标悬停在上面的 `MyBool` 上，会看到它被归类为 `boolean`。这是结构化类型系统的一个特性，下文将对此做进一步介绍。

联合类型的一个常见用例是描述一个值所允许的 `string` 或 `number` [字面量](/docs/handbook/2/everyday-types.html#literal-types)集合：

```ts twoslash
type WindowStates = 'open' | 'closed' | 'minimized'
type LockStates = 'locked' | 'unlocked'
type PositiveOddNumbersUnderTen = 1 | 3 | 5 | 7 | 9
```

联合类型还提供了一种处理不同类型的方式。例如，你可能有一个接收 `array` 或 `string` 的函数：

```ts twoslash
function getLength(obj: string | string[]) {
  return obj.length
}
```

若要获取变量的类型，可以使用 `typeof`：

| 类型      | 判定条件                           |
| --------- | ---------------------------------- |
| string    | `typeof s === "string"`            |
| number    | `typeof n === "number"`            |
| boolean   | `typeof b === "boolean"`           |
| undefined | `typeof undefined === "undefined"` |
| function  | `typeof f === "function"`          |
| array     | `Array.isArray(a)`                 |

例如，你可以让函数根据传入的是字符串还是数组来返回不同的值：

<!-- prettier-ignore -->
```ts twoslash
function wrapInArray(obj: string | string[]) {
  if (typeof obj === "string") {
    return [obj];
//          ^?
  }
  return obj;
}
```

### 泛型

泛型为类型提供了变量。一个常见的例子是数组。没有泛型的数组可以包含任何内容，而带有泛型的数组则可以描述该数组所包含的值。

```ts
type StringArray = Array<string>
type NumberArray = Array<number>
type ObjectWithNameArray = Array<{ name: string }>
```

你可以声明使用泛型的自定义类型：

```ts twoslash
// @errors: 2345
interface Backpack<Type> {
  add: (obj: Type) => void
  get: () => Type
}

// 这一行是一种简写方式，用来告诉 TypeScript 存在一个
// 名为 `backpack` 的常量，不用担心它来自何处。
declare const backpack: Backpack<string>

// object 是 string 类型，因为我们上面将它声明为了 Backpack 的可变部分。
const object = backpack.get()

// 由于 backpack 变量是 string 类型，因此不能将数字传递给 add 函数。
backpack.add(23)
```

## 结构化类型系统

TypeScript 的核心原则之一是类型检查关注于值所具有的**形状（shape）**。这有时被称为“鸭子类型（duck typing）”或“结构化类型（structural typing）”。

在结构化类型系统中，如果两个对象具有相同的形状，它们就被视为具有相同的类型。

```ts twoslash
interface Point {
  x: number
  y: number
}

function logPoint(p: Point) {
  console.log(`${p.x}, ${p.y}`)
}

// 打印 "12, 26"
const point = { x: 12, y: 26 }
logPoint(point)
```

变量 `point` 从未被声明为 `Point` 类型。然而，TypeScript 在类型检查中比较了 `point` 的形状与 `Point` 的形状。由于它们具有相同的形状，因此代码顺利通过了检查。

形状匹配只要求对象的字段子集匹配即可。

```ts twoslash
// @errors: 2345
interface Point {
  x: number
  y: number
}

function logPoint(p: Point) {
  console.log(`${p.x}, ${p.y}`)
}
// ---cut---
const point3 = { x: 12, y: 26, z: 89 }
logPoint(point3) // 打印 "12, 26"

const rect = { x: 33, y: 3, width: 30, height: 80 }
logPoint(rect) // 打印 "33, 3"

const color = { hex: '#187ABF' }
logPoint(color)
```

类与对象在符合形状要求的方式上没有任何区别：

```ts twoslash
// @errors: 2345
interface Point {
  x: number
  y: number
}

function logPoint(p: Point) {
  console.log(`${p.x}, ${p.y}`)
}
// ---cut---
class VirtualPoint {
  x: number
  y: number

  constructor(x: number, y: number) {
    this.x = x
    this.y = y
  }
}

const newVPoint = new VirtualPoint(13, 56)
logPoint(newVPoint) // 打印 "13, 56"
```

只要对象或类具备所有必需的属性，无论其具体实现细节如何，TypeScript 都会认定它们相互匹配。

## 下一步

以上是对日常 TypeScript 中使用的语法和工具的简要概述。接下来，你可以：

- [从头到尾](/docs/handbook/intro.html)阅读完整手册
- 探索 [Playground 示例](/play#show-examples)
