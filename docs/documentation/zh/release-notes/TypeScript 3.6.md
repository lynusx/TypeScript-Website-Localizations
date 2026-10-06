---
title: TypeScript 3.6
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-3-6.html
oneline: TypeScript 3.6 发布说明
---

## 更严格的生成器（Generators）检查

TypeScript 3.6 引入了对迭代器和生成器函数的更严格检查。
在早期版本中，生成器的使用者无法区分某个值是通过 `yield` 产出的还是通过 `return` 返回的。

```ts
function* foo() {
  if (Math.random() < 0.5) yield 100
  return 'Finished!'
}

let iter = foo()
let curr = iter.next()
if (curr.done) {
  // TypeScript 3.5 and prior thought this was a 'string | number'.
  // It should know it's 'string' since 'done' was 'true'!
  curr.value
}
```

此外，生成器过去总是假定 `yield` 的类型为 `any`。

```ts
function* bar() {
  let x: { hello(): void } = yield
  x.hello()
}

let iter = bar()
iter.next()
iter.next(123) // oops! runtime error!
```

在 TypeScript 3.6 中，检查器现在知道在第一个示例中 `curr.value` 的正确类型应为 `string`，并且在最后一个示例中调用 `next()` 时会正确报错。
这得益于对 `Iterator` 和 `IteratorResult` 类型声明所做的一些更改（引入了几个新的类型参数），以及 TypeScript 用于表示生成器的新类型——`Generator` 类型。

现在，`Iterator` 类型允许用户指定产出值的类型、返回值类型以及 `next` 可以接收的类型。

```ts
interface Iterator<T, TReturn = any, TNext = undefined> {
  // Takes either 0 or 1 arguments - doesn't accept 'undefined'
  next(...args: [] | [TNext]): IteratorResult<T, TReturn>
  return?(value?: TReturn): IteratorResult<T, TReturn>
  throw?(e?: any): IteratorResult<T, TReturn>
}
```

在此基础上，新的 `Generator` 类型是一种始终同时具备 `return` 和 `throw` 方法的 `Iterator`，并且它本身也是可迭代的。

```ts
interface Generator<
  T = unknown,
  TReturn = any,
  TNext = unknown,
> extends Iterator<T, TReturn, TNext> {
  next(...args: [] | [TNext]): IteratorResult<T, TReturn>
  return(value: TReturn): IteratorResult<T, TReturn>
  throw(e: any): IteratorResult<T, TReturn>
  [Symbol.iterator](): Generator<T, TReturn, TNext>
}
```

为了能够区分返回值与产出值，TypeScript 3.6 将 `IteratorResult` 类型改造成了可辨识联合类型（discriminated union type）：

```ts
type IteratorResult<T, TReturn = any> =
  IteratorYieldResult<T> | IteratorReturnResult<TReturn>

interface IteratorYieldResult<TYield> {
  done?: false
  value: TYield
}

interface IteratorReturnResult<TReturn> {
  done: true
  value: TReturn
}
```

简而言之，这意味着在直接处理来自迭代器的值时，你将能够对它们进行适当的类型收窄。

为了准确表示通过调用 `next()` 传入生成器的参数类型，TypeScript 3.6 还会推断生成器函数体内对 `yield` 的特定用法。

```ts
function* foo() {
  let x: string = yield
  console.log(x.toUpperCase())
}

let x = foo()
x.next() // first call to 'next' is always ignored
x.next(42) // error! 'number' is not assignable to 'string'
```

如果你希望更加明确，还可以使用显式返回类型来约束可以返回、产出以及从 `yield` 表达式中计算求出的值的类型。
在下例中，`next()` 只能传入 `boolean` 类型参数，且取决于 `done` 的值，`value` 要么是 `string` 要么是 `number`。

```ts
/**
 * - yields numbers
 * - returns strings
 * - can be passed in booleans
 */
function* counter(): Generator<number, string, boolean> {
  let i = 0
  while (true) {
    if (yield i++) {
      break
    }
  }
  return 'done!'
}

var iter = counter()
var curr = iter.next()
while (!curr.done) {
  console.log(curr.value)
  curr = iter.next(curr.value === 5)
}
console.log(curr.value.toUpperCase())

// prints:
//
// 0
// 1
// 2
// 3
// 4
// 5
// DONE!
```

