---
title: TypeScript 4.0
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-4-0.html
oneline: TypeScript 4.0 发布说明
---

## 变长元组类型

考虑一个 JavaScript 函数 `concat`，它接收两个数组或元组类型，并将它们拼接在一起生成一个新数组。

```js
function concat(arr1, arr2) {
  return [...arr1, ...arr2]
}
```

再来看 `tail` 函数，它接收一个数组或元组，并返回除第一个元素外的所有元素。

```js
function tail(arg) {
  const [_, ...result] = arg
  return result
}
```

我们该如何在 TypeScript 中为这两个函数添加类型定义？

对于 `concat`，在较早版本的语言中，唯一可行的做法就是尝试编写大量函数重载。

```ts
function concat(arr1: [], arr2: []): []
function concat<A>(arr1: [A], arr2: []): [A]
function concat<A, B>(arr1: [A, B], arr2: []): [A, B]
function concat<A, B, C>(arr1: [A, B, C], arr2: []): [A, B, C]
function concat<A, B, C, D>(arr1: [A, B, C, D], arr2: []): [A, B, C, D]
function concat<A, B, C, D, E>(arr1: [A, B, C, D, E], arr2: []): [A, B, C, D, E]
function concat<A, B, C, D, E, F>(
  arr1: [A, B, C, D, E, F],
  arr2: [],
): [A, B, C, D, E, F]
```

额……好吧，仅仅针对第二个数组为空的情况就写了七个重载。
我们再为 `arr2` 有一个元素的情况添加一些重载。

<!-- prettier-ignore -->
```ts
function concat<A2>(arr1: [], arr2: [A2]): [A2];
function concat<A1, A2>(arr1: [A1], arr2: [A2]): [A1, A2];
function concat<A1, B1, A2>(arr1: [A1, B1], arr2: [A2]): [A1, B1, A2];
function concat<A1, B1, C1, A2>(arr1: [A1, B1, C1], arr2: [A2]): [A1, B1, C1, A2];
function concat<A1, B1, C1, D1, A2>(arr1: [A1, B1, C1, D1], arr2: [A2]): [A1, B1, C1, D1, A2];
function concat<A1, B1, C1, D1, E1, A2>(arr1: [A1, B1, C1, D1, E1], arr2: [A2]): [A1, B1, C1, D1, E1, A2];
function concat<A1, B1, C1, D1, E1, F1, A2>(arr1: [A1, B1, C1, D1, E1, F1], arr2: [A2]): [A1, B1, C1, D1, E1, F1, A2];
```

显而易见，这种做法已经变得非常不合理了。
不幸的是，为类似 `tail` 的函数编写类型时也会遇到同样的问题。

这又是我们常说的“重载过多导致不堪重负”（death by a thousand overloads）的典型场景，而且它根本没有从根本上通用地解决问题。
它只能为我们不厌其烦手写出的那些重载提供正确的类型。
如果我们想提供一个通用的兜底方案，就必须写一个类似下面这样的重载：

```ts
function concat<T, U>(arr1: T[], arr2: U[]): Array<T | U>
```

但当使用元组时，该签名无法保留任何有关输入长度或元素顺序的信息。

TypeScript 4.0 带来了两项根本性变革以及推断能力的提升，使得为这类场景编写类型成为可能。

第一项改动是：元组类型语法中的展开运算符（spread）现在可以是泛型的。
这意味着即使我们不知道具体操作的实际类型，也可以表达对元组和数组的高阶操作。
当这些元组类型中的泛型展开被实例化（即被替换为真实类型）时，它们可以生成另一组数组和元组类型。

例如，这意味着我们可以为类似 `tail` 的函数声明类型，而不再饱受“海量重载”的困扰。

```ts twoslash
function tail<T extends any[]>(arr: readonly [any, ...T]) {
  const [_ignored, ...rest] = arr
  return rest
}

const myTuple = [1, 2, 3, 4] as const
const myArray = ['hello', 'world']

const r1 = tail(myTuple)
//    ^?

const r2 = tail([...myTuple, ...myArray] as const)
//    ^?
```

