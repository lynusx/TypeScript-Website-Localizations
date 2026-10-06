---
title: TypeScript 3.9
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-3-9.html
oneline: TypeScript 3.9 发布说明
---

## 推断与 `Promise.all` 的改进

TypeScript 最近的几个版本（大约在 3.7 左右）更新了诸如 `Promise.all` 和 `Promise.race` 等函数的声明。然而，这引入了一些退化问题，尤其是在混用带有 `null` 或 `undefined` 的值时。

```ts
interface Lion {
  roar(): void
}

interface Seal {
  singKissFromARose(): void
}

async function visitZoo(
  lionExhibit: Promise<Lion>,
  sealExhibit: Promise<Seal | undefined>,
) {
  let [lion, seal] = await Promise.all([lionExhibit, sealExhibit])
  lion.roar() // uh oh
  //  ~~~~
  // Object is possibly 'undefined'.
}
```

这是一种反常的行为！`sealExhibit` 包含 `undefined` 的事实，不知为何“污染”了 `lion` 的类型，使其也包含了 `undefined`。

感谢 [Jack Bates](https://github.com/jablko) 提交的 [Pull Request](https://github.com/microsoft/TypeScript/pull/34501)，TypeScript 3.9 通过改进推断过程修复了这个问题。上述代码现在不再报错。如果你之前因为 `Promise` 相关的问题而停留在旧版本，我们建议你尝试一下 3.9！

### 那么 `awaited` 类型呢？

如果你一直在关注我们的 issue 跟踪器和设计会议记录，可能会了解到我们围绕[名为 `awaited` 的新类型运算符](https://github.com/microsoft/TypeScript/pull/35998)所做的工作。该类型运算符的目标是准确模拟 JavaScript 中 `Promise` 解包的运作方式。

我们最初计划在 TypeScript 3.9 中发布 `awaited`，但在现有代码库中运行早期构建版本后，我们意识到该特性在能够顺利推向所有人之前还需要更多的设计工作。因此，我们决定将其从主分支中撤出，直到我们更有信心为止。我们将对该特性进行更多实验，但不会在本次发布中提供。

## 速度与性能提升

TypeScript 3.9 带来了诸多新的速度改进。在观察到诸如 material-ui 和 styled-components 等包在编辑和编译速度上表现极差之后，我们的团队一直专注于性能优化。我们深入剖析了这些问题，通过一系列不同的 Pull Request 优化了涉及大型联合类型、交叉类型、条件类型和映射类型的某些病态场景：

- https://github.com/microsoft/TypeScript/pull/36576
- https://github.com/microsoft/TypeScript/pull/36590
- https://github.com/microsoft/TypeScript/pull/36607
- https://github.com/microsoft/TypeScript/pull/36622
- https://github.com/microsoft/TypeScript/pull/36754
- https://github.com/microsoft/TypeScript/pull/36696

这些 Pull Request 中的每一个都能在特定代码库上将编译时间减少约 5-10%。总计下来，我们相信 material-ui 的编译时间减少了约 40%！

我们还针对编辑器场景中的文件重命名功能进行了一些调整。我们从 Visual Studio Code 团队获悉，在重命名文件时，仅仅确定需要更新哪些导入语句就可能花费 5 到 10 秒。TypeScript 3.9 通过[改变编译器和语言服务缓存文件查找的内部机制](https://github.com/microsoft/TypeScript/pull/37055)解决了这个问题。

虽然仍有改进空间，但我们希望这些成果能为每个人带来更加流畅迅捷的体验！

## `// @ts-expect-error` 注释指令

假设我们正在用 TypeScript 编写一个库，并导出一个名为 `doStuff` 的函数作为公共 API 的一部分。该函数的类型声明指定它接收两个 `string` 参数，以便其他 TypeScript 用户可以获得类型检查报错；但它同时还执行运行时检查（可能仅在开发构建中），以便向 JavaScript 用户提供有用的错误提示。

```ts
function doStuff(abc: string, xyz: string) {
  assert(typeof abc === 'string')
  assert(typeof xyz === 'string')

  // do some stuff
}
```

因此，当 TypeScript 用户误用此函数时会看到红色的波浪线和错误消息，而 JavaScript 用户则会得到断言错误。我们希望对此行为进行测试，于是编写了一个单元测试：

```ts
expect(() => {
  doStuff(123, 456)
}).toThrow()
```

遗憾的是，如果我们的测试也是用 TypeScript 编写的，TypeScript 就会向我们报错！

```ts
doStuff(123, 456)
//          ~~~
// error: Type 'number' is not assignable to type 'string'.
```

这就是为什么 TypeScript 3.9 带来了一项新特性：`// @ts-expect-error` 注释。
当某一行前面带有 `// @ts-expect-error` 注释时，TypeScript 将抑制该错误的报告；但如果该行并没有错误，TypeScript 则会提示 `// @ts-expect-error` 是多余的。

举个简单的例子，以下代码是正常的：

```ts
// @ts-expect-error
console.log(47 * 'octopus')
```

而以下代码：

```ts
// @ts-expect-error
console.log(1 + 1)
```

则会导致报错：

```
Unused '@ts-expect-error' directive.
```

我们非常感谢实现该特性的贡献者 [Josh Goldberg](https://github.com/JoshuaKGoldberg)。有关更多信息，你可以查看 [关于 `ts-expect-error` 的 Pull Request](https://github.com/microsoft/TypeScript/pull/36014)。

### 应该选择 `ts-ignore` 还是 `ts-expect-error`？

在某些方面，`// @ts-expect-error` 可以充当错误抑制注释，类似于 `// @ts-ignore`。不同之处在于，如果下一行没有错误，`// @ts-ignore` 什么都不会做。

你可能会想把现有的 `// @ts-ignore` 注释替换为 `// @ts-expect-error`，并且想知道在未来的代码中哪一个更合适。虽然这完全取决于你和你的团队，但在某些特定场景下，我们有一些选择建议。

在以下情况下选择 `ts-expect-error`：

- 你正在编写测试代码，确实希望类型系统对某项操作报错
- 你预期修复很快就会到来，只需要一个临时的通融方案
- 你的项目规模适中，团队积极主动，希望在受影响代码恢复有效后立即移除抑制注释

在以下情况下选择 `ts-ignore`：

- 你的项目规模较大，且在归属不明确的代码中出现了新错误
- 你正处于两个不同 TypeScript 版本之间的升级过程中，某行代码在一个版本中报错但在另一个版本中不报错
- 你确实没有时间去权衡这两种选项哪一个更优

## 条件表达式中未调用的函数检查

在 TypeScript 3.7 中，我们引入了未调用函数检查，以便在你忘记调用函数时报告错误。

```ts
function hasImportantPermissions(): boolean {
  // ...
}

// Oops!
if (hasImportantPermissions) {
  //  ~~~~~~~~~~~~~~~~~~~~~~~
  // This condition will always return true since the function is always defined.
  // Did you mean to call it instead?
  deleteAllTheImportantFiles()
}
```

然而，该错误此前仅适用于 `if` 语句中的条件。感谢 [Alexander Tarasyuk](https://github.com/a-tarasyuk) 提交的 [Pull Request](https://github.com/microsoft/TypeScript/pull/36402)，该特性现在在三元条件表达式（即 `cond ? trueExpr : falseExpr` 语法）中也得到了支持。

```ts
declare function listFilesOfDirectory(dirPath: string): string[]
declare function isDirectory(): boolean

function getAllFiles(startFileName: string) {
  const result: string[] = []
  traverse(startFileName)
  return result

  function traverse(currentPath: string) {
    return isDirectory
      ? //     ~~~~~~~~~~~
        // This condition will always return true
        // since the function is always defined.
        // Did you mean to call it instead?
        listFilesOfDirectory(currentPath).forEach(traverse)
      : result.push(currentPath)
  }
}
```

https://github.com/microsoft/TypeScript/issues/36048

## 编辑器改进

TypeScript 编译器不仅驱动着大多数主流编辑器中的 TypeScript 编辑体验，还驱动着 Visual Studio 系列编辑器等工具中的 JavaScript 体验。在编辑器中使用新的 TypeScript/JavaScript 功能因具体编辑器而异，但：

- Visual Studio Code 支持[选择不同版本的 TypeScript](https://code.visualstudio.com/docs/typescript/typescript-compiling#_using-the-workspace-version-of-typescript)。此外，还可以使用 [JavaScript/TypeScript Nightly 扩展](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-typescript-next)来体验前沿版本（通常非常稳定）。
- Visual Studio 2017/2019 提供了上述 SDK 安装程序以及 [MSBuild 安装包](https://www.nuget.org/packages/Microsoft.TypeScript.MSBuild)。
- Sublime Text 3 支持[选择不同版本的 TypeScript](https://github.com/microsoft/TypeScript-Sublime-Plugin#note-using-different-versions-of-typescript)。

### JavaScript 中的 CommonJS 模块自动导入

一个极佳的新改进是对使用 CommonJS 模块的 JavaScript 文件进行自动导入。

在旧版本中，TypeScript 总是假定无论你的文件如何，你都希望使用 ECMAScript 风格的导入，例如：

```js
import * as fs from 'fs'
```

然而，并非每个人在编写 JavaScript 文件时都面向 ECMAScript 风格的模块。许多用户仍在使用 CommonJS 风格的 `require(...)` 导入，如下所示：

```js
const fs = require('fs')
```

TypeScript 现在会自动检测你所使用的导入风格，以保持文件风格整洁且一致。

<video src="https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2020/03/ERkaliGU0AA5anJ1.mp4"></video>

有关该变更的更多详情，请参见[对应的 Pull Request](https://github.com/microsoft/TypeScript/pull/37027)。

### 代码操作保留换行符

TypeScript 的重构和快速修复此前在保留换行符方面表现得不够理想。以一个非常基础的代码为例：

```ts
const maxValue = 100

/*start*/
for (let i = 0; i <= maxValue; i++) {
  // First get the squared value.
  let square = i ** 2

  // Now print the squared value.
  console.log(square)
}
/*end*/
```

如果我们在编辑器中高亮从 `/*start*/` 到 `/*end*/` 的范围以提取为新函数，最终会得到如下代码：

```ts
const maxValue = 100

printSquares()

function printSquares() {
  for (let i = 0; i <= maxValue; i++) {
    // First get the squared value.
    let square = i ** 2
    // Now print the squared value.
    console.log(square)
  }
}
```

![在旧版本 TypeScript 中将 for 循环提取为函数。换行符未被保留。](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2020/03/printSquaresWithoutNewlines-3.9.gif.gif)

这并不理想——我们在 `for` 循环中的每条语句之间都留了空行，但重构却移除了它！
TypeScript 3.9 做了额外的工作来保留我们的书写格式。

```ts
const maxValue = 100

printSquares()

function printSquares() {
  for (let i = 0; i <= maxValue; i++) {
    // First get the squared value.
    let square = i ** 2

    // Now print the squared value.
    console.log(square)
  }
}
```

![在 TypeScript 3.9 中将 for 循环提取为函数。换行符得以保留。](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2020/03/printSquaresWithNewlines-3.9.gif.gif)

你可以在[此 Pull Request](https://github.com/microsoft/TypeScript/pull/36688) 中了解有关实现的更多信息。

### 缺失返回值表达式的快速修复

有时我们可能会忘记返回函数中最后一条语句的值，尤其是在给箭头函数添加花括号时：

```ts
// before
let f1 = () => 42

// oops - not the same!
let f2 = () => {
  42
}
```

感谢社区成员 [王文璐（Wenlu Wang）](https://github.com/Kingwl) 提交的 [Pull Request](https://github.com/microsoft/TypeScript/pull/26434)，TypeScript 现在可以提供快速修复来添加缺失的 `return` 语句、移除花括号，或者为看似对象字面量的箭头函数体添加圆括号。

![TypeScript 通过添加 `return` 语句或移除花括号来修复未返回表达式的错误。](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2020/04/missingReturnValue-3-9.gif)

### 支持“解决方案风格”的 `tsconfig.json` 文件

编辑器需要确定某个文件属于哪个配置文件，以便应用适当的选项并弄清当前“项目”中包含哪些其他文件。默认情况下，基于 TypeScript 语言服务的编辑器通过向上遍历每个父目录来查找 `tsconfig.json`。

然而，当某个 `tsconfig.json` 的存在仅仅是为了引用其他 `tsconfig.json` 文件时，就会出现问题：

```jsonc tsconfig
// tsconfig.json
{
  "files": [],
  "references": [
    { "path": "./tsconfig.shared.json" },
    { "path": "./tsconfig.frontend.json" },
    { "path": "./tsconfig.backend.json" },
  ],
}
```

这种除了管理其他项目文件外实际上不做任何事情的文件，在某些环境中通常被称为“解决方案”。在此场景下，语言服务不会加载这些 `tsconfig.*.json` 文件中的任何一个，但我们非常希望语言服务能够理解当前 `.ts` 文件可能属于该根 `tsconfig.json` 中提及的项目之一。

TypeScript 3.9 在编辑场景中添加了对此类配置的支持。有关更多详情，请查看[添加此功能的 Pull Request](https://github.com/microsoft/TypeScript/pull/37239)。

## 破坏性变更

### 可选链与非空断言中的解析差异

TypeScript 最近实现了可选链运算符，但我们收到了用户反馈，指出可选链（`?.`）与非空断言运算符（`!`）结合使用时的行为极度反直觉。

具体而言，在之前的版本中，代码：

```ts
foo?.bar!.baz
```

被解释为等同于以下 JavaScript 代码：

```js
;(foo?.bar).baz
```

在上述代码中，圆括号阻止了可选链的“短路”行为，因此如果 `foo` 是 `undefined`，访问 `baz` 将导致运行时错误。

指出这种行为的 Babel 团队以及向我们提供反馈的大多数用户都认为这种行为是不正确的。我们也这么认为！我们听到的最普遍的看法是：`!` 运算符应该直接“消失”，因为其本意只是从 `bar` 的类型中移除 `null` 和 `undefined`。

换句话说，大多数人认为原始代码片段应该被解释为：

```js
foo?.bar.baz
```

当 `foo` 为 `undefined` 时，它只会求值为 `undefined`。

这是一项破坏性变更，但我们相信大多数代码在编写时都是基于新的解释预期的。希望恢复旧行为的用户可以在 `!` 运算符左侧显式添加圆括号：

```ts
foo?.bar!.baz
```

### `}` 和 `>` 现在是无效的 JSX 文本字符

JSX 规范禁止在文本位置使用 `}` 和 `>` 字符。为了更好地符合规范，TypeScript 和 Babel 均已决定强制执行此规则。插入这些字符的新方式是使用 HTML 转义实体（例如 `<span> 2 &gt 1 </span>`）或插入带有字符串字面量的表达式（例如 `<span> 2 {">"} 1 </span>`）。

幸运的是，感谢 [Brad Zacher](https://github.com/bradzacher) 提交的用于强制执行此规则的 [Pull Request](https://github.com/microsoft/TypeScript/pull/36636)，你将收到类似如下的错误消息：

```
Unexpected token. Did you mean `{'>'}` or `&gt;`?
Unexpected token. Did you mean `{'}'}` or `&rbrace;`?
```

例如：

```tsx
let directions = <span>Navigate to: Menu Bar > Tools > Options</span>;
//                                           ~       ~
// Unexpected token. Did you mean `{'>'}` or `&gt;`?
```

该错误消息附带了一个便捷的快速修复功能；感谢 [Alexander Tarasyuk](https://github.com/a-tarasyuk)，如果存在大量错误，[你可以批量应用这些更改](https://github.com/microsoft/TypeScript/pull/37436)。

### 对交叉类型与可选属性进行更严格的检查

通常情况下，如果 `A` 或 `B` 中有任意一个可以赋值给 `C`，那么像 `A & B` 这样的交叉类型就可以赋值给 `C`；然而，这在涉及可选属性时有时会出现问题。例如以下代码：

```ts
interface A {
  a: number // notice this is 'number'
}

interface B {
  b: string
}

interface C {
  a?: boolean // notice this is 'boolean'
  b: string
}

declare let x: A & B
declare let y: C

y = x
```

在较早版本的 TypeScript 中，这是被允许的，因为虽然 `A` 与 `C` 完全不兼容，但 `B` **确实**与 `C` 兼容。

在 TypeScript 3.9 中，只要交叉类型中的每个类型都是具体对象类型，类型系统就会同时考虑所有的属性。因此，TypeScript 会发现 `A & B` 的 `a` 属性与 `C` 的 `a` 属性不兼容：

```
Type 'A & B' is not assignable to type 'C'.
  Types of property 'a' are incompatible.
    Type 'number' is not assignable to type 'boolean | undefined'.
```

有关此变更的更多信息，请[参见对应的 Pull Request](https://github.com/microsoft/TypeScript/pull/37195)。

### 通过判别式属性缩减交叉类型

在某些情况下，你可能会得到描述根本不存在的值的类型。例如：

```ts
declare function smushObjects<T, U>(x: T, y: U): T & U

interface Circle {
  kind: 'circle'
  radius: number
}

interface Square {
  kind: 'square'
  sideLength: number
}

declare let x: Circle
declare let y: Square

let z = smushObjects(x, y)
console.log(z.kind)
```

这段代码有些奇特，因为实际上根本无法创建 `Circle` 和 `Square` 的交叉——它们具有两个不兼容的 `kind` 字段。在 TypeScript 之前的版本中，这段代码是被允许的，`kind` 本身的类型为 `never`，因为 `"circle" & "square"` 描述了一组永远不可能存在的值。

在 TypeScript 3.9 中，类型系统在此处更加严格——它注意到由于两者的 `kind` 属性，`Circle` 和 `Square` 无法进行交叉。因此，它不再仅仅将 `z.kind` 的类型折叠为 `never`，而是将 `z` 本身的类型（`Circle & Square`）直接折叠为 `never`。这意味着上述代码现在会报错：

```
Property 'kind' does not exist on type 'never'.
```

我们观察到的大多数破坏性报错似乎都与存在细微错误的类型声明有关。更多详情，请[查看原始 Pull Request](https://github.com/microsoft/TypeScript/pull/36696)。

### Getter/Setter 不再可枚举

在旧版本的 TypeScript 中，类中的 `get` 和 `set` 访问器在编译输出时被设为可枚举；然而，这并不符合 ECMAScript 规范关于访问器必须不可枚举的规定。因此，目标为 ES5 和 ES2015 的 TypeScript 代码在行为上可能会有所不同。

感谢 GitHub 用户 [pathurs](https://github.com/pathurs) 提交的 [Pull Request](https://github.com/microsoft/TypeScript/pull/32264)，TypeScript 3.9 现在在这方面更加严格地遵循了 ECMAScript 规范。

### 继承 `any` 的类型参数不再表现为 `any`

在较早版本的 TypeScript 中，约束为 `any` 的类型参数可以被当作 `any` 处理。

```ts
function foo<T extends any>(arg: T) {
  arg.spfjgerijghoied // no error!
}
```

这属于设计上的疏漏，因此 TypeScript 3.9 采取了更稳妥的处理方式，对此类存疑操作报告错误。

```ts
function foo<T extends any>(arg: T) {
  arg.spfjgerijghoied
  //  ~~~~~~~~~~~~~~~
  // Property 'spfjgerijghoied' does not exist on type 'T'.
}
```

### `export *` 始终被保留

在以往的 TypeScript 版本中，如果 `foo` 未导出任何值，诸如 `export * from "foo"` 的声明就会在 JavaScript 输出中被丢弃。这种编译发射方式存在问题，因为它是类型导向的，Babel 无法对其进行模拟。TypeScript 3.9 将始终生成这些 `export *` 声明。在实际场景中，我们预计这不会对太多现有代码造成影响。

### 针对 libdom.d.ts 的更多调整

我们正在继续将更多 TypeScript 内置的 .d.ts 库（lib.d.ts 系列）迁移为直接从 DOM 规范的 Web IDL 文件生成。因此，一些与媒体访问相关的厂商特有类型已被移除。

在项目的环境声明 `*.d.ts` 文件中添加以下内容即可恢复它们：

<!-- prettier-ignore -->
```ts
interface AudioTrackList {
     [Symbol.iterator](): IterableIterator<AudioTrack>;
 }

interface HTMLVideoElement {
  readonly audioTracks: AudioTrackList

  msFrameStep(forward: boolean): void;
  msInsertVideoEffect(activatableClassId: string, effectRequired: boolean, config?: any): void;
  msSetVideoRectangle(left: number, top: number, right: number, bottom: number): void;
  webkitEnterFullScreen(): void;
  webkitEnterFullscreen(): void;
  webkitExitFullScreen(): void;
  webkitExitFullscreen(): void;

  msHorizontalMirror: boolean;
  readonly msIsLayoutOptimalForPlayback: boolean;
  readonly msIsStereo3D: boolean;
  msStereo3DPackingMode: string;
  msStereo3DRenderMode: string;
  msZoom: boolean;
  onMSVideoFormatChanged: ((this: HTMLVideoElement, ev: Event) => any) | null;
  onMSVideoFrameStepCompleted: ((this: HTMLVideoElement, ev: Event) => any) | null;
  onMSVideoOptimalLayoutChanged: ((this: HTMLVideoElement, ev: Event) => any) | null;
  webkitDisplayingFullscreen: boolean;
  webkitSupportsFullscreen: boolean;
}

interface MediaError {
  readonly msExtendedCode: number;
  readonly MS_MEDIA_ERR_ENCRYPTED: number;
}
```
