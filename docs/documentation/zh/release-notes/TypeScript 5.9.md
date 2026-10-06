---
title: TypeScript 5.9
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-5-9.html
oneline: TypeScript 5.9 发布说明
---

## 极简化并更新的 `tsc --init`

长期以来，TypeScript 编译器一直支持通过 `--init` 标志在当前目录下创建一个 `tsconfig.json`。
在过去的几年中，运行 `tsc --init` 会生成一个内容非常“充实”的 `tsconfig.json`，其中塞满了被注释掉的配置项及其说明。
我们最初这样设计的目的是为了让各种配置项更易于被发现和快速开启。

然而，根据外部反馈（以及我们自己的实际体验），我们发现大家通常会在生成后立即删掉这些新 `tsconfig.json` 文件中的大部分内容。
当用户想要了解新的配置项时，往往更依赖编辑器的自动补全，或者直接查阅[官网上的 tsconfig 参考文档](https://www.typescriptlang.org/tsconfig/)（生成的 `tsconfig.json` 中也包含了该链接！）。
每个配置项的作用在该页面上都有详尽的记录，并且也可以通过编辑器的悬停提示（hover / tooltips / quick info）随时查看。
虽然列出部分被注释的配置项可能有所帮助，但生成的 `tsconfig.json` 往往显得过于冗长繁杂。

同时，我们也觉得是时候让 `tsc --init` 默认启用一些更具规范指导性的设置了。
我们审视了用户在创建新 TypeScript 项目时经常遇到的一些痛点和不便。
例如，大多数用户编写的都是模块（而非全局脚本），而 `--moduleDetection` 可以强制 TypeScript 将每个实现文件都视为模块。
开发者通常也希望直接在运行时中使用最新的 ECMAScript 特性，因此 `--target` 通常可以设置为 `esnext`。
JSX 用户经常觉得回过头去配置 `--jsx` 是一种不必要的繁琐操作，而且其可选值也有点令人困惑。
此外，项目往往会从 `node_modules/@types` 加载远超 TypeScript 实际所需的声明文件，而指定一个空的 `types` 数组有助于限制这一点。

在 TypeScript 5.9 中，不带任何其他参数直接运行 `tsc --init` 将生成如下 `tsconfig.json`：

```json5
{
  // 访问 https://aka.ms/tsconfig 阅读关于该文件的更多信息
  compilerOptions: {
    // 文件布局
    // "rootDir": "./src",
    // "outDir": "./dist",

    // 环境设置
    // 另请参阅 https://aka.ms/tsconfig_modules
    module: 'nodenext',
    target: 'esnext',
    types: [],
    // 针对 nodejs：
    // "lib": ["esnext"],
    // "types": ["node"],
    // 以及 npm install -D @types/node

    // 其他输出
    sourceMap: true,
    declaration: true,
    declarationMap: true,

    // 更严格的类型检查选项
    noUncheckedIndexedAccess: true,
    exactOptionalPropertyTypes: true,

    // 代码风格选项
    // "noImplicitReturns": true,
    // "noImplicitOverride": true,
    // "noUnusedLocals": true,
    // "noUnusedParameters": true,
    // "noFallthroughCasesInSwitch": true,
    // "noPropertyAccessFromIndexSignature": true,

    // 推荐选项
    strict: true,
    jsx: 'react-jsx',
    verbatimModuleSyntax: true,
    isolatedModules: true,
    noUncheckedSideEffectImports: true,
    moduleDetection: 'force',
    skipLibCheck: true,
  },
}
```

更多详情请参阅[实现该功能的 PR](https://github.com/microsoft/TypeScript/pull/61813) 和[讨论议题](https://github.com/microsoft/TypeScript/issues/58420)。

## 支持 `import defer`

TypeScript 5.9 引入了对 [ECMAScript 延迟模块求值提案](https://github.com/tc39/proposal-defer-import-eval/)的支持，采用了全新的 `import defer` 语法。
该特性允许你在导入模块时不会立即执行该模块及其依赖项，从而更好地控制计算任务和副作用的触发时机。

该语法仅允许命名空间导入：

```ts
import defer * as feature from './some-feature.js'
```

`import defer` 的核心优势在于，只有在首次访问该模块的某个导出成员时，模块才会被求值执行。
考虑以下示例：

```ts
// ./some-feature.ts
initializationWithSideEffects()

function initializationWithSideEffects() {
  // ...
  specialConstant = 42

  console.log('Side effects have occurred!')
}

export let specialConstant: number
```

当使用 `import defer` 时，直到你真正访问导入的命名空间的属性时，`initializationWithSideEffects()` 函数才会被调用：

```ts
import defer * as feature from './some-feature.js'

// 尚未产生任何副作用

// ...

// 一旦访问 `specialConstant`，`feature` 模块的内容
// 就会执行并产生副作用。
console.log(feature.specialConstant) // 42
```

由于模块的求值被延迟到你访问其成员时才发生，因此不能在 `import defer` 中使用具名导入或默认导入：

```ts
// ❌ 不允许
import defer { doSomething } from "some-module";

// ❌ 不允许
import defer defaultExport from "some-module";

// ✅ 仅支持该语法
import defer * as feature from "some-module";
```

请注意，当你编写 `import defer` 时，该模块及其依赖项已经被完全加载并准备好执行。
这意味着模块必须存在，并且会从文件系统或网络资源中加载。
常规 `import` 与 `import defer` 的关键区别在于，_语句和声明的执行_被延迟到了你访问导入命名空间的属性之时。

这一特性对于按需加载包含昂贵或特定平台初始化逻辑的模块特别有用。它还可以通过将应用特性的模块求值延迟到真正需要时，从而显著提升应用的启动性能。

需要注意的是，TypeScript 完全不会对 `import defer` 进行任何语法转换或“降级”。
它旨在直接用于原生支持该特性的运行时，或配合能够进行相应转换的打包工具使用。
这意味着 `import defer` 仅在 `--module` 模式为 `preserve` 和 `esnext` 时可用。

在此我们要感谢在 TC39 中主导该提案并提交了[该功能实现 PR](https://github.com/microsoft/TypeScript/pull/60757) 的 [Nicolò Ribaudo](https://github.com/nicolo-ribaudo)。

## 支持 `--module node20`

TypeScript 为 `--module` 和 `--moduleResolution` 配置提供了若干 `node*` 选项。
最近，`--module nodenext` 支持了在 CommonJS 模块中通过 `require()` 加载 ECMAScript 模块的能力，并且正确地拒绝了导入断言（转而支持符合规范标准的[导入属性](https://github.com/tc39/proposal-import-attributes)）。

TypeScript 5.9 为这些配置项带来了一个名为 `node20` 的稳定选项，旨在模拟 Node.js v20 的行为。
与 `--module nodenext` 或 `--moduleResolution nodenext` 不同，该选项未来不太可能引入新的行为变动。
同样不同于 `nodenext` 的是，除非另行配置，否则指定 `--module node20` 将默认推断 `--target es2023`。
而 `--module nodenext` 则推断为浮动的 `--target esnext`。

有关更多信息，请[参阅此处具体实现](https://github.com/microsoft/TypeScript/pull/61805)。

## DOM API 中新增概要描述

以往，TypeScript 中的许多 DOM API 仅仅包含指向该 API 对应 MDN 文档的链接。
这些链接固然有用，但并未提供关于该 API 作用的快速概要。
得益于 [Adam Naji](https://github.com/Bashamega) 的贡献，TypeScript 现在根据 MDN 文档为许多 DOM API 补充了概要描述。
你可以[在此处](https://github.com/microsoft/TypeScript-DOM-lib-generator/pull/1993)以及[此处](https://github.com/microsoft/TypeScript-DOM-lib-generator/pull/1940)查看这些变更的更多详情。

## 可展开的悬停提示（预览版）

快速信息（Quick Info，亦称“编辑器工具提示”或“悬停提示”）对于快速查看变量类型或查看类型别名实际指向的内容非常有用。
不过，开发者往往希望能进一步深入查看快速信息提示中展示的更详细内容。
例如，在下述示例中，如果我们将鼠标悬停在参数 `options` 上：

```ts
export function drawButton(options: Options): void
```

提示内容仅显示 `(parameter) options: Options`。

![Tooltip for a parameter declared as `options` which just shows `options: Options`.](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2025/06/bare-hover-5.8-01.png)

难道仅仅为了查看该对象包含哪些成员，我们就必须跳转到类型 `Options` 的定义处吗？

在此之前，确实只能如此。
为了改善这一体验，TypeScript 5.9 目前正在预览一项名为*可展开的悬停提示*（或“快速信息详略控制”）的特性。
如果你使用的是 VS Code 等编辑器，现在会在这些悬停工具提示的左侧看到 `+` 和 `-` 按钮。
点击 `+` 按钮可以更深层次地展开类型详情，而点击 `-` 按钮则会折叠回上一层视图。

<video autoplay loop style="width: 100%;" src="https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2025/06/expandable-quick-info-1.mp4" aria-label="Expanding quick info to see more about the type of `Options`."></video>

该特性目前处于预览阶段，我们正在收集针对 TypeScript 以及 Visual Studio Code 团队的反馈。
有关更多详细信息，请[参见该特性的 PR](https://github.com/microsoft/TypeScript/pull/59940)。

## 可配置的最大悬停长度

有时，快速信息提示可能会变得非常长，以至于 TypeScript 会将其截断以保证可读性。
这样做的缺点在于，悬停提示中最关键的信息往往会被省略掉，令人颇感困扰。
为了解决这一问题，TypeScript 5.9 的语言服务器支持了可配置的悬停长度，在 VS Code 中可以通过 `js/ts.hover.maximumLength` 设置项进行调整。

此外，新的默认悬停长度大幅超越了以往的默认值。
这意味着在 TypeScript 5.9 中，默认情况下你会在悬停提示中看到更丰富的信息。
更多详情请参阅[该特性的 PR](https://github.com/microsoft/TypeScript/pull/61662) 以及[对 Visual Studio Code 的对应改动](https://github.com/microsoft/vscode/pull/248181)。

## 性能优化

### 缓存映射器上的类型实例化

当 TypeScript 将类型参数替换为具体的类型实参时，可能会反复实例化大量相同的中间类型。
在 Zod 和 tRPC 等复杂类型库中，这可能会引发性能问题，甚至报告类型实例化深度过大的错误。
得益于 [Mateusz Burzyński](https://github.com/Andarist) 的[改进](https://github.com/microsoft/TypeScript/pull/61505)，TypeScript 5.9 在对某个特定类型实例化开始处理时，能够缓存许多中间实例化结果。
这进而避免了大量不必要的工作和内存分配。

### 避免在 `fileOrDirectoryExistsUsingSource` 中创建闭包

在 JavaScript 中，函数表达式通常会分配一个新的函数对象，即使该包装函数只是将参数透传给另一个函数且没有捕获任何变量也是如此。
在文件存在性检查相关的代码路径中，[Vincent Bailly](https://github.com/VincentBailly) 发现了这类透传函数调用的实例，而底层函数实际上只接收单个参数。
考虑到大型项目中可能执行的大量存在性检查，他指出这一改进带来了约 11% 的速度提升。
[在此查看有关此变更的更多内容](https://github.com/microsoft/TypeScript/pull/61822/)。

## 显著的行为变更

### `lib.d.ts` 变更

为 DOM 生成的类型可能会对代码库的类型检查产生影响。

此外，一项显著的变化是：`ArrayBuffer` 的类型定义进行了调整，不再作为多种不同 `TypedArray` 类型的超类型。
这也包括 `Uint8Array` 的子类型，例如来自 Node.js 的 `Buffer`。
因此，你可能会看到如下新的错误信息：

```
error TS2345: Argument of type 'ArrayBufferLike' is not assignable to parameter of type 'BufferSource'.
error TS2322: Type 'ArrayBufferLike' is not assignable to type 'ArrayBuffer'.
error TS2322: Type 'Buffer' is not assignable to type 'Uint8Array<ArrayBufferLike>'.
error TS2322: Type 'Buffer' is not assignable to type 'ArrayBuffer'.
error TS2345: Argument of type 'Buffer' is not assignable to parameter of type 'string | Uint8Array<ArrayBufferLike>'.
```

如果遇到与 `Buffer` 相关的问题，你可能首先需要确认是否正在使用最新版本的 `@types/node` 包。
这可能包括运行：

```
npm update @types/node --save-dev
```

在很多情况下，解决方案是指定更明确的底层缓冲区类型，而不是使用默认的 `ArrayBufferLike`（即显式写出 `Uint8Array<ArrayBuffer>` 而非单纯的 `Uint8Array`）。
如果某个 `TypedArray`（如 `Uint8Array`）被传递给期望接收 `ArrayBuffer` 或 `SharedArrayBuffer` 的函数，你也可以尝试访问该 `TypedArray` 的 `buffer` 属性，如下例所示：

```diff
  let data = new Uint8Array([0, 1, 2, 3, 4]);
- someFunc(data)
+ someFunc(data.buffer)
```

## 类型实参推断变更

为了修复推断过程中类型变量的“泄漏”问题，TypeScript 5.9 可能会在某些代码库中引入类型变化，并可能产生新的错误。
这些变化通常难以预先预测，但多数情况下可以通过在泛型函数调用中显式添加类型实参来解决。
[参见此处了解更多详情](https://github.com/microsoft/TypeScript/pull/61668)。
