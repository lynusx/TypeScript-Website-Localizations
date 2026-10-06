---
title: TypeScript 4.1
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-4-1.html
oneline: TypeScript 4.1 发布说明
---

## 模板字面量类型

TypeScript 中的字符串字面量类型允许我们为期望接收一组特定字符串的函数和 API 建模。

```ts twoslash
// @errors: 2345
function setVerticalAlignment(location: 'top' | 'middle' | 'bottom') {
  // ...
}

setVerticalAlignment('middel')
```

这非常棒，因为字符串字面量类型基本上可以为我们的字符串值提供拼写检查。

我们也很喜欢字符串字面量可以在映射类型中用作属性名。
从这个意义上说，它们还可以作为构建基块：

```ts
type Options = {
  [K in 'noImplicitAny' | 'strictNullChecks' | 'strictFunctionTypes']?: boolean
}
// same as
//   type Options = {
//       noImplicitAny?: boolean,
//       strictNullChecks?: boolean,
//       strictFunctionTypes?: boolean
//   };
```

但字符串字面量类型还有另一个可以作为构建基块的地方：构建其他字符串字面量类型。

这就是为什么 TypeScript 4.1 引入了模板字面量类型。
它具有与 [JavaScript 中的模板字面量字符串](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)相同的语法，但是用于类型位置。
当与具体的字面量类型结合使用时，它通过拼接内容生成新的字符串字面量类型。

```ts twoslash
type World = 'world'

type Greeting = `hello ${World}`
//   ^?
```

当在插值位置传入联合类型时会发生什么？
它会生成由每个联合类型成员所能表示的所有可能字符串字面量的集合。

```ts twoslash
type Color = 'red' | 'blue'
type Quantity = 'one' | 'two'

type SeussFish = `${Quantity | Color} fish`
//   ^?
```

这项功能不仅限于发布说明中这些简短有趣的示例。
例如，多个 UI 组件库都在其 API 中提供了同时指定垂直和水平对齐方式的方法，通常使用单个类似 `"bottom-right"` 的字符串一次性指定两者。
在垂直对齐的 `"top"`、`"middle"`、`"bottom"` 与水平对齐的 `"left"`、`"center"`、`"right"` 之间，通过短横线将前者与后者拼接起来，共有 9 种可能的字符串。

```ts twoslash
// @errors: 2345
type VerticalAlignment = 'top' | 'middle' | 'bottom'
type HorizontalAlignment = 'left' | 'center' | 'right'

// Takes
//   | "top-left"    | "top-center"    | "top-right"
//   | "middle-left" | "middle-center" | "middle-right"
//   | "bottom-left" | "bottom-center" | "bottom-right"

declare function setAlignment(
  value: `${VerticalAlignment}-${HorizontalAlignment}`,
): void

setAlignment('top-left') // works!
setAlignment('top-middel') // error!
setAlignment('top-pot') // error! but good doughnuts if you're ever in Seattle
```

虽然在实际开发中存在**大量**此类 API 的用例，但这仍然算是一个简单的示例，因为我们本可以手动把它们列出来。
事实上，对于 9 个字符串来说手动列出完全没问题；但当你需要大量字符串时，应该考虑提前自动生成它们以节省每次类型检查的工作量（或者直接使用 `string`，这样理解起来也简单得多）。

这项功能的真正价值很大程度上来自于动态创建新的字符串字面量。
例如，设想一个 `makeWatchedObject` API，它接收一个对象并生成一个大体相同的对象，但附带一个新的 `on` 方法来监听属性的变更。

```ts
let person = makeWatchedObject({
  firstName: 'Homer',
  age: 42, // give-or-take
  location: 'Springfield',
})

person.on('firstNameChanged', () => {
  console.log(`firstName was changed!`)
})
```

请注意，`on` 监听的是事件 `"firstNameChanged"`，而不仅仅是 `"firstName"`。
我们该如何为其定义类型？