第二项改动是：剩余元素（rest elements）可以出现在元组中的任何位置——而不仅仅是末尾！

```ts
type Strings = [string, string]
type Numbers = [number, number]

type StrStrNumNumBool = [...Strings, ...Numbers, boolean]
```

此前，TypeScript 会报告类似以下的错误：

```
A rest element must be last in a tuple type.
```

但在 TypeScript 4.0 中，放宽了这一限制。

需要注意的是，当展开一个长度未知的类型时，结果类型也会变成无界（unbounded）的，并且其后的所有元素都会合并到最终的剩余元素类型中。

```ts
type Strings = [string, string]
type Numbers = number[]

type Unbounded = [...Strings, ...Numbers, boolean]
```

结合这两种特性，我们就可以为 `concat` 编写一个类型完备且单一的签名：

```ts twoslash
type Arr = readonly any[]

function concat<T extends Arr, U extends Arr>(arr1: T, arr2: U): [...T, ...U] {
  return [...arr1, ...arr2]
}
```

虽然这单个签名看起来仍然有点长，但只需写这一次，无需重复编写，并且能为所有数组和元组提供可预测的行为表现。

这项功能本身就已经非常出色，但在更复杂的场景中更能大放异彩。
例如，考虑一个用于[偏函数应用（partially apply arguments）](https://en.wikipedia.org/wiki/Partial_application)的函数 `partialCall`。
`partialCall` 接收一个函数（假设为 `f`）以及 `f` 所期望的前几个参数。
然后它返回一个新函数，该新函数接收 `f` 所需的其余参数，并在接收到这些参数时调用 `f`。

```js
function partialCall(f, ...headArgs) {
  return (...tailArgs) => f(...headArgs, ...tailArgs)
}
```

TypeScript 4.0 改进了剩余参数和剩余元组元素的推断过程，使得我们能够为其定义类型并“直接开箱即用”。

```ts twoslash
type Arr = readonly unknown[]

function partialCall<T extends Arr, U extends Arr, R>(
  f: (...args: [...T, ...U]) => R,
  ...headArgs: T
) {
  return (...tailArgs: U) => f(...headArgs, ...tailArgs)
}
```

在这种情况下，`partialCall` 能够识别最初可以和不可以接收哪些参数，并返回能正确接受或拒绝剩余参数的函数。

```ts twoslash
// @errors: 2345 2554 2554 2345
type Arr = readonly unknown[]

function partialCall<T extends Arr, U extends Arr, R>(
  f: (...args: [...T, ...U]) => R,
  ...headArgs: T
) {
  return (...tailArgs: U) => f(...headArgs, ...tailArgs)
}
// ---cut---
const foo = (x: string, y: number, z: boolean) => {}

const f1 = partialCall(foo, 100)

const f2 = partialCall(foo, 'hello', 100, true, 'oops')

// This works!
const f3 = partialCall(foo, 'hello')
//    ^?

// What can we do with f3 now?

// Works!
f3(123, true)

f3()

f3(123, 'hello')
```

变长元组类型开创了许多令人兴奋的新模式，尤其是在函数组合方面。
我们期望未来能够利用它更好地对 JavaScript 内置的 `bind` 方法进行类型检查。
这一特性中还包含了若干其他推断改进和模式支持，如果你有兴趣了解更多，可以查看变长元组的 [Pull Request](https://github.com/microsoft/TypeScript/pull/39094)。

## 带标签的元组元素

改善元组类型和参数列表的使用体验至关重要，因为它能为常见的 JavaScript 惯用模式提供强类型验证——尤其是对参数列表进行切片拆分并传递给其他函数的场景。
在剩余参数中使用元组类型就是这一特性的关键体现之一。

例如，下面这个使用元组类型作为剩余参数的函数……

```ts
function foo(...args: [string, number]): void {
  // ...
}
```

……对于 `foo` 的任何调用者而言，与下面这个函数……

```ts
function foo(arg0: string, arg1: number): void {
  // ...
}
```

……应该没有任何区别。

```ts twoslash
// @errors: 2554
function foo(arg0: string, arg1: number): void {
  // ...
}
// ---cut---
foo('hello', 42)

foo('hello', 42, true)
foo('hello')
```

然而，两者之间在某个方面开始显现出差异：可读性。
在第一个例子中，我们无法为第一和第二个元素提供参数名称。
尽管这不会对类型检查产生任何影响，但元组各位置缺少标签会使其更难使用，也更难清晰表达我们的意图。

这就是为什么在 TypeScript 4.0 中，元组类型现在支持提供标签。

```ts
type Range = [start: number, end: number]
```

为了加深参数列表与元组类型之间的联系，剩余元素和可选元素的语法与参数列表的语法保持了一致。

```ts
type Foo = [first: number, second?: string, ...rest: any[]]
```

使用带标签的元组时有一些规则。
首先，只要为元组中的某个元素添加了标签，该元组中的所有其他元素也必须添加标签。

```ts twoslash
// @errors: 5084
type Bar = [first: string, number]
```

值得注意的是——解构时标签并不强制我们使用对应的变量名称。
它们纯粹是为了文档说明和编辑器工具支持而存在的。

```ts twoslash
function foo(x: [first: string, second: number]) {
  // ...

  // note: we didn't need to name these 'first' and 'second'
  const [a, b] = x
  a
  //  ^?
  b
  //  ^?
}
```

总体而言，在利用元组和参数列表的模式以及以类型安全的方式实现重载时，带标签的元组非常方便。
事实上，TypeScript 的编辑器支持会尽可能尝试将它们显示为重载。

![参数签名帮助中将带标签元组的联合类型像参数列表一样显示为两个签名](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2020/08/signatureHelpLabeledTuples.gif)

欲了解更多信息，请查看带标签元组元素的 [Pull Request](https://github.com/microsoft/TypeScript/pull/38234)。

## 从构造函数推断类属性

在启用了 [`noImplicitAny`](/tsconfig#noImplicitAny) 的情况下，TypeScript 4.0 现在可以使用控制流分析来确定类中属性的类型。

<!--prettier-ignore -->
```ts twoslash
class Square {
  // Previously both of these were any
  area;
// ^?
  sideLength;
// ^?
  constructor(sideLength: number) {
    this.sideLength = sideLength;
    this.area = sideLength ** 2;
  }
}
```

如果构造函数并非所有代码分支都为某个实例成员赋值，则该属性将被视为可能包含 `undefined`。

<!--prettier-ignore -->
```ts twoslash
// @errors: 2532 18048
class Square {
  sideLength;
// ^?

  constructor(sideLength: number) {
    if (Math.random()) {
      this.sideLength = sideLength;
    }
  }

  get area() {
    return this.sideLength ** 2;
  }
}
```

如果你明确知晓属性会被赋值（例如你有某种 `initialize` 方法），且处于 [`strictPropertyInitialization`](/tsconfig#strictPropertyInitialization) 模式下，你仍需要显式的类型注解以及明确赋值断言（`!`）。

```ts twoslash
class Square {
  // definite assignment assertion
  //        v
  sideLength!: number
  //         ^^^^^^^^
  // type annotation

  constructor(sideLength: number) {
    this.initialize(sideLength)
  }

  initialize(sideLength: number) {
    this.sideLength = sideLength
  }

  get area() {
    return this.sideLength ** 2
  }
}
```

有关更多详细信息，请[查看实现的 Pull Request](https://github.com/microsoft/TypeScript/pull/37920)。

## 短路赋值运算符

JavaScript 以及许多其他语言都支持一组称为“复合赋值运算符”（_compound assignment_ operators）的操作符。
复合赋值运算符将某种运算应用于两个操作数，然后将结果赋值给左侧操作数。
你之前可能见过这些：

```ts
// Addition
// a = a + b
a += b

// Subtraction
// a = a - b
a -= b

// Multiplication
// a = a * b
a *= b

// Division
// a = a / b
a /= b

// Exponentiation
// a = a ** b
a **= b

// Left Bit Shift
// a = a << b
a <<= b
```

JavaScript 中的绝大多数运算符都有对应的赋值运算符！
然而直到最近，还有三个显著的例外：逻辑与（`&&`）、逻辑或（`||`）和空值合并（`??`）。

正因如此，TypeScript 4.0 支持了一项新的 ECMAScript 特性，新增了三个赋值运算符：`&&=`、`||=` 和 `??=`。

这些运算符非常适合替代开发者以前写出的如下代码：

```ts
a = a && b
a = a || b
a = a ?? b
```

或者类似的 `if` 代码块：

```ts
// could be 'a ||= b'
if (!a) {
  a = b
}
```

我们甚至还见过（或者，额……自己写过）一些仅在需要时惰性初始化值的模式：

```ts
let values: string[]
;(values ?? (values = [])).push('hello')

// After
;(values ??= []).push('hello')
```

（好吧，我们写的代码也并非_每一行_都值得夸耀……）

在少数使用带有副作用的 getter 或 setter 的情况下，需要注意的是，这些运算符仅在必要时才会执行赋值。
从这个意义上说，不仅运算符右侧被“短路”了——赋值操作本身也被短路了。

```ts
obj.prop ||= foo()

// roughly equivalent to either of the following

obj.prop || (obj.prop = foo())

if (!obj.prop) {
  obj.prop = foo()
}
```

[尝试运行以下示例](https://www.typescriptlang.org/play?ts=next#code/MYewdgzgLgBCBGArGBeGBvAsAKBnmA5gKawAOATiKQBQCUGO+TMokIANkQHTsgHUAiYlChFyMABYBDCDHIBXMANoBuHI2Z4A9FpgAlIqXZTgRGAFsiAQg2byJeeTAwAslKgSu5KWAAmIczoYAB4YAAYuAFY1XHwAXwAaWxgIEhgKKmoAfQA3KXYALhh4EA4iH3osWM1WCDKePkFUkTFJGTlFZRimOJw4mJwAM0VgKABLcBhB0qCqplr63n4BcjGCCVgIMd8zIjz2eXciXy7k+yhHZygFIhje7BwFzgblgBUJMdlwM3yAdykAJ6yBSQGAeMzNUTkU7YBCILgZUioOBIBGUJEAHwxUxmqnU2Ce3CWgnenzgYDMACo6pZxpYIJSOqDwSkSFCYXC0VQYFi0NMQHQVEA)，看看这与_总是_执行赋值有何不同。

```ts twoslash
const obj = {
  get prop() {
    console.log('getter has run')

    // Replace me!
    return Math.random() < 0.5
  },
  set prop(_val: boolean) {
    console.log('setter has run')
  },
}

function foo() {
  console.log('right side evaluated')
  return true
}

console.log('This one always runs the setter')
obj.prop = obj.prop || foo()

console.log('This one *sometimes* runs the setter')
obj.prop ||= foo()
```

我们衷心感谢社区成员 [Wenlu Wang](https://github.com/Kingwl) 贡献了这一特性！

有关更多详细信息，你可以[在此查看 Pull Request](https://github.com/microsoft/TypeScript/pull/37727)。
你还可以[查看该特性的 TC39 提案仓库](https://github.com/tc39/proposal-logical-assignment/)。

## catch 子句绑定中的 `unknown` 类型

自 TypeScript 诞生初期以来，`catch` 子句中的变量类型一直都是 `any`。
这意味着 TypeScript 允许你对其进行任意操作。

```ts twoslash
// @useUnknownInCatchVariables: false
try {
  // Do some work
} catch (x) {
  // x has type 'any' - have fun!
  console.log(x.message)
  console.log(x.toUpperCase())
  x++
  x.yadda.yadda.yadda()
}
```

如果我们想防止在错误处理代码中发生_更多_错误，上述行为就会带来隐患！
因为这些变量默认具有 `any` 类型，它们缺乏任何类型安全性，本应对非法操作报错的情况却被放过了。

这就是为什么 TypeScript 4.0 现在允许将 `catch` 子句变量的类型指定为 `unknown`。
`unknown` 比 `any` 更安全，因为它会提醒我们在操作这些值之前必须执行某种类型检查。

<!--prettier-ignore -->
```ts twoslash
// @errors: 2571 18046
try {
  // ...
} catch (e: unknown) {
  // Can't access values on unknowns
  console.log(e.toUpperCase());

  if (typeof e === "string") {
    // We've narrowed 'e' down to the type 'string'.
    console.log(e.toUpperCase());
  }
}
```

虽然 `catch` 变量的类型默认不会改变，但我们未来可能会考虑引入一个新的 [`strict`](/tsconfig#strict) 模式标志，以便用户主动选择启用此行为。
与此同时，开发者也可以编写一条 lint 规则，强制要求 `catch` 变量显式标注 `: any` 或 `: unknown`。

有关更多详细信息，你可以[查看此特性的相关改动](https://github.com/microsoft/TypeScript/pull/39015)。

## 自定义 JSX 工厂

使用 JSX 时，[片段（_fragment_）](https://reactjs.org/docs/fragments.html)是一种允许我们返回多个子元素的 JSX 元素类型。
最初我们在 TypeScript 中实现片段支持时，并不完全清楚其他库会如何使用它们。
如今，大多数鼓励使用 JSX 且支持片段的库都具有类似的 API 结构。

在 TypeScript 4.0 中，用户可以通过新增的 [`jsxFragmentFactory`](/tsconfig#jsxFragmentFactory) 选项来自定义片段工厂。

例如，以下 `tsconfig.json` 文件指示 TypeScript 以与 React 兼容的方式转换 JSX，但将每个工厂函数调用从 `React.createElement` 替换为 `h`，并使用 `Fragment` 替代 `React.Fragment`。

```jsonc tsconfig
{
  "compilerOptions": {
    "target": "esnext",
    "module": "commonjs",
    "jsx": "react",
    "jsxFactory": "h",
    "jsxFragmentFactory": "Fragment",
  },
}
```

如果需要在单个文件级别使用不同的 JSX 工厂<!-- (maybe you like to ship React, Preact, and Inferno to give a blazing fast experience) -->，你可以利用全新的 `/** @jsxFrag */` 编译指示注释（pragma comment）。
例如，以下代码……

```tsx twoslash
// @noErrors
// Note: these pragma comments need to be written
// with a JSDoc-style multiline syntax to take effect.

/** @jsx h */
/** @jsxFrag Fragment */

import { h, Fragment } from 'preact'

export const Header = (
  <>
    <h1>Welcome</h1>
  </>
)
```

……将被转换为以下 JavaScript 输出……

```tsx twoslash
// @noErrors
// @showEmit
// Note: these pragma comments need to be written
// with a JSDoc-style multiline syntax to take effect.

/** @jsx h */
/** @jsxFrag Fragment */

import { h, Fragment } from 'preact'

export const Header = (
  <>
    <h1>Welcome</h1>
  </>
)
```

我们衷心感谢社区成员 [Noj Vek](https://github.com/nojvek) 提交了该 Pull Request 并耐心地与我们团队共同完善它。

你可以查看该 [Pull Request](https://github.com/microsoft/TypeScript/pull/38720) 了解更多细节！

## `--build` 模式与 `--noEmitOnError` 协作下的速度提升

此前，在 [`incremental`](/tsconfig#incremental) 模式下，当使用了 [`noEmitOnError`](/tsconfig#noEmitOnError) 标志时，在先前存在错误的编译之后再次编译程序会变得极其缓慢。
这是因为受到 [`noEmitOnError`](/tsconfig#noEmitOnError) 标志的影响，上次编译的任何信息都不会被缓存在 `.tsbuildinfo` 文件中。

TypeScript 4.0 改变了这一机制，从而在这些场景下带来了巨大的速度提升，进而优化了 `--build` 模式的场景（该模式会同时隐含 [`incremental`](/tsconfig#incremental) 和 [`noEmitOnError`](/tsconfig#noEmitOnError)）。

详情请参阅该 [Pull Request](https://github.com/microsoft/TypeScript/pull/38853)。

## `--incremental` 与 `--noEmit` 搭配使用

TypeScript 4.0 允许我们在使用 [`noEmit`](/tsconfig#noEmit) 标志的同时，依然利用 [`incremental`](/tsconfig#incremental) 进行增量编译。
此前这并不被允许，因为 [`incremental`](/tsconfig#incremental) 需要输出 `.tsbuildinfo` 文件；然而，实现更快速的增量构建这一使用场景极为重要，因此值得为所有用户开放此功能。

有关更多详细信息，你可以[查看实现的 Pull Request](https://github.com/microsoft/TypeScript/pull/39122)。

## 编辑器改进

TypeScript 编译器不仅在绝大多数主流编辑器中支撑了 TypeScript 本身的编辑体验，还在 Visual Studio 系列等编辑器中驱动着 JavaScript 的开发体验。
正因如此，我们的许多工作都聚焦于改进编辑器场景——也就是开发者花费大部分时间的地方。

在编辑器中使用新的 TypeScript/JavaScript 功能的具体方式因编辑器而异，不过：

- Visual Studio Code 支持[选择不同版本的 TypeScript](https://code.visualstudio.com/docs/typescript/typescript-compiling#_using-the-workspace-version-of-typescript)。此外，还可以使用 [JavaScript/TypeScript Nightly 扩展](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-typescript-next)来体验前沿版本（通常也很稳定）。
- Visual Studio 2017/2019 提供了上述 SDK 安装程序以及 [MSBuild 安装包](https://www.nuget.org/packages/Microsoft.TypeScript.MSBuild)。
- Sublime Text 3 支持[选择不同版本的 TypeScript](https://github.com/microsoft/TypeScript-Sublime-Plugin#note-using-different-versions-of-typescript)。

你可以查看这份不完全的[支持 TypeScript 的编辑器列表](https://github.com/Microsoft/TypeScript/wiki/TypeScript-Editor-Support)，了解你喜爱的编辑器是否支持使用新版本。

### 转换为可选链

可选链（Optional chaining）是近期备受喜爱的一项特性。
因此，TypeScript 4.0 带来了一项新的重构功能，可将常见模式转换为利用[可选链](https://devblogs.microsoft.com/typescript/announcing-typescript-3-7/#optional-chaining)和[空值合并](https://devblogs.microsoft.com/typescript/announcing-typescript-3-7/#nullish-coalescing)的形式！

![将 `a && a.b.c && a.b.c.d.e.f()` 转换为 `a?.b.c?.d.e.f.()`](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2020/08/convertToOptionalChain-4-0.gif)

请注意，由于 JavaScript 中真值（truthiness）和假值（falsiness）的微妙差异，该重构可能无法_完全_复刻原有的精确行为，但我们相信它能够满足绝大多数用例的意图，尤其是在 TypeScript 对类型具有更精确了解的情况下。

欲了解更多详情，请[查看该特性的 Pull Request](https://github.com/microsoft/TypeScript/pull/39135)。

### 支持 `/** @deprecated */`

TypeScript 的编辑器支持现在能够识别使用 `/** @deprecated */` JSDoc 注释标记的声明。
该信息会体现在补全列表中，并作为建议诊断信息提供给编辑器进行特殊处理。
在 VS Code 等编辑器中，已废弃的值通常会以删除线样式（~~像这样~~）显示。

![编辑器中带有删除线文本的已废弃声明示例](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2020/06/deprecated_4-0.png)

这项新功能的实现得益于 [Wenlu Wang](https://github.com/Kingwl)。
详情请参阅该 [Pull Request](https://github.com/microsoft/TypeScript/pull/38523)。

### 启动时的部分语义模式

我们收到过许多关于启动时间过长的用户反馈，尤其是在较大型的项目中。
罪魁祸首通常是一个被称为“程序构建”（_program construction_）的过程。
该过程从一组初始根文件开始，解析它们，查找其依赖项，解析这些依赖项，查找这些依赖项的依赖项，以此类推。
项目越大，在获得跳转到定义（go-to-definition）或快速信息（quick info）等基本编辑器操作之前需要等待的时间就越长。

因此，我们一直在为编辑器开发一种新模式，在完整的语言服务完全加载之前提供“部分”（_partial_）体验。
核心理念是编辑器可以运行一个轻量级的部分服务，仅关注编辑器当前打开的文件。

很难确切地说你会看到多大幅度的提升，但从经验来看，在 Visual Studio Code 自己的代码库上，以前通常需要 _20 秒到 1 分钟_ TypeScript 才能完全响应。
相比之下，**我们全新的部分语义模式似乎将该延迟缩短到了仅仅几秒钟**。
作为示例，在下方的视频中，你可以看到两个并排的编辑器，左侧运行 TypeScript 3.9，右侧运行 TypeScript 4.0。

<video loop autoplay muted style="width:100%;height:100%;" src="https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2020/08/partialModeFast.mp4">
</video>

在一个特别庞大的代码库上重启两个编辑器时，运行 TypeScript 3.9 的编辑器完全无法提供代码补全或快速信息。
另一方面，运行 TypeScript 4.0 的编辑器即使在后台加载完整项目的同时，也能_立即_在当前正在编辑的文件中提供丰富的语言服务体验。

目前唯一支持该模式的编辑器是 [Visual Studio Code](http://code.visualstudio.com/)，其体验将在 [Visual Studio Code Insiders](http://code.visualstudio.com/insiders) 中迎来进一步的 UX 改进。
我们意识到这一体验在用户体验和功能上仍有打磨空间，并且我们已经拟定了一份[改进清单](https://github.com/microsoft/TypeScript/issues/39035)在心中。
我们期待收集更多关于你认为有价值的反馈。

有关更多信息，你可以[查看最初的提案](https://github.com/microsoft/TypeScript/issues/37713)、[实现的 Pull Request](https://github.com/microsoft/TypeScript/pull/38561)，以及[后续的汇总 issue](https://github.com/microsoft/TypeScript/issues/39035)。

### 更智能的自动导入

自动导入（Auto-import）是一项出色的功能，能大幅简化编码工作；然而，每当自动导入失效时，都会让开发者大为困扰。
我们从用户那里听到的一个具体问题是：自动导入对用 TypeScript 编写的依赖包不起作用——除非他们在项目的其他地方至少写过一次显式的 `import`。

为什么自动导入对 `@types` 模块有效，却对自带类型声明的包无效呢？
原因在于自动导入仅对项目_已经_包含的包生效。
由于 TypeScript 有一些特殊的默认设置会自动将 `node_modules/@types` 中的包添加到项目中，因此_这些_包能够被自动导入。
而另一方面，其他包则被排除了，因为遍历检索整个 `node_modules` 中的所有包开销_非常_昂贵。

这一切导致了一种糟糕的初次使用体验：当你尝试自动导入刚安装好但尚未在任何地方使用过的依赖时，它无法工作。

TypeScript 4.0 现在在编辑器场景下做了一些额外的工作，将你在 `package.json` 的 `dependencies`（以及 `peerDependencies`）字段中列出的包包含进来。
来自这些包的信息仅用于改进自动导入，不会影响类型检查等其他任何行为。
这使我们能够为所有包含类型的依赖项提供自动导入，而无需承担全面扫描 `node_modules` 的高昂代价。

在极少数情况下，如果你的 `package.json` 中列出了超过十个包含类型但尚未被导入的依赖项，该功能会自动禁用以防止项目加载过慢。
若要强制启用该功能或完全将其禁用，可以在编辑器中进行配置。
对于 Visual Studio Code，对应的设置项为“Include Package JSON Auto Imports”（或 `typescript.preferences.includePackageJsonAutoImports`）。

![配置“include package JSON auto imports”](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2020/08/configurePackageJsonAutoImports4-0.png)
有关更多详细信息，你可以查看[提案 issue](https://github.com/microsoft/TypeScript/issues/37812) 以及[实现的 Pull Request](https://github.com/microsoft/TypeScript/pull/38923)。

## 我们的全新官网！

[TypeScript 官方网站](https://www.typescriptlang.org/)最近进行了彻底的重构并已正式上线！

![新 TypeScript 官网的截图](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2020/08/ts-web.png)

[我们之前已经专门撰文介绍了我们的新网站](https://devblogs.microsoft.com/typescript/announcing-the-new-typescript-website/)，你可以在那里了解更多；同时值得一提的是，我们非常期待听到大家的反馈！
如果你有任何疑问、评论或建议，欢迎[在官网的 issue 跟踪器中提交反馈](https://github.com/microsoft/TypeScript-Website)。

## 破坏性变更

### `lib.d.ts` 的变更

内置的 `lib.d.ts` 声明发生了变更——特别是 DOM 相关的类型发生了变化。
其中最值得注意的变化是移除了 [`document.origin`](https://developer.mozilla.org/en-US/docs/Web/API/Document/origin)，该属性仅在旧版 IE 和 Safari 中受支持。
MDN 建议迁移到 [`self.origin`](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/origin)。

### 属性重写访问器（以及反向重写）将报错

此前，只有在使用 [`useDefineForClassFields`](/tsconfig#useDefineForClassFields) 时，属性重写访问器或访问器重写属性才会报错；然而，现在当派生类中声明的属性重写了基类中的 getter 或 setter 时，TypeScript 总是会报告错误。

```ts twoslash
// @errors: 1049 2610
class Base {
  get foo() {
    return 100
  }
  set foo(value) {
    // ...
  }
}

class Derived extends Base {
  foo = 10
}
```

```ts twoslash
// @errors: 2611
class Base {
  prop = 10
}

class Derived extends Base {
  get prop() {
    return 100
  }
}
```

有关更多详细信息，请参阅[实现的 Pull Request](https://github.com/microsoft/TypeScript/pull/37894)。

### `delete` 操作数必须为可选属性

在 [`strictNullChecks`](/tsconfig#strictNullChecks) 模式下使用 `delete` 运算符时，操作数现在必须是 `any`、`unknown`、`never`，或者是可选的（即类型中包含 `undefined`）。
否则，使用 `delete` 运算符将报错。

```ts twoslash
// @errors: 2790
interface Thing {
  prop: string
}

function f(x: Thing) {
  delete x.prop
}
```

有关更多详细信息，请参阅[实现的 Pull Request](https://github.com/microsoft/TypeScript/pull/37921)。

### 废弃 TypeScript 旧版节点工厂（Node Factory）的使用

目前 TypeScript 提供了一组用于生成 AST 节点的“工厂”（factory）函数；然而，TypeScript 4.0 提供了全新的节点工厂 API。
因此，在 TypeScript 4.0 中，我们决定废弃这些旧的函数，转而推荐使用新的 API。

有关更多详细信息，请[阅读该变更的相关 Pull Request](https://github.com/microsoft/TypeScript/pull/35282)。
