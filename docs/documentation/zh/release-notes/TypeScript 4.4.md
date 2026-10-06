---
title: TypeScript 4.4
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-4-4.html
oneline: TypeScript 4.4 发布说明
---

## 别名条件与判别属性的控制流分析

在 JavaScript 中，我们经常需要以不同的方式检查某个值，并在了解其类型后执行不同的操作。
TypeScript 能够理解这些检查，并称之为*类型守卫*（type guards）。
类型检查器利用所谓的*控制流分析*（control flow analysis）来观察我们在某段代码之前是否使用了类型守卫，而不是每次使用变量时都需要向 TypeScript 证明其类型。

例如，我们可以编写如下代码：

```ts twoslash
function foo(arg: unknown) {
  if (typeof arg === 'string') {
    console.log(arg.toUpperCase())
    //           ^?
  }
}
```

在这个示例中，我们检查了 `arg` 是否为 `string`。
TypeScript 识别出 `typeof arg === "string"` 检查并将其视为类型守卫，从而得知在 `if` 代码块内部 `arg` 的类型是 `string`。
这使我们能够直接访问 `toUpperCase()` 等 `string` 方法而不会报错。

然而，如果我们把该条件提取到一个名为 `argIsString` 的常量中，会发生什么呢？

```ts
// 在 TS 4.3 及更早版本中

function foo(arg: unknown) {
  const argIsString = typeof arg === 'string'
  if (argIsString) {
    console.log(arg.toUpperCase())
    //              ~~~~~~~~~~~
    // 错误！类型 'unknown' 上不存在属性 'toUpperCase'。
  }
}
```

在 TypeScript 的早期版本中，这会导致错误——即使 `argIsString` 被赋值为一个类型守卫的结果，TypeScript 也会直接丢失该信息。
这很不方便，因为我们可能希望在多处复用同一个检查。
为了绕过这一限制，开发者往往不得不重复编写检查条件，或者使用类型断言（即类型转换）。

在 TypeScript 4.4 中，这种情况不复存在。
上面的示例现在完全可以正常工作且不会报错！
当 TypeScript 发现我们在测试一个常量值时，它会做一些额外的工作来检查该常量是否包含类型守卫。
如果该类型守卫作用于 `const` 常量、`readonly` 只读属性或未被修改的参数，那么 TypeScript 就能对该值进行恰当的类型窄化。

不仅是 `typeof` 检查，各类不同的类型守卫条件均得以保留。
例如，对可辨识联合的检查也能完美支持：

```ts twoslash
type Shape =
  { kind: 'circle'; radius: number } | { kind: 'square'; sideLength: number }

function area(shape: Shape): number {
  const isCircle = shape.kind === 'circle'
  if (isCircle) {
    // 此时我们确定这是一个圆形！
    return Math.PI * shape.radius ** 2
  } else {
    // 此时我们确定剩下的是正方形！
    return shape.sideLength ** 2
  }
}
```

TypeScript 4.4 对判别属性的分析也更进了一步——我们现在可以解构提取出判别属性，TypeScript 依然能够窄化原始对象：

```ts twoslash
type Shape =
  { kind: 'circle'; radius: number } | { kind: 'square'; sideLength: number }

function area(shape: Shape): number {
  // 先提取出 'kind' 字段。
  const { kind } = shape

  if (kind === 'circle') {
    // 此时我们确定这是一个圆形！
    return Math.PI * shape.radius ** 2
  } else {
    // 此时我们确定剩下的是正方形！
    return shape.sideLength ** 2
  }
}
```

再举一个例子，下面这个函数用于检查它的两个入参是否都存在内容：

```ts twoslash
function doSomeChecks(
  inputA: string | undefined,
  inputB: string | undefined,
  shouldDoExtraWork: boolean,
) {
  const mustDoWork = inputA && inputB && shouldDoExtraWork
  if (mustDoWork) {
    // 我们可以同时访问 'inputA' 和 'inputB' 上的 'string' 属性！
    const upperA = inputA.toUpperCase()
    const upperB = inputB.toUpperCase()
    // ...
  }
}
```