```ts twslash
type PropEventSource<T> = {
  on(eventName: `${string & keyof T}Changed`, callback: () => void): void
}

/// Create a "watched object" with an 'on' method
/// so that you can watch for changes to properties.
declare function makeWatchedObject<T>(obj: T): T & PropEventSource<T>
```

有了这个定义，当我们传入错误的属性名时，TypeScript 就会报错！

```ts twoslash
// @errors: 2345
type PropEventSource<T> = {
  on(eventName: `${string & keyof T}Changed`, callback: () => void): void
}
declare function makeWatchedObject<T>(obj: T): T & PropEventSource<T>
let person = makeWatchedObject({
  firstName: 'Homer',
  age: 42, // give-or-take
  location: 'Springfield',
})

// ---cut---
// error!
person.on('firstName', () => {})

// error!
person.on('frstNameChanged', () => {})
```

我们还可以在模板字面量类型中做一些特殊的操作：我们可以从插值位置进行_推断_（_infer_）。
我们可以将上一个示例泛型化，从 `eventName` 字符串的各部分进行推断，从而确定对应的属性。

```ts twoslash
type PropEventSource<T> = {
  on<K extends string & keyof T>(
    eventName: `${K}Changed`,
    callback: (newValue: T[K]) => void,
  ): void
}

declare function makeWatchedObject<T>(obj: T): T & PropEventSource<T>

let person = makeWatchedObject({
  firstName: 'Homer',
  age: 42,
  location: 'Springfield',
})

// works! 'newName' is typed as 'string'
person.on('firstNameChanged', (newName) => {
  // 'newName' has the type of 'firstName'
  console.log(`new name is ${newName.toUpperCase()}`)
})

// works! 'newAge' is typed as 'number'
person.on('ageChanged', (newAge) => {
  if (newAge < 0) {
    console.log('warning! negative age')
  }
})
```

这里我们将 `on` 定义为了一个泛型方法。
当用户使用字符串 `"firstNameChanged"` 调用它时，TypeScript 会尝试推断出 `K` 的正确类型。
为此，它将 `K` 与 `"Changed"` 前面的内容进行匹配，推断出字符串 `"firstName"`。
一旦 TypeScript 确定了这一点，`on` 方法就可以获取原始对象上 `firstName` 的类型（本例中为 `string`）。
类似地，当我们使用 `"ageChanged"` 调用时，它会查找到属性 `age` 的类型（即 `number`）。

推断可以以不同的方式组合使用，通常用于解构字符串并以不同方式重构它们。
实际上，为了帮助修改这些字符串字面量类型，我们新增了几个用于修改字母大小写的工具类型别名（即转换为小写和大写字符）。

```ts twoslash
type EnthusiasticGreeting<T extends string> = `${Uppercase<T>}`

type HELLO = EnthusiasticGreeting<'hello'>
//   ^?
```

新增的类型别名包括 `Uppercase`、`Lowercase`、`Capitalize` 和 `Uncapitalize`。
前两者会转换字符串中的每一个字符，而后两者仅转换字符串的首字母。

