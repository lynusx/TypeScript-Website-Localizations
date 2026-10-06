---
title: TypeScript 4.5
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-4-5.html
oneline: TypeScript 4.5 发布说明
---

## 支持从 `node_modules` 引入 `lib`

为了确保对 TypeScript 和 JavaScript 提供良好的开箱即用支持，TypeScript 内置打包了一系列声明文件（`.d.ts` 文件）。
这些声明文件代表了 JavaScript 语言中的可用 API 以及标准的浏览器 DOM API。
虽然 TypeScript 会根据你的 [`target`](/tsconfig#target) 设置提供合理的默认值，但你也可以通过在 `tsconfig.json` 中配置 [`lib`](https://www.typescriptlang.org/tsconfig#lib) 选项，自由挑选项目所需的声明文件。

然而，将这些声明文件与 TypeScript 捆绑在一起偶尔会带来两个缺点：

- 当你升级 TypeScript 时，你也必须同时应对 TypeScript 内置声明文件的改动；而在 DOM API 变动如此频繁的情况下，这可能是一项挑战。
- 很难定制这些文件以同时满足你自身及项目依赖的需求（例如，如果你的依赖声明其使用了 DOM API，你可能也会被迫使用 DOM API）。

TypeScript 4.5 引入了一种覆盖特定内置 `lib` 的方式，其工作机制类似于 `@types/` 的支持方式。
在决定应包含哪些 `lib` 文件时，TypeScript 会首先在 `node_modules` 中查找 `@typescript/lib-*` 作用域包。
例如，当在 `lib` 中包含 `dom` 选项时，如果 `node_modules/@typescript/lib-dom` 存在，TypeScript 将优先使用其中的类型。

随后，你可以使用包管理器安装特定包来接管指定的 `lib`。
例如，目前 TypeScript 在 `@types/web` 上发布了 DOM API 的各个版本。
如果你希望将项目锁定在特定版本的 DOM API 上，可以在 `package.json` 中添加如下配置：

```json
{
  "dependencies": {
    "@typescript/lib-dom": "npm:@types/web"
  }
}
```

这样从 4.5 版本开始，你升级 TypeScript 时，依赖管理器的 lockfile 将确保其始终使用完全相同的 DOM 类型版本。
这意味着你可以完全按照自己的节奏来更新类型定义。

我们由衷感谢 [saschanaz](https://github.com/saschanaz)，在我们构建和试验该特性的过程中，他给予了极大的帮助和耐心。

更多详细信息，你可以查看 [该改动的实现 PR](https://github.com/microsoft/TypeScript/pull/45771)。

## `Awaited` 类型与 Promise 改进

TypeScript 4.5 引入了一个名为 `Awaited` 的新工具类型。
该类型旨在对 `async` 函数中的 `await` 操作或 `Promise` 上的 `.then()` 方法进行建模——具体而言，即它们递归解包 `Promise` 的方式。

```ts
// A = string
type A = Awaited<Promise<string>>

// B = number
type B = Awaited<Promise<Promise<number>>>

// C = boolean | number
type C = Awaited<boolean | Promise<number>>
```

`Awaited` 类型对于对现有 API 进行类型建模非常有帮助，包括 JavaScript 内置的 `Promise.all`、`Promise.race` 等。
事实上，围绕 `Promise.all` 类型推断的一些既有缺陷正是推动设计 `Awaited` 的初衷之一。
以下是在 TypeScript 4.4 及更早版本中会报错的示例：

```ts
declare function MaybePromise<T>(value: T): T | Promise<T> | PromiseLike<T>

async function doSomething(): Promise<[number, number]> {
  const result = await Promise.all([MaybePromise(100), MaybePromise(200)])

  // 错误！
  //
  //    [number | Promise<100>, number | Promise<200>]
  //
  // 不能分配给类型
  //
  //    [number, number]
  return result
}
```

现在，`Promise.all` 结合某些特性与 `Awaited` 带来了更出色的推断结果，上述示例已能顺利通过类型检查。

更多详细信息，你可以 [在 GitHub 上阅读有关该改动的讨论](https://github.com/microsoft/TypeScript/pull/45350)。

## 模板字符串类型作为判别属性

TypeScript 4.5 现在能够窄化具有模板字符串类型的值，并能够将模板字符串类型识别为判别属性（discriminants）。

例如，以下代码此前无法通过类型检查，但在 TypeScript 4.5 中可以成功通过：

```ts twoslash
export interface Success {
  type: `${string}Success`
  body: string
}

export interface Error {
  type: `${string}Error`
  message: string
}

export function handler(r: Success | Error) {
  if (r.type === 'HttpSuccess') {
    const token = r.body
    //            ^?
  }
}
```

更多详细信息，请参阅 [支持该特性的改动](https://github.com/microsoft/TypeScript/pull/46137)。

## `module es2022`

感谢 [Kagami S. Rosylight](https://github.com/saschanaz)，TypeScript 现在支持了一个新的 `module` 设置：`es2022`。
[`module es2022`](/tsconfig#module) 的主要特性是顶层 `await`（top-level `await`），这意味着你可以在 `async` 函数之外使用 `await`。
此前 `--module esnext`（以及现在的 [`--module nodenext`](/tsconfig#target)）中已经支持了该特性，但 `es2022` 是该特性的首个稳定发布目标。

你可以 [在此处阅读有关该改动的更多信息](https://github.com/microsoft/TypeScript/pull/44656)。

## 条件类型的尾递归消除

当检测到可能存在的无限递归，或任何可能耗时过长并影响编辑器体验的类型展开时，TypeScript 通常需要优雅地报错退出。
因此，TypeScript 采用了一套启发式机制，以确保在分析无限深度的类型或处理生成大量中间结果的类型时不会失控。

```ts
type InfiniteBox<T> = { item: InfiniteBox<T> }

type Unpack<T> = T extends { item: infer U } ? Unpack<U> : T

// 错误：类型实例化过深，且可能无限。
type Test = Unpack<InfiniteBox<number>>
```

上述示例特意编写得简单且没有实际意义，但在实际开发中有许多非常有用的类型，却不幸触发了我们的启发式报错限制。
例如，下面的 `TrimLeft` 类型用于移除类字符串类型开头的空格。
如果传入的字符串类型开头带有空格，它会立即将剩余字符串递归传入 `TrimLeft`：

```ts
type TrimLeft<T extends string> = T extends ` ${infer Rest}`
  ? TrimLeft<Rest>
  : T

// Test = "hello" | "world"
type Test = TrimLeft<'   hello' | ' world'>
```

这种类型非常有用，但如果字符串开头有 50 个空格，就会报错：

```ts
type TrimLeft<T extends string> = T extends ` ${infer Rest}`
  ? TrimLeft<Rest>
  : T

// 错误：类型实例化过深，且可能无限。
type Test = TrimLeft<'                                                oops'>
```

这令人遗憾，因为这类类型在对字符串操作建模时往往极其有用——例如编写 URL 路由的解析器。
更糟糕的是，越实用的类型通常会产生越多的类型实例化，进而对其输入长度施加更严苛的限制。

但好在有一个转机：`TrimLeft` 在其中一个分支上的编写方式是*尾递归*（tail-recursive）的。
当它再次调用自身时，会直接返回结果而不再对其做任何二次处理。
因为这些类型不需要创建任何中间结果，所以可以更快速地实现，并且能够避免触发 TypeScript 内置的许多类型递归启发式检查。

因此，TypeScript 4.5 对条件类型进行了尾递归消除（tail-recursion elimination）优化。
只要条件类型的一个分支仅仅是另一个条件类型，TypeScript 就可以避免中间的实例化过程。
虽然仍有启发式机制确保这些类型不会无限膨胀，但其限制阈值已经宽松得多。

需要注意的是，以下类型*不会*被优化，因为它使用了条件类型的结果并将其加入到了联合类型中：

```ts
type GetChars<S> = S extends `${infer Char}${infer Rest}`
  ? Char | GetChars<Rest>
  : never
```

如果你希望将其改写为尾递归形式，可以引入一个接收“累加器”（accumulator）类型参数的辅助类型，就像编写尾递归函数一样：

```ts
type GetChars<S> = GetCharsHelper<S, never>
type GetCharsHelper<S, Acc> = S extends `${infer Char}${infer Rest}`
  ? GetCharsHelper<Rest, Char | Acc>
  : Acc
```

你可以在 [此处](https://github.com/microsoft/TypeScript/pull/45711) 详细阅读有关该实现的更多内容。

## 禁用导入省略（`preserveValueImports`）

在某些情况下，TypeScript 无法检测到你正在使用某个导入。
例如以下代码：

```ts
import { Animal } from './animal.js'

eval('console.log(new Animal().isDangerous())')
```

默认情况下，TypeScript 总是会移除该导入，因为表面上看它并未被使用。
在 TypeScript 4.5 中，你可以启用一个名为 [`preserveValueImports`](/tsconfig#preserveValueImports) 的新标志，以防止 TypeScript 从生成的 JavaScript 代码中剥除任何导入的值。
使用 `eval` 的正当理由屈指可数，但类似的情况在 Svelte 中非常普遍：

```html
<!-- A .svelte File -->
<script>
  import { someFunc } from './some-module.js'
</script>

<button on:click="{someFunc}">Click me!</button>
```

在 Vue.js 的 `<script setup>` 特性中也存在类似情况：

```html
<!-- A .vue File -->
<script setup>
  import { someFunc } from './some-module.js'
</script>

<button @click="someFunc">Click me!</button>
```

这些框架会在 `<script>` 标签外部根据标记生成代码，但 TypeScript *仅*能看到 `<script>` 标签内部的代码。
这意味着 TypeScript 会自动丢弃对 `someFunc` 的导入，导致上述代码无法正常运行！
借助 TypeScript 4.5 中的 [`preserveValueImports`](/tsconfig#preserveValueImports)，你可以避免这些情况。

请注意，当此标志与 [`--isolatedModules`](/tsconfig#isolatedModules) 结合使用时有一项特殊要求：导入的类型*必须*标记为仅类型导入（type-only import），因为一次只处理单个文件的编译器无法判断导入的内容是表面看似未使用的值，还是为了避免运行时崩溃而必须移除的类型。

```ts
// 其中哪一个是应该被保留的值？tsc 知道，但 `ts.transpileModule`、
// ts-loader、esbuild 等不知道，因此 `isolatedModules` 会报错。
import { someFunc, BaseType } from './some-module.js'
//                 ^^^^^^^^
// 错误：当同时启用 'preserveValueImports' 和 'isolatedModules' 时，
// 'BaseType' 是一个类型，必须使用仅类型导入进行导入。
```

这使得 TypeScript 4.5 的另一项特性——[导入名称上的 `type` 修饰符](#type-on-import-names) 显得尤为重要。

更多信息请参阅 [此处的 Pull Request](https://github.com/microsoft/TypeScript/pull/44619)。

## 导入名称上的 `type` 修饰符

如前文所述，[`preserveValueImports`](/tsconfig#preserveValueImports) 和 [`isolatedModules`](/tsconfig#isolatedModules) 提出了特殊要求，以确保构建工具在判断是否可以安全丢弃类型导入时不会产生二义性。

```ts
// 其中哪一个是应该被保留的值？tsc 知道，但 `ts.transpileModule`、
// ts-loader、esbuild 等不知道，因此 `isolatedModules` 会报错。
import { someFunc, BaseType } from './some-module.js'
//                 ^^^^^^^^
// 错误：当同时启用 'preserveValueImports' 和 'isolatedModules' 时，
// 'BaseType' 是一个类型，必须使用仅类型导入进行导入。
```

当这些选项组合使用时，我们需要一种方式来表明某个导入可以被合法丢弃。
TypeScript 已经通过 `import type` 提供了相应语法：

```ts
import type { BaseType } from './some-module.js'
import { someFunc } from './some-module.js'

export class Thing implements BaseType {
  // ...
}
```

这确实可行，但如果能避免对同一模块编写两条 import 语句就更好了。
这也正是 TypeScript 4.5 允许在单个具名导入上使用 `type` 修饰符的原因之一，从而让你能够根据需要按需混用：

```ts
import { someFunc, type BaseType } from './some-module.js'

export class Thing implements BaseType {
  someMethod() {
    someFunc()
  }
}
```

在上述示例中，在 [`preserveValueImports`](/tsconfig#preserveValueImports) 下，`BaseType` 始终保证被擦除，而 `someFunc` 则会被保留，最终输出如下代码：

```js
import { someFunc } from './some-module.js'

export class Thing {
  someMethod() {
    someFunc()
  }
}
```

更多信息请参阅 [GitHub 上的改动](https://github.com/microsoft/TypeScript/pull/45998)。

## 私有字段存在性检查

TypeScript 4.5 支持了一项用于检查对象是否包含私有字段的 ECMAScript 提案。
你现在可以在类中定义一个 `#private` 字段成员，并通过 `in` 运算符来检查另一个对象是否包含该字段：

```ts
class Person {
  #name: string
  constructor(name: string) {
    this.#name = name
  }

  equals(other: unknown) {
    return (
      other &&
      typeof other === 'object' &&
      #name in other && // <- 这是新增特性！
      this.#name === other.#name
    )
  }
}
```

该特性的一个有趣之处在于，`#name in other` 检查意味着 `other` 必定是通过 `Person` 构造实例化的，因为除此之外没有任何其他方式能让该私有字段存在。
这实际上是该提案的核心特性之一，也是该提案被称为“人体工程学品牌检查”（ergonomic brand checks）的原因——因为私有字段往往充当一种“品牌”（brand），用于防范非该类实例的对象。
因此，TypeScript 能够在每次检查时适当地窄化 `other` 的类型，直至将其窄化为 `Person` 类型。

我们非常感谢彭博社（Bloomberg）的朋友们 [贡献了此 Pull Request](https://github.com/microsoft/TypeScript/pull/44648)：[Ashley Claymore](https://github.com/acutmore)、[Titian Cernicova-Dragomir](https://github.com/dragomirtitian)、[Kubilay Kahveci](https://github.com/mkubilayk) 以及 [Rob Palmer](https://github.com/robpalme)！

## 导入断言（Import Assertions）

TypeScript 4.5 支持了一项关于*导入断言*（import assertions）的 ECMAScript 提案。
运行时可以使用这种语法来确保导入的内容具有预期的格式。

```ts
import obj from './something.json' assert { type: 'json' }
```

TypeScript 不会检查这些断言的内容，因为它们是宿主环境特有的；TypeScript 会原样保留它们，交由浏览器和运行时去处理（并可能由其抛出错误）：

```ts
// TypeScript 允许这样做。
// 但你的浏览器？很可能不支持。
import obj from './something.json' assert { type: 'fluffy bunny' }
```

动态 `import()` 调用也可以通过第二个参数使用导入断言：

```ts
const obj = await import('./something.json', {
  assert: { type: 'json' },
})
```

该第二个参数的预期类型由名为 `ImportCallOptions` 的新类型定义，目前仅接受 `assert` 属性。

感谢 [王文璐 (Wenlu Wang)](https://github.com/Kingwl/) [实现了该特性](https://github.com/microsoft/TypeScript/pull/40698)！

## JSDoc 中的常量断言和默认类型参数

TypeScript 4.5 为我们的 JSDoc 支持带来了更丰富的表达能力。

其中一个例子是 `const` 常量断言。在 TypeScript 中，你可以在字面量后编写 `as const` 来获得更精确且不可变的类型：

```ts
// 类型为 { prop: string }
let a = { prop: 'hello' }

// 类型为 { readonly prop: "hello" }
let b = { prop: 'hello' } as const
```

在 JavaScript 文件中，你现在可以使用 JSDoc 类型断言来实现同样的效果：

```ts
// 类型为 { prop: string }
let a = { prop: 'hello' }

// 类型为 { readonly prop: "hello" }
let b = /** @type {const} */ { prop: 'hello' }
```

温馨提示，JSDoc 类型断言注释以 `/** @type {TheTypeWeWant} */` 开头，后跟用括号包裹的表达式：

```js
/** @type {TheTypeWeWant} */` (someExpression)
```

TypeScript 4.5 还为 JSDoc 添加了默认类型参数支持，这意味着 TypeScript 中的如下 `type` 声明：

```ts
type Foo<T extends string | number = number> = { prop: T }
```

在 JavaScript 中可以改写为如下 `@typedef` 声明：

```js
/**
 * @template {string | number} [T=number]
 * @typedef Foo
 * @property prop {T}
 */

// 或

/**
 * @template {string | number} [T=number]
 * @typedef {{ prop: T }} Foo
 */
```

更多信息请参阅 [常量断言的 Pull Request](https://github.com/microsoft/TypeScript/pull/45464) 以及 [类型参数默认值的改动](https://github.com/microsoft/TypeScript/pull/45483)。

## 使用 `realPathSync.native` 获得更快的加载时间

TypeScript 现在在所有操作系统上均利用 Node.js `realPathSync` 函数的系统原生实现。

此前该函数仅在 Linux 上使用，但在 TypeScript 4.5 中，它已被应用到 Windows 和 MacOS 等通常不区分大小写的操作系统中。
在某些代码库中，此项改动使项目加载速度提升了 5-13%（取决于宿主操作系统）。

更多信息请参阅 [此处的原始改动](https://github.com/microsoft/TypeScript/pull/44966)，以及 [4.5 特定的改动](https://github.com/microsoft/TypeScript/pull/44966)。

## JSX 属性的代码片段补全

TypeScript 4.5 带来了针对 JSX 属性的*代码片段补全*（snippet completions）。
在 JSX 标签中编写属性时，TypeScript 已经能为这些属性提供补全建议；
而通过代码片段补全，它可以通过自动添加初始值并将光标置于合适的位置，从而减少额外的键盘输入。

![JSX 属性的代码片段补全。对于字符串属性，会自动添加引号；对于数字属性，会自动添加大括号](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2021/10/jsx-attributes-snippets-4-5.gif)

TypeScript 通常会根据属性的类型来决定插入哪种初始值格式，但你可以在 Visual Studio Code 中自定义此行为。

![VS Code 中关于 JSX 属性补全的设置](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2021/10/jsx-snippet-settings-4-5.png)

请注意，该特性仅在较新版本的 Visual Studio Code 中有效，因此你可能需要使用 Insiders 预览版才能体验。
更多信息请 [参阅原始 Pull Request](https://github.com/microsoft/TypeScript/pull/45903)。

## 编辑器对未解析类型提供更好的支持

在某些情况下，编辑器会使用轻量级的“部分”（partial）语义模式——无论是在编辑器等待完整项目加载期间，还是在 [GitHub 基于 Web 的编辑器](https://docs.github.com/en/codespaces/developing-in-codespaces/web-based-editor) 等上下文中。

在较旧版本的 TypeScript 中，如果语言服务找不到某个类型，它会直接显示 `any`：

![悬停在找不到 `Buffer` 的签名上时，TypeScript 将其替换为 `any`](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2021/10/quick-info-unresolved-4-4.png)

在上面的示例中，由于未找到 `Buffer`，TypeScript 在*快速信息*（quick info）中将其替换为了 `any`。
在 TypeScript 4.5 中，TypeScript 将尽最大努力保留你书写的原始名称：

![悬停在找不到 `Buffer` 的签名上时，它继续使用名称 `Buffer`](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2021/10/quick-info-unresolved-4-5.png)

但是，如果你将鼠标悬停在 `Buffer` 本身之上，你会得到一个提示，说明 TypeScript 无法找到 `Buffer`：

![TypeScript 显示 `type Buffer = /* unresolved */ any;`](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2021/10/quick-info-unresolved-on-type-4-5.png)

综合起来，这在 TypeScript 无法获取完整程序信息时提供了更加流畅的体验。
请记住，在常规场景下，当找不到某个类型时，你始终会收到错误提示。

更多详细信息，请 [参阅此处的实现](https://github.com/microsoft/TypeScript/pull/45976)。

## 破坏性变更

### `lib.d.ts` 变更

TypeScript 4.5 包含了对其内置声明文件的改动，这可能会影响你的编译；
然而，[这些改动相当轻微](https://github.com/microsoft/TypeScript-DOM-lib-generator/issues/1143)，我们预计大多数代码不会受到影响。

### `Awaited` 引起的推断变更

由于 `Awaited` 现在已在 `lib.d.ts` 中使用并作为 `await` 的运算结果，你可能会看到某些泛型类型发生了变化，从而可能导致不兼容；
然而，鉴于围绕 `Awaited` 做出了许多旨在避免破坏性变更的精心设计，我们预计大多数代码不会受到影响。

### `tsconfig.json` 根级别的编译选项检查

不小心遗漏 `tsconfig.json` 中的 `compilerOptions` 配置层级是一个很容易犯的错误。
为了帮助捕获该错误，在 TypeScript 4.5 中，如果 `tsconfig.json` 中*未*定义 `compilerOptions`，而直接在顶层添加了与 `compilerOptions` 中任何可用选项相匹配的字段，将会报错。