有关该改动的更多详情，[请参阅此处的 Pull Request](https://github.com/Microsoft/TypeScript/issues/2983)。

## 更精准的数组展开

在低于 ES2015 的目标版本中，对 `for`/`of` 循环和数组展开等结构的最精确编译产物可能会相对繁重。
出于这个原因，TypeScript 默认采用更轻量的编译产物，该产物仅支持数组类型，而在需要对其他类型进行迭代时，则通过 [`downlevelIteration`](/tsconfig#downlevelIteration) 标志提供支持。
未启用 [`downlevelIteration`](/tsconfig#downlevelIteration) 时的宽松默认行为通常工作得很好；但在某些常见情况下，数组展开的转换结果存在明显的行为差异。
例如，以下包含展开操作的数组：

```ts
;[...Array(5)]
```

可以重写为如下数组字面量：

```js
;[undefined, undefined, undefined, undefined, undefined]
```

然而，TypeScript 此前会将原始代码转换为以下代码：

```ts
Array(5).slice()
```

这两者存在细微差异。
`Array(5)` 会生成一个长度为 5 但没有已定义属性槽位（slot）的数组。

TypeScript 3.6 引入了全新的 `__spreadArrays` 辅助函数，以便在未开启 [`downlevelIteration`](/tsconfig#downlevelIteration) 时，在旧目标版本中准确模拟 ECMAScript 2015 的行为。
`__spreadArrays` 也在 [tslib](https://github.com/Microsoft/tslib/) 中提供。

更多信息，[请参阅相关 Pull Request](https://github.com/microsoft/TypeScript/pull/31166)。

## 改进关于 Promise 的使用体验

TypeScript 3.6 针对错误使用 `Promise` 的情况做出了若干改进。

例如，在将 `Promise` 的内容传递给另一个函数之前，忘记使用 `.then()` 或 `await` 是非常常见的错误。
TypeScript 的错误消息现在针对该情况进行了专门优化，会提示用户也许应该考虑使用 `await` 关键字。

```ts
interface User {
  name: string
  age: number
  location: string
}

declare function getUserData(): Promise<User>
declare function displayUser(user: User): void

async function f() {
  displayUser(getUserData())
  //              ~~~~~~~~~~~~~
  // Argument of type 'Promise<User>' is not assignable to parameter of type 'User'.
  //   ...
  // Did you forget to use 'await'?
}
```

同样，在对 `Promise` 执行 `await` 或 `.then()` 之前就尝试访问其上的方法也很常见。
这也是我们能够做得更好的另一个例子。

```ts
async function getCuteAnimals() {
  fetch('https://reddit.com/r/aww.json').json()
  //   ~~~~
  // Property 'json' does not exist on type 'Promise<Response>'.
  //
  // Did you forget to use 'await'?
}
```

更多详情，[请参阅原始 Issue](https://github.com/microsoft/TypeScript/issues/30646) 以及关联的各个 Pull Request。

## 标识符更好的 Unicode 支持

在输出目标为 ES2015 及更高版本时，TypeScript 3.6 对标识符中的 Unicode 字符提供了更好的支持。

```ts
const 𝓱𝓮𝓵𝓵𝓸 = 'world' // previously disallowed, now allowed in '--target es2015'
```

## SystemJS 中的 `import.meta` 支持

当 `module` 目标设置为 `system` 时，TypeScript 3.6 支持将 `import.meta` 转换为 `context.meta`。

```ts
// This module:

console.log(import.meta.url)

// gets turned into the following:

System.register([], function (exports, context) {
  return {
    setters: [],
    execute: function () {
      console.log(context.meta.url)
    },
  }
})
```

## 环境上下文中允许使用 `get` 和 `set` 访问器

在之前的 TypeScript 版本中，语言不允许在环境上下文（ambient contexts，如 `declare` 声明的类中，或普通的 `.d.ts` 文件中）中使用 `get` 和 `set` 访问器。
这样做的理由是，就读写这些属性而言，访问器与普通属性并没有区别；
然而，[由于 ECMAScript 的类字段提案可能与现有的 TypeScript 行为存在差异](https://github.com/tc39/proposal-class-fields/issues/248)，我们意识到需要一种方式来传达这种差异，以便在子类中提供恰当的错误提示。

因此，在 TypeScript 3.6 中，用户可以在环境上下文中编写 getter 和 setter。

```ts
declare class Foo {
  // Allowed in 3.6+.
  get x(): number
  set x(val: number)
}
```

在 TypeScript 3.7 中，编译器自身也将利用该特性，使得生成的 `.d.ts` 文件同样会输出 `get`/`set` 访问器。

## 环境类与环境函数可以合并

在之前的 TypeScript 版本中，任何情况下合并类与函数都会报错。
现在，环境类与环境函数（带有 `declare` 修饰符的类/函数，或位于 `.d.ts` 文件中的类/函数）可以进行合并。
这意味着你现在可以这样写：

```ts
export declare function Point2D(x: number, y: number): Point2D
export declare class Point2D {
  x: number
  y: number
  constructor(x: number, y: number)
}
```

而不再需要写成：

```ts
export interface Point2D {
  x: number
  y: number
}
export declare var Point2D: {
  (x: number, y: number): Point2D
  new (x: number, y: number): Point2D
}
```

这样做的一个优势是，可以轻松表达可调用的构造函数模式（callable constructor pattern），同时也允许命名空间与这些声明进行合并（因为 `var` 声明无法与 `namespace` 合并）。

在 TypeScript 3.7 中，编译器将利用此特性，使得从 `.js` 文件生成的 `.d.ts` 文件能够准确记录类函数（class-like function）的可调用性（callability）和可构造性（constructability）。

更多详情，[请参阅 GitHub 上的原始 PR](https://github.com/microsoft/TypeScript/pull/32584)。

## 支持 `--build` 和 `--incremental` 的 API

TypeScript 3.0 引入了使用 `--build` 标志引用其他项目并进行增量构建的支持。
此外，TypeScript 3.4 引入了 [`incremental`](/tsconfig#incremental) 标志，用于保存先前编译的信息以便仅重新构建特定文件。
这些标志在更灵活地组织项目结构和加快构建速度方面极为有用。
遗憾的是，这些标志无法直接与 Gulp 和 Webpack 等第三方构建工具配合使用。
TypeScript 3.6 现在公开了两套 API，用于操作项目引用和增量程序构建。

为了创建 [`incremental`](/tsconfig#incremental) 构建，用户可以使用 `createIncrementalProgram` 和 `createIncrementalCompilerHost` API。
用户还可以通过新公开的 `readBuilderProgram` 函数，从该 API 生成的 `.tsbuildinfo` 文件中重新水合（re-hydrate）旧的程序实例；该函数仅用于创建新程序（即你不能修改返回的实例——它仅用于传递给其他 `create*Program` 函数中的 `oldProgram` 参数）。

为了利用项目引用，新增公开了 `createSolutionBuilder` 函数，该函数返回新类型 `SolutionBuilder` 的实例。

有关这些 API 的更多详情，[请参阅原始 Pull Request](https://github.com/microsoft/TypeScript/pull/31432)。

## 感知分号的代码编辑

Visual Studio 和 Visual Studio Code 等编辑器可以自动应用快速修复、重构以及其他转换（例如从其他模块自动导入值）。
这些转换由 TypeScript 提供底层支持，而旧版 TypeScript 会无条件地在每个语句末尾添加分号；不幸的是，这与许多用户的代码风格规范相冲突，很多用户对编辑器强行插入分号感到不满。

TypeScript 现在足够智能，可以在应用此类编辑时检测你的文件是否使用分号。
如果你的文件普遍没有分号，TypeScript 就不会添加分号。

更多详情，[请参阅对应的 Pull Request](https://github.com/microsoft/TypeScript/pull/31801)。

## 更智能的自动导入语法

JavaScript 拥有许多不同的模块语法或规范：ECMAScript 标准语法、Node 原生支持的语法（CommonJS）、AMD、System.js 等等！
在大多数情况下，TypeScript 默认会使用 ECMAScript 模块语法进行自动导入，但这在某些具有不同编译器设置的 TypeScript 项目中，或者在使用纯 JavaScript 和 `require` 调用的 Node 项目中往往并不合适。

TypeScript 3.6 现在在决定如何自动导入其他模块之前，会更加智能地分析现有的导入语句。
[在此处的原始 Pull Request 中可以查看更多详情](https://github.com/microsoft/TypeScript/pull/32684)。

## 全新的 TypeScript 演练场（Playground）

TypeScript Playground 迎来了久违的革新，并带来了许多便捷的新功能！
新的演练场很大程度上基于 [Artem Tyurin](https://github.com/agentcooper) 开发的 [TypeScript playground](https://github.com/agentcooper/typescript-play) 进行派生开发，社区成员之前对该项目的使用越来越广泛。
非常感谢 Artem 在此提供的帮助！

新的演练场现在支持许多新选项，包括：

- [`target`](/tsconfig#target) 选项（允许用户从 `es5` 切换到 `es3`、`es2015`、`esnext` 等）
- 所有严格性标志（包括仅设置 [`strict`](/tsconfig#strict)）
- 支持纯 JavaScript 文件（通过 `allowJS` 以及可选的 [`checkJs`](/tsconfig#checkJs)）

这些选项在分享演练场示例链接时也会持久保留，让用户可以更可靠地分享示例，而无需向接收者交代“哦，别忘了开启 [`noImplicitAny`](/tsconfig#noImplicitAny) 选项！”。

在不久的将来，我们将更新演练场示例、添加 JSX 支持并优化自动类型获取（automatic type acquisition），这意味着你在演练场上将能获得与个人编辑器完全一致的体验。

随着我们不断改进演练场和官方网站，[非常欢迎大家在 GitHub 上提出反馈和 Pull Request](https://github.com/microsoft/TypeScript-Website/)！