欲了解更多详情，请[查看最初的 Pull Request](https://github.com/microsoft/TypeScript/pull/40336) 以及[切换为类型别名辅助类型的后续 Pull Request](https://github.com/microsoft/TypeScript/pull/40580)。

## 映射类型中的键重映射

复习一下，映射类型可以基于任意键创建新的对象类型：

```ts
type Options = {
  [K in 'noImplicitAny' | 'strictNullChecks' | 'strictFunctionTypes']?: boolean
}
// same as
//   type Options = {
//       noImplicitAny?: boolean,
//       strictNullChecks?: boolean,
//       strictFunctionTypes?: boolean
//   };
```

或者基于其他对象类型创建新的对象类型：

```ts
/// 'Partial<T>' is the same as 'T', but with each property marked optional.
type Partial<T> = {
  [K in keyof T]?: T[K]
}
```

在此之前，映射类型只能使用你提供的键来生成新的对象类型；然而在很多情况下，你希望能够根据输入创建新键，或者过滤掉某些键。

这就是为什么 TypeScript 4.1 允许你在映射类型中通过全新的 `as` 子句来对键进行重映射（re-map）。

```ts
type MappedTypeWithNewKeys<T> = {
  [K in keyof T as NewKeyType]: T[K]
  //            ^^^^^^^^^^^^^
  //            This is the new syntax!
}
```

借助这个新的 `as` 子句，你可以利用模板字面量类型等特性，轻松基于旧属性名创建新的属性名。

```ts twoslash
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
}

interface Person {
  name: string
  age: number
  location: string
}

type LazyPerson = Getters<Person>
//   ^?
```

你甚至可以通过生成 `never` 来过滤掉键。
这意味着在某些情况下你不再需要使用额外的 `Omit` 辅助类型。

```ts twoslash
// Remove the 'kind' property
type RemoveKindField<T> = {
  [K in keyof T as Exclude<K, 'kind'>]: T[K]
}

interface Circle {
  kind: 'circle'
  radius: number
}

type KindlessCircle = RemoveKindField<Circle>
//   ^?
```

有关更多信息，请查看 [GitHub 上的原始 Pull Request](https://github.com/microsoft/TypeScript/pull/40336)。

## 递归条件类型

在 JavaScript 中，能够展平或在任意嵌套层级构建容器类型的函数相当常见。
例如，考虑 `Promise` 实例上的 `.then()` 方法。
`.then(...)` 会不断解包每个 promise，直到找到一个非“Promise-like”的值，并将该值传递给回调函数。
此外，`Array` 上还有一个相对较新的 `flat` 方法，它可以接收一个深度参数来指定展平层数。

从所有实际使用的角度来看，在以往的 TypeScript 类型系统中表达这一点几乎是不可能的。
虽然有一些 hack 手段可以实现，但最终写出的类型看起来极不合理。

因此，TypeScript 4.1 放宽了对条件类型的一些限制，使其能够对这些模式进行建模。
在 TypeScript 4.1 中，条件类型现在可以直接在其分支内引用自身，从而更轻松地编写递归类型别名。

例如，如果我们想编写一个类型来获取嵌套数组的元素类型，可以编写如下的 `deepFlatten` 类型：

```ts
type ElementType<T> = T extends ReadonlyArray<infer U> ? ElementType<U> : T

function deepFlatten<T extends readonly unknown[]>(x: T): ElementType<T>[] {
  throw 'not implemented'
}

// All of these return the type 'number[]':
deepFlatten([1, 2, 3])
deepFlatten([[1], [2, 3]])
deepFlatten([[1], [[2]], [[[3]]]])
```

类似地，在 TypeScript 4.1 中，我们可以编写一个 `Awaited` 类型来深度解包 `Promise`：

```ts
type Awaited<T> = T extends PromiseLike<infer U> ? Awaited<U> : T

/// Like `promise.then(...)`, but more accurate in types.
declare function customThen<T, U>(
  p: Promise<T>,
  onFulfilled: (value: Awaited<T>) => U,
): Promise<Awaited<U>>
```

需要牢记的是，递归类型虽然功能强大，但应当谨慎且克制地使用。

首先，这些类型可能会执行大量的计算工作，这意味着它们会增加类型检查时间。
在类型系统中为考拉兹猜想（Collatz conjecture）或斐波那契数列建模可能很有趣，但千万不要将其发布到 npm 上的 `.d.ts` 文件中。

除了计算开销巨大之外，面对足够复杂的输入时，这些类型可能会触及内部的递归深度上限。
一旦触及递归上限，就会导致编译期错误。
通常而言，宁可完全不使用这些类型，也不要编写在更实际的用例中可能报错的类型。

欲了解更多信息，请[查看其实现 Pull Request](https://github.com/microsoft/TypeScript/pull/40002)。

## 未检查的索引访问（`--noUncheckedIndexedAccess`）

TypeScript 拥有一项称为“索引签名”（_index signatures_）的特性。
这些签名用于告知类型系统：用户可以访问任意命名的属性。

```ts twoslash
interface Options {
  path: string
  permissions: number

  // Extra properties are caught by this index signature.
  [propName: string]: string | number
}

function checkOptions(opts: Options) {
  opts.path // string
  opts.permissions // number

  // These are all allowed too!
  // They have the type 'string | number'.
  opts.yadda.toString()
  opts['foo bar baz'].toString()
  opts[Math.random()].toString()
}
```

在上面的示例中，`Options` 具有一个索引签名，该签名表明任何未显式列出的被访问属性都应具有类型 `string | number`。
对于假定开发者清楚自己在做什么的乐观代码而言，这通常很方便；但事实是，JavaScript 中的大多数值并不会支持每一个潜在的属性名。
例如在前面的示例中，大多数类型不可能存在由 `Math.random()` 生成的属性键所对应的值。
对许多用户来说，这种默认行为并不理想，感觉并没有充分发挥 [`strictNullChecks`](/tsconfig#strictNullChecks) 完整严格检查的能力。

这就是为什么 TypeScript 4.1 引入了一个名为 [`noUncheckedIndexedAccess`](/tsconfig#noUncheckedIndexedAccess) 的新标志。
在此新模式下，每次属性访问（如 `foo.bar`）或索引访问（如 `foo["bar"]`）都会被视为可能为 undefined。
这意味着在上一个示例中，`opts.yadda` 的类型将是 `string | number | undefined`，而不是仅仅是 `string | number`。
如果你需要访问该属性，要么必须先检查其是否存在，要么使用非空断言运算符（后缀 `!` 字符）。

```ts twoslash
// @errors: 2532 18048
// @noUncheckedIndexedAccess
interface Options {
  path: string
  permissions: number

  // Extra properties are caught by this index signature.
  [propName: string]: string | number
}
// ---cut---
function checkOptions(opts: Options) {
  opts.path // string
  opts.permissions // number

  // These are not allowed with noUncheckedIndexedAccess
  opts.yadda.toString()
  opts['foo bar baz'].toString()
  opts[Math.random()].toString()

  // Checking if it's really there first.
  if (opts.yadda) {
    console.log(opts.yadda.toString())
  }

  // Basically saying "trust me I know what I'm doing"
  // with the '!' non-null assertion operator.
  opts.yadda!.toString()
}
```

使用 [`noUncheckedIndexedAccess`](/tsconfig#noUncheckedIndexedAccess) 的一个连带影响是，数组索引访问也会受到更严格的检查，即使是在进行了边界检查的循环中也是如此。

```ts twoslash
// @errors: 2532 18048
// @noUncheckedIndexedAccess
function screamLines(strs: string[]) {
  // This will have issues
  for (let i = 0; i < strs.length; i++) {
    console.log(strs[i].toUpperCase())
  }
}
```

如果不需要索引，你可以使用 `for`-`of` 循环或 `forEach` 调用来遍历各个元素：

```ts twoslash
// @noUncheckedIndexedAccess
function screamLines(strs: string[]) {
  // This works fine
  for (const str of strs) {
    console.log(str.toUpperCase())
  }

  // This works fine
  strs.forEach((str) => {
    console.log(str.toUpperCase())
  })
}
```

该标志对于捕获越界错误非常方便，但对很多代码来说可能会产生过多提示，因此它不会被 [`strict`](/tsconfig#strict) 标志自动启用；然而，如果你对这项特性感兴趣，不妨尝试一下，评估它是否适合你团队的代码库！

你可以在[实现的 Pull Request](https://github.com/microsoft/TypeScript/pull/39560) 中了解更多。

## 无需 `baseUrl` 即可配置 `paths`

路径映射（path-mapping）的使用相当普遍——通常是为了让导入路径更优雅，或者为了模拟 monorepo 的链接行为。

遗憾的是，此前指定 [`paths`](/tsconfig#paths) 来启用路径映射时，必须同时指定名为 [`baseUrl`](/tsconfig#baseUrl) 的选项，这导致纯模块说明符（bare specifier）路径也会相对于 [`baseUrl`](/tsconfig#baseUrl) 解析。
这还经常导致自动导入生成不良的路径。

在 TypeScript 4.1 中，[`paths`](/tsconfig#paths) 选项可以在没有 [`baseUrl`](/tsconfig#baseUrl) 的情况下独立使用。
这有助于规避上述问题。

## `checkJs` 隐式启用 `allowJs`

此前，如果你要启动一个需要类型检查的 JavaScript 项目，必须同时设置 [`allowJs`](/tsconfig#allowJs) 和 [`checkJs`](/tsconfig#checkJs)。
这在体验上造成了一些不必要的摩擦，因此 [`checkJs`](/tsconfig#checkJs) 现在默认隐式启用 [`allowJs`](/tsconfig#allowJs)。

[查看 Pull Request 了解更多详情](https://github.com/microsoft/TypeScript/pull/40275)。

## React 17 JSX 工厂

TypeScript 4.1 通过为 [`jsx`](/tsconfig#jsx) 编译器选项新增两个配置值，支持了 React 17 即将推出的 `jsx` 和 `jsxs` 工厂函数：

- `react-jsx`
- `react-jsxdev`

这两个选项分别用于生产环境和开发环境的编译。
通常，其中一个配置可以继承自另一个配置。
例如，用于生产构建的 `tsconfig.json` 可能如下所示：

```json tsconfig
// ./src/tsconfig.json
{
  "compilerOptions": {
    "module": "esnext",
    "target": "es2015",
    "jsx": "react-jsx",
    "strict": true
  },
  "include": ["./**/*"]
}
```

而用于开发构建的配置可能如下所示：

```json tsconfig
// ./src/tsconfig.dev.json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "jsx": "react-jsxdev"
  }
}
```

欲了解更多信息，请[查看对应的 PR](https://github.com/microsoft/TypeScript/pull/39199)。

## 编辑器支持 JSDoc `@see` 标签

JSDoc 的 `@see` 标签在 TypeScript 和 JavaScript 编辑器中得到了更好的支持。
这使得你可以在标签后面的点分隔名称上使用“跳转到定义”（go-to-definition）等功能。
例如，在以下示例的 JSDoc 注释中，直接在 `first` 或 `C` 上跳转到定义即可正常工作：

```ts
// @filename: first.ts
export class C {}

// @filename: main.ts
import * as first from './first'

/**
 * @see first.C
 */
function related() {}
```

感谢活跃贡献者 [Wenlu Wang](https://github.com/Kingwl) [实现了此功能](https://github.com/microsoft/TypeScript/pull/39760)！

## 破坏性变更

### `lib.d.ts` 的变更

`lib.d.ts` 中有一组 API 发生了变更，部分原因可能是由于 DOM 类型的自动生成机制。
其中一个具体的变更是移除了 `Reflect.enumerate`，因为它已在 ES2016 中被移除。

### `abstract` 成员不能标记为 `async`

标记为 `abstract` 的成员不能再被标记为 `async`。
此处的修复方式是移除 `async` 关键字，因为调用方只关心返回类型。

### `any`/`unknown` 会在假值位置传播

此前，对于形如 `foo && somethingElse` 的表达式，当 `foo` 的类型为 `any` 或 `unknown` 时，整个表达式的类型就是 `somethingElse` 的类型。

例如，此前这里 `x` 的类型是 `{ someProp: string }`：

```ts
declare let foo: unknown
declare let somethingElse: { someProp: string }

let x = foo && somethingElse
```

然而在 TypeScript 4.1 中，我们在推断该类型时更加严谨。
由于对 `&&` 左侧的类型一无所知，我们将 `any` 和 `unknown` 向外传播，而不是直接采用右侧的类型。

我们发现最常见的模式往往出现在与 `boolean` 检查兼容性的场景中，尤其是在谓词函数（predicate functions）中：

```ts
function isThing(x: any): boolean {
  return x && typeof x === 'object' && x.blah === 'foo'
}
```

通常，合适的修复方法是将 `foo && someExpression` 更改为 `!!foo && someExpression`。

### `Promise` 中 `resolve` 的参数不再是可选的

编写如下代码时：

```ts
new Promise((resolve) => {
  doSomethingAsync(() => {
    doSomething()
    resolve()
  })
})
```

可能会遇到如下错误：

```
  resolve()
  ~~~~~~~~~
error TS2554: Expected 1 arguments, but got 0.
  An argument for 'value' was not provided.
```

这是因为 `resolve` 的参数不再是可选的，因此默认情况下现在必须向其传递一个值。
这往往能捕获使用 `Promise` 时实际存在的 bug。
典型的修复方法是为其传递正确的参数，有时还需要添加显式的类型参数：

```ts
new Promise<number>((resolve) => {
  //     ^^^^^^^^
  doSomethingAsync((value) => {
    doSomething()
    resolve(value)
    //      ^^^^^
  })
})
```

然而，有时 `resolve()` 确实需要在没有参数的情况下调用。
在这些情况下，我们可以为 `Promise` 提供显式的 `void` 泛型类型参数（即写成 `Promise<void>`）。
这利用了 TypeScript 4.1 中的新功能：可能为 `void` 的尾随参数可以变为可选参数。

```ts
new Promise<void>((resolve) => {
  //     ^^^^^^
  doSomethingAsync(() => {
    doSomething()
    resolve()
  })
})
```

TypeScript 4.1 提供了一个快速修复（quick fix）来帮助解决该破坏性变更。

### 条件展开会生成可选属性

在 JavaScript 中，对象展开（如 `{ ...foo }`）不会作用于假值。
因此在形如 `{ ...foo }` 的代码中，如果 `foo` 是 `null` 或 `undefined`，它会被跳过。

许多用户利用这一特性来“条件式地”（conditionally）展开属性：

```ts
interface Person {
  name: string
  age: number
  location: string
}

interface Animal {
  name: string
  owner: Person
}

function copyOwner(pet?: Animal) {
  return {
    ...(pet && pet.owner),
    otherStuff: 123,
  }
}

// We could also use optional chaining here:

function copyOwner(pet?: Animal) {
  return {
    ...pet?.owner,
    otherStuff: 123,
  }
}
```

这里，如果 `pet` 有定义，`pet.owner` 的属性就会被展开进去——否则，不会有任何属性展开到返回的对象中。

此前，`copyOwner` 的返回类型是基于每个展开生成的联合类型：

```
{ x: number } | { x: number, name: string, age: number, location: string }
```

这精确地模拟了操作的实际执行过程：如果 `pet` 有定义，来自 `Person` 的所有属性都会存在；否则，返回结果中不会定义其中的任何属性。
这是一个“全有或全无”（all-or-nothing）的操作。

然而，我们看到这种模式被推向了极致：单个对象中包含数百个展开，每个展开可能添加数百或数千个属性。
事实证明，由于种种原因，这最终会带来极高的计算开销，而通常并没有多大收益。

在 TypeScript 4.1 中，返回的类型有时会采用全可选属性（all-optional properties）：

```
{
    x: number;
    name?: string;
    age?: number;
    location?: string;
}
```

这最终带来了更好的性能，通常在展示上也更加清晰。

有关更多详细信息，请[参阅原始变更](https://github.com/microsoft/TypeScript/pull/40778)。
虽然该行为目前尚未完全一致，但我们预计未来的版本会产生更清晰、更可预测的结果。

### 不匹配的参数不再建立关联

此前，TypeScript 会通过将彼此不对应的参数关联到 `any` 类型来建立参数间的关联关系。
随着 [TypeScript 4.1 中的变更](https://github.com/microsoft/TypeScript/pull/41308)，语言现在完全跳过了这一过程。
这意味着某些可赋值性检查现在会失败，但这也意味着某些重载解析也可能会失败。
例如，Node.js 中的 `util.promisify` 重载解析在 TypeScript 4.1 中可能会选择不同的重载，有时会导致下游出现新的或不同的错误。

作为临时通融方案，最好使用类型断言来消除错误。