TypeScript 能够推断出如果 `mustDoWork` 为 `true`，则 `inputA` 和 `inputB` 必定同时存在。
这意味着我们不必编写类似 `inputA!` 的非空断言来向 TypeScript 证明 `inputA` 不是 `undefined`。

这里一个很棒的特性是：这种分析是传递性的。
TypeScript 会跨多个常量进行追踪，以了解你之前执行过哪些检查：

<!-- prettier-ignore -->
```ts twoslash
function f(x: string | number | boolean) {
  const isString = typeof x === "string";
  const isNumber = typeof x === "number";
  const isStringOrNumber = isString || isNumber;
  if (isStringOrNumber) {
    x;
//  ^?
  } else {
    x;
//  ^?
  }
}
```

需要注意的是，这种分析存在深度限制——TypeScript 在检查这些条件时不会无限深入，但其分析深度足以应对大多数场景。

该特性能够让许多符合直觉的 JavaScript 代码在 TypeScript 中“直接可用”，不再受类型系统的阻碍。
更多详细信息，请查看 [GitHub 上的实现 PR](https://github.com/microsoft/TypeScript/pull/44730)！

## Symbol 与模板字符串模式索引签名

TypeScript 允许我们使用*索引签名*（index signatures）来描述所有属性都必须具有某种类型的对象。
这使我们能够将这些对象当作类似字典的类型使用，通过中括号配合字符串键进行索引访问。

例如，我们可以编写一个带有索引签名的类型，它接收 `string` 类型的键并映射到 `boolean` 类型的值。
如果我们尝试赋予除 `boolean` 以外的任何值，都会报错：

```ts twoslash
// @errors: 2322 2375
interface BooleanDictionary {
  [key: string]: boolean
}

declare let myDict: BooleanDictionary

// 允许赋值布尔值
myDict['foo'] = true
myDict['bar'] = false

// 错误，"oops" 不是布尔值
myDict['baz'] = 'oops'
```

尽管在此类场景中 [使用 `Map` 可能是更合适的数据结构](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)（具体而言是 `Map<string, boolean>`），但在很多情况下，JavaScript 对象使用起来往往更便捷，或者正是我们接收到的数据格式。

类似地，`Array<T>` 也已经定义了一个 `number` 索引签名，允许我们存取 `T` 类型的值：

```ts
// @errors: 2322 2375
// 这是 TypeScript 内置 Array 类型定义的一部分。
interface Array<T> {
  [index: number]: T

  // ...
}

let arr = new Array<string>()

// 有效
arr[0] = 'hello!'

// 错误，此处期望一个 'string' 值
arr[1] = 123
```

索引签名在表达实际代码中非常有用；
然而，此前它们仅限于 `string` 和 `number` 类型的键（而且 `string` 索引签名还有一个特意的设计，即允许接收 `number` 键，因为它们无论如何都会被强制转换为字符串）。
这意味着 TypeScript 不允许使用 `symbol` 键来索引对象。
TypeScript 也无法表达某些 `string` 键的*子集*索引签名——例如，仅描述名称以 `data-` 开头的属性的索引签名。

TypeScript 4.4 解决了这些局限，支持了针对 `symbol` 和模板字符串模式的索引签名。

例如，TypeScript 现在允许我们声明一个可以使用任意 `symbol` 作为键的类型：

```ts twoslash
// @errors: 2322 2375
interface Colors {
  [sym: symbol]: number
}

const red = Symbol('red')
const green = Symbol('green')
const blue = Symbol('blue')

let colors: Colors = {}

// 允许赋值数字
colors[red] = 255
let redVal = colors[red]
//  ^?

colors[blue] = 'da ba dee'
```

同样地，我们也可以编写带有模板字符串模式类型的索引签名。
它的一个用例是使以 `data-` 开头的属性免受 TypeScript 额外属性检查的影响。
当我们把对象字面量传递给期望特定类型的参数时，TypeScript 会查找未在期望类型中声明的额外属性：

```ts
// @errors: 2322 2375
interface Options {
  width?: number
  height?: number
}

let a: Options = {
  width: 100,
  height: 100,

  'data-blah': true,
}

interface OptionsWithDataProps extends Options {
  // 允许以 'data-' 开头的任何属性。
  [optName: `data-${string}`]: unknown
}

let b: OptionsWithDataProps = {
  width: 100,
  height: 100,
  'data-blah': true,

  // 对于既非已知属性，也不以
  // 'data-' 开头的属性，将会报错
  'unknown-property': true,
}
```

关于索引签名的最后一点说明是，它们现在允许使用联合类型，只要这些联合类型是由具有无限值域的原始类型组成的即可——具体包括：

- `string`
- `number`
- `symbol`
- 模板字符串模式（例如 `` `hello-${string}` ``）

参数为这些类型联合的索引签名将被脱糖（de-sugar）为几个不同的索引签名：

```ts
interface Data {
  [optName: string | symbol]: any
}

// 等同于

interface Data {
  [optName: string]: any
  [optName: symbol]: any
}
```

更多详细信息，请参阅 [该 Pull Request](https://github.com/microsoft/TypeScript/pull/44512)。

## Catch 变量中默认使用 `unknown` 类型（`--useUnknownInCatchVariables`）

在 JavaScript 中，任何类型的值都可以通过 `throw` 抛出并在 `catch` 子句中捕获。
正因如此，TypeScript 在历史上一直将 catch 子句中的变量类型设为 `any`，且不允许任何其他类型注解：

```ts
try {
  // 谁知道这可能会抛出什么...
  executeSomeThirdPartyCode()
} catch (err) {
  // err: any
  console.error(err.message) // 允许，因为是 'any'
  err.thisWillProbablyFail() // 允许，因为是 'any' :(
}
```

在 TypeScript 引入 `unknown` 类型后，对于追求最高代码正确性与类型安全的用户而言，在 `catch` 子句变量中使用 `unknown` 显然比 `any` 是更好的选择，因为它能够更好地进行类型窄化，并迫使我们针对任意值进行检查。
最终，TypeScript 4.0 允许用户在每个 `catch` 子句变量上显式指定 `unknown`（或 `any`）类型注解，以便按需开启更严格的类型检查；
然而对某些开发者来说，在每个 `catch` 子句上手动标注 `: unknown` 依然是一件繁琐的事。

因此，TypeScript 4.4 引入了一个名为 [`useUnknownInCatchVariables`](/tsconfig#useUnknownInCatchVariables) 的新标志。
该标志将 `catch` 子句变量的默认类型从 `any` 更改为 `unknown`。

```ts twoslash
// @errors: 2571 18046
declare function executeSomeThirdPartyCode(): void
// ---cut---
try {
  executeSomeThirdPartyCode()
} catch (err) {
  // err: unknown

  // 错误！类型 'unknown' 上不存在属性 'message'。
  console.error(err.message)

  // 正常工作！我们可以将 'err' 从 'unknown' 收窄为 'Error'。
  if (err instanceof Error) {
    console.error(err.message)
  }
}
```

该标志在 [`strict`](/tsconfig#strict) 严格模式选项族下默认启用。
这意味着如果你使用 [`strict`](/tsconfig#strict) 检查代码，该选项将自动开启。
在 TypeScript 4.4 中，你可能会遇到如下错误：

```
Property 'message' does not exist on type 'unknown'.
Property 'name' does not exist on type 'unknown'.
Property 'stack' does not exist on type 'unknown'.
```

如果我们不想在 `catch` 子句中处理 `unknown` 变量，随时可以通过添加显式的 `: any` 注解来选择*停用*更严格的类型：

<!-- prettier-ignore -->
```ts twoslash
declare function executeSomeThirdPartyCode(): void;
// ---cut---
try {
  executeSomeThirdPartyCode();
} catch (err: any) {
  console.error(err.message); // 再次正常工作！
}
```

更多信息请参阅 [实现该特性的 Pull Request](https://github.com/microsoft/TypeScript/pull/41013)。

## 精确的可选属性类型（`--exactOptionalPropertyTypes`）

在 JavaScript 中，读取对象上*缺失*的属性会得到 `undefined` 值。
同时，对象也有可能*真正包含*一个值为 `undefined` 的属性。
JavaScript 中的很多代码往往对这两种情况不加区分，因此最初 TypeScript 只是将每个可选属性都解释为用户在类型中写了 `undefined`。
例如：

```ts
interface Person {
  name: string
  age?: number
}
```

被视同为：

```ts
interface Person {
  name: string
  age?: number | undefined
}
```

这意味着用户可以显式地将 `undefined` 赋给 `age`：

```ts
const p: Person = {
  name: 'Daniel',
  age: undefined, // 默认情况下这是允许的。
}
```

因此在默认情况下，TypeScript 并不区分“值为 `undefined` 的已存在属性”与“缺失的属性”。
虽然这在大多数情况下行得通，但并非所有 JavaScript 代码都基于相同的假设。
像 `Object.assign`、`Object.keys`、对象展开（`{ ...obj }`）以及 `for`-`in` 循环等函数和运算符的行为，都会根据属性是否实际存在于对象上而有所不同。
在我们的 `Person` 示例中，如果观察 `age` 属性的上下文关注该属性是否存在，这可能会潜在地导致运行时错误。

在 TypeScript 4.4 中，新的 [`exactOptionalPropertyTypes`](/tsconfig#exactOptionalPropertyTypes) 标志规定可选属性类型应严格按照书写方式解释，这意味着类型中不会自动添加 `| undefined`：

```ts twoslash
// @exactOptionalPropertyTypes
// @errors: 2322 2375
interface Person {
  name: string
  age?: number
}
// ---cut---
// 启用 'exactOptionalPropertyTypes' 后：
const p: Person = {
  name: 'Daniel',
  age: undefined, // 错误！undefined 不是数字
}
```

该标志**不属于** [`strict`](/tsconfig#strict) 严格模式选项族，如果你希望启用该行为，需要显式开启。
它还要求同时启用 [`strictNullChecks`](/tsconfig#strictNullChecks)。
我们已经对 DefinitelyTyped 和其他类型定义进行了更新，以尽量保证迁移的平滑过渡，但根据你的代码组织方式，可能会遇到一些需要适配的情况。

更多信息请参阅 [实现该特性的 Pull Request](https://github.com/microsoft/TypeScript/pull/43947)。

## 类中的 `static` 块

TypeScript 4.4 带来了对 [类中 `static` 块](https://github.com/tc39/proposal-class-static-block#ecmascript-class-static-initialization-blocks) 的支持，这是一项 ECMAScript 特性，可帮助你为静态成员编写更复杂的初始化逻辑。

```ts twoslash
declare function someCondition(): boolean
// ---cut---
class Foo {
  static count = 0

  // 这是一个静态块：
  static {
    if (someCondition()) {
      Foo.count++
    }
  }
}
```

这些静态块允许你编写具有独立作用域的一系列语句，并且可以访问所在类中的私有字段。
这意味着我们可以充分利用语句的所有能力来编写初始化代码，既不会泄漏变量，又能完全访问类的内部细节：

```ts twoslash
declare function loadLastInstances(): any[]
// ---cut---
class Foo {
  static #count = 0

  get count() {
    return Foo.#count
  }

  static {
    try {
      const lastInstances = loadLastInstances()
      Foo.#count += lastInstances.length
    } catch {}
  }
}
```

在没有 `static` 块之前，编写上述代码虽然可行，但通常需要采用各种各样的变通方式（hack），并且往往需要作出某种妥协。

请注意，一个类可以拥有多个 `static` 块，它们会按照书写顺序依次执行：

```ts twoslash
// 输出：
//    1
//    2
//    3
class Foo {
  static prop = 1
  static {
    console.log(Foo.prop++)
  }
  static {
    console.log(Foo.prop++)
  }
  static {
    console.log(Foo.prop++)
  }
}
```

我们由衷感谢 [王文璐 (Wenlu Wang)](https://github.com/Kingwl) 为 TypeScript 实现该特性所作的贡献。
更多详细信息，你可以查看 [该 Pull Request](https://github.com/microsoft/TypeScript/pull/43370)。

## `tsc --help` 更新与改进

TypeScript 的 `--help` 选项迎来了焕新！
部分得益于 [Song Gao](https://github.com/ShuiRuTian) 的工作，我们引入了一系列改进以 [更新编译器选项的描述](https://github.com/microsoft/TypeScript/pull/44409)，并通过颜色和视觉分隔符 [重新设计了 `--help` 菜单的样式](https://github.com/microsoft/TypeScript/pull/44157)。

![全新的 TypeScript `--help` 菜单，输出内容被划分到几个不同的区域](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2021/08/tsc-help-ps-wt-4-4.png)

你可以阅读 [原始提案讨论帖](https://github.com/microsoft/TypeScript/issues/44074) 了解更多内容。

## 性能提升

### 更快的声明文件生成

TypeScript 现在会缓存内部符号在不同上下文中是否可访问，以及特定类型应如何输出。
这些改动可以提升 TypeScript 在处理包含较为复杂类型的代码时的整体性能，在启用 [`declaration`](/tsconfig#declaration) 标志生成 `.d.ts` 文件时尤为明显。

[查看更多详细信息](https://github.com/microsoft/TypeScript/pull/43973)。

### 更快的路径规范化

TypeScript 经常需要对文件路径进行多种“规范化”（normalization）处理，使其转换为编译器各处通用的统一格式。
这涉及将反斜杠替换为正斜杠，或者移除路径中多余的 `/./` 和 `/../` 片段。
当 TypeScript 必须处理数百万个此类路径时，这些操作最终会带来一定的耗时。
在 TypeScript 4.4 中，路径首先会经过快速检查，判断其本身是否根本不需要任何规范化处理。
这些改进共同使大型项目的项目加载时间缩短了 5% 到 10%，而在我们内部测试的超大型项目中，提升更为显著。

更多详细信息，你可以查看 [路径片段规范化的 PR](https://github.com/microsoft/TypeScript/pull/44173) 以及 [斜杠规范化的 PR](https://github.com/microsoft/TypeScript/pull/44100)。

### 更快的路径映射

TypeScript 现在会缓存构建路径映射的方式（通过 `tsconfig.json` 中的 [`paths`](/tsconfig#paths) 选项）。
对于包含数百个映射的项目，耗时减少十分明显。
你可以参阅 [该改动本身](https://github.com/microsoft/TypeScript/pull/44078) 了解更多信息。

### 启用 `--strict` 时更快的增量构建

此前存在一个实际上的 bug：在开启 [`strict`](/tsconfig#strict) 时，TypeScript 会在 [`incremental`](/tsconfig#incremental) 增量编译下重复执行类型检查工作。
这导致许多构建过程的耗时与关闭 [`incremental`](/tsconfig#incremental) 时一样缓慢。
TypeScript 4.4 修复了此问题，并且该改动也已反向移植到 TypeScript 4.3 中。

在此处查看 [更多信息](https://github.com/microsoft/TypeScript/pull/44394)。

### 大型输出更快的 Source Map 生成

TypeScript 4.4 针对超大输出文件的 Source Map 生成进行了优化。
在构建较旧版本的 TypeScript 编译器时，这使得生成（emit）耗时减少了约 8%。

我们由衷感谢 [David Michon](https://github.com/dmichon-msft)，他提供了一个 [简洁明了的改动](https://github.com/microsoft/TypeScript/pull/44031) 实现了这一性能收益。

### 更快的 `--force` 构建

在项目引用上使用 `--build` 模式时，TypeScript 必须执行时效性检查（up-to-date checks）以确定哪些文件需要重新构建。
然而在执行 [`--force`](/tsconfig#force) 构建时，由于每个项目依赖都将从头重新构建，这些信息是无关紧要的。
在 TypeScript 4.4 中，[`--force`](/tsconfig#force) 构建会跳过这些不必要的步骤并直接开始完整构建。
在此处查看 [有关该改动的更多信息](https://github.com/microsoft/TypeScript/pull/43666)。

## 针对 JavaScript 的拼写建议

TypeScript 为 Visual Studio 和 Visual Studio Code 等编辑器中的 JavaScript 编辑体验提供支持。
大多数情况下，TypeScript 尽量在 JavaScript 文件中保持克制；
然而，TypeScript 往往拥有足够的信息来提供高可信度的建议，并能以不太突兀的方式展示这些建议。

正因如此，TypeScript 现在会在普通 JavaScript 文件（未添加 `// @ts-check` 或项目中关闭了 [`checkJs`](/tsconfig#checkJs) 的文件）中提供拼写建议。
这些建议与 TypeScript 文件中已有的 _“Did you mean...?”_（你是不是指……？）建议相同，现在它们以某种形式普及到了*所有* JavaScript 文件中。

这些拼写建议可以为你的代码错误提供微妙的线索。
我们在测试该特性时，甚至在现有代码中发现了几处 bug！

关于该新特性的更多详细信息，请参阅 [该 Pull Request](https://github.com/microsoft/TypeScript/pull/44271)！

## 内联提示（Inlay Hints）

TypeScript 4.4 支持了*内联提示*（inlay hints），可以在你的代码中显示参数名称和返回类型等有用信息。
你可以将其视为一种友好的“幽灵文本”（ghost text）。

![Visual Studio Code 中的内联提示预览](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2021/08/inlayHints-4.4-rc-ghd.png)

该特性由 [王文璐 (Wenlu Wang)](https://github.com/Kingwl) 开发，其 [Pull Request](https://github.com/microsoft/TypeScript/pull/42089) 提供了更多细节。

王文璐还贡献了 [Visual Studio Code 中的内联提示集成](https://github.com/microsoft/vscode/pull/113412)，该集成已随 [2021 年 7 月发布的 1.59 版本](https://code.visualstudio.com/updates/v1_59#_typescript-44) 一同提供。
如果你想体验内联提示，请确保使用较新的 [稳定版](https://code.visualstudio.com/updates/v1_59) 或 [Insiders 预览版](https://code.visualstudio.com/insiders/) 编辑器。
你还可以在 Visual Studio Code 的设置中调整内联提示的显示时机和位置。

## 补全列表中自动导入显示真实路径

当 Visual Studio Code 等编辑器显示补全列表时，包含自动导入的补全项会显示对应模块的路径；
然而，该路径通常并不是 TypeScript 最终写入模块说明符（module specifier）中的路径。
该路径通常是相对于*工作区*的路径，这意味着如果你从 `moment` 这样的包导入，通常会看到类似 `node_modules/moment` 的路径。

![包含冗长 'node_modules' 路径的补全列表。例如，'calendarFormat' 的标签是 'node_modules/moment/moment' 而不是 'moment'](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2021/08/completion-import-labels-pre-4-4.png)

这些路径往往显得繁琐且容易引起误解，尤其是考虑到实际插入文件中的路径还需要结合 Node 的 `node_modules` 解析、路径映射、符号链接以及重新导出等机制。

正因如此，在 TypeScript 4.4 中，补全项标签现在会显示将用于导入的*实际*模块路径！

![包含清晰路径且没有中间 'node_modules' 的补全列表。例如，'calendarFormat' 的标签是 'moment' 而不是 'node_modules/moment/moment'](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2021/08/completion-import-labels-4-4.png)

由于此计算可能开销较大，包含大量自动导入的补全列表可能会在你输入更多字符时分批填入最终的模块说明符。你可能仍会偶尔看到旧的工作区相对路径标签；但随着编辑体验的“预热”，在你继续输入一两个字符后，它们就会被替换为实际路径。

## 破坏性变更

### TypeScript 4.4 的 `lib.d.ts` 变更

与每个 TypeScript 版本一样，`lib.d.ts` 的声明（特别是为 Web 上下文生成的声明）发生了一些改动。
你可以查阅 [已知 `lib.dom.d.ts` 变更列表](https://github.com/microsoft/TypeScript-DOM-lib-generator/issues/1029#issuecomment-869224737) 来了解受影响的内容。

### 更符合规范的导入函数间接调用

在早期版本的 TypeScript 中，从 CommonJS、AMD 以及其他非 ES 模块系统调用导入项时，会设置被调用函数的 `this` 值。
具体来说，在以下示例中，当调用 `fooModule.foo()` 时，`foo()` 方法会将 `this` 的值设置为 `fooModule`。

```ts
// 假设这是我们导入的模块，且它包含一个名为 'foo' 的导出。
let fooModule = {
  foo() {
    console.log(this)
  },
}

fooModule.foo()
```

但这并不是 ECMAScript 导出函数在被调用时应当遵循的方式。
因此，TypeScript 4.4 在调用导入函数时特意通过以下代码生成方式丢弃了 `this` 值：

```ts
// 假设这是我们导入的模块，且它包含一个名为 'foo' 的导出。
let fooModule = {
  foo() {
    console.log(this)
  },
}

// 注意我们现在实际上调用的是 '(0, fooModule.foo)'，这有着细微的差别。
;(0, fooModule.foo)()
```

你可以 [在此处阅读有关该改动的更多信息](https://github.com/microsoft/TypeScript/pull/44624)。

### 在 Catch 变量中使用 `unknown`

使用 [`strict`](/tsconfig#strict) 标志的用户可能会看到由于 `catch` 变量为 `unknown` 而产生的新错误，特别是当现有代码假定仅捕获到 `Error` 值时。
这通常会导致如下错误信息：

```
Property 'message' does not exist on type 'unknown'.
Property 'name' does not exist on type 'unknown'.
Property 'stack' does not exist on type 'unknown'.
```

为了解决这个问题，你可以特意添加运行时检查，以确保抛出的类型与你的预期类型匹配。
或者，你也可以直接使用类型断言、为 catch 变量显式添加 `: any`，或者关闭 [`useUnknownInCatchVariables`](/tsconfig#useUnknownInCatchVariables)。

### 更广泛的始终为真 Promise 检查

在之前的版本中，TypeScript 引入了“始终为真的 Promise 检查”，以捕获可能遗漏 `await` 的代码；
然而，该检查此前仅适用于具名声明。
这意味着以下代码能够正确报错……

```ts
async function foo(): Promise<boolean> {
  return false
}

async function bar(): Promise<string> {
  const fooResult = foo()
  if (fooResult) {
    // <- 错误！:D
    return 'true'
  }
  return 'false'
}
```

……但以下代码却不会报错：

```ts
async function foo(): Promise<boolean> {
  return false
}

async function bar(): Promise<string> {
  if (foo()) {
    // <- 无错误 :(
    return 'true'
  }
  return 'false'
}
```

TypeScript 4.4 现在对这两种情况都会报告错误。
更多信息请参阅 [原始改动](https://github.com/microsoft/TypeScript/pull/44491)。

### 抽象属性不允许初始值设定项

以下代码现在会报错，因为抽象属性不能具有初始值设定项：

```ts
abstract class C {
  abstract prop = 1;
  //       ~~~~
  // 属性 'prop' 不能有初始化器，因为它被标记为 abstract。
}
```

取而代之的是，你只能为该属性指定类型：

```ts
abstract class C {
  abstract prop: number
}
```
