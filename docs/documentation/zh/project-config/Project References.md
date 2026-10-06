---
title: 项目引用（Project References）
layout: docs
permalink: /zh/docs/handbook/project-references.html
oneline: 如何拆分大型 TypeScript 项目
translatable: true
---

项目引用（Project references）允许你将 TypeScript 程序拆分为更小的组成部分，该特性在 TypeScript 3.0 及更高版本中可用。

通过使用项目引用，你可以大幅缩短构建时间、强制组件之间的逻辑解耦，并以更优秀的新方式来组织代码。

我们还为 `tsc` 引入了一种全新模式——`--build` 标志，它与项目引用配合使用，从而实现更快的 TypeScript 构建。

## 示例项目

让我们来看一个相当常见的程序，了解项目引用如何帮助我们更好地组织它。
假设你有一个包含 `converter` 和 `units` 两个模块的项目，并且每个模块都有对应的测试文件：

```
/
├── src/
│   ├── converter.ts
│   └── units.ts
├── test/
│   ├── converter-tests.ts
│   └── units-tests.ts
└── tsconfig.json
```

测试文件导入实现文件并执行某些测试：

```ts
// converter-tests.ts
import * as converter from '../src/converter'

assert.areEqual(converter.celsiusToFahrenheit(0), 32)
```

在以前，如果只使用单个 tsconfig 文件，这种结构处理起来会相当棘手：

- 实现文件可能会错误地导入测试文件
- 无法在同时构建 `test` 和 `src` 的同时避免 `src` 出现在输出目录名称中（这通常不是你想要的结果）
- 仅修改实现文件的*内部实现*，也需要重新对测试文件进行*类型检查*，即便这绝不会导致新的错误
- 仅修改测试文件，也需要重新对实现文件进行类型检查，即便实现代码没有任何变化

你也可以使用多个 tsconfig 文件来解决其中的*部分*问题，但又会带来新的问题：

- 缺乏内置的新鲜度检查（up-to-date checking），导致你总是需要运行两次 `tsc`
- 调用两次 `tsc` 会产生额外的启动时间开销
- `tsc -w` 无法同时监听多个配置文件

而项目引用可以解决上述所有问题，甚至带来更多好处。

## 什么是项目引用？

`tsconfig.json` 文件新增了一个顶层属性 [`references`](/tsconfig#references)。它是一个对象数组，用于指定要引用的项目：

```js tsconfig
{
    "compilerOptions": {
        // The usual
    },
    "references": [
        { "path": "../src" }
    ]
}
```

每个引用的 `path` 属性可以指向包含 `tsconfig.json` 文件的目录，也可以直接指向配置文件本身（配置文件可以使用任意名称）。

当你引用一个项目时，会产生以下新行为：

- 从被引用的项目中导入模块时，将改为加载其*输出*声明文件（`.d.ts`）
- 如果被引用的项目生成了 [`outFile`](/tsconfig#outFile)，则该输出文件 `.d.ts` 中的声明将在此项目中可见
- 构建模式（见下文）会在需要时自动构建被引用的项目

通过拆分为多个项目，你可以大幅提升类型检查和编译的速度，降低在编辑器中使用的内存占用，并更好地强化程序的逻辑分组。

## `composite`

被引用的项目必须启用新增的 [`composite`](/tsconfig#composite) 设置。
这项设置是必须的，以确保 TypeScript 能够快速确定在哪里找到被引用项目的输出产物。
启用 [`composite`](/tsconfig#composite) 标志会带来以下变化：

- 如果未显式设置 [`rootDir`](/tsconfig#rootDir)，其默认值为包含该 `tsconfig` 文件的目录
- 所有实现文件都必须与 [`include`](/tsconfig#include) 匹配模式相匹配，或列在 [`files`](/tsconfig#files) 数组中。如果违反了此约束，`tsc` 会提示你哪些文件未被指定
- 必须开启 [`declaration`](/tsconfig#declaration)

## `declarationMap`

我们还添加了对[声明源映射（declaration source maps）](https://github.com/Microsoft/TypeScript/issues/14479)的支持。
如果启用 [`declarationMap`](/tsconfig#declarationMap)，在受支持的编辑器中，你将能够使用诸如“跳转到定义（Go to Definition）”和重命名等编辑器功能，无缝跨越项目边界导航并编辑代码。

## 项目引用的注意事项

项目引用存在一些需要注意的权衡取舍。

因为依赖其他项目的工程需要使用从其依赖项构建生成的 `.d.ts` 文件，所以你必须要么将某些构建输出检入版本控制，*要么*在克隆项目后先构建一次项目，然后才能在编辑器中正常浏览项目而不会看到虚假报错。

在使用 VS Code 时（自 TS 3.7 起），我们在后台提供了一个内存中生成 `.d.ts` 的处理流程，这应该能够缓解该问题，但它会对性能产生一定影响。对于超大型 composite 项目，你可能需要通过 [disableSourceOfProjectReferenceRedirect 选项](/tsconfig#disableSourceOfProjectReferenceRedirect)将其禁用。

此外，为了保持与现有构建工作流的兼容性，除非使用 `--build` 参数调用，否则 `tsc` *不会*自动构建依赖项。
接下来让我们深入了解 `--build`。

## TypeScript 的构建模式

许多人期待已久的一项功能是面向 TypeScript 项目的智能增量构建（incremental builds）。
在 3.0 中，你可以为 `tsc` 使用 `--build` 标志。
这实际上是 `tsc` 的一个新入口点，其行为更像是一个构建编排器（build orchestrator），而不仅仅是一个简单的编译器。

运行 `tsc --build`（简写为 `tsc -b`）将执行以下操作：

- 查找所有被引用的项目
- 检测它们是否是最新的（up-to-date）
- 按照正确的顺序构建已过期的项目

你可以为 `tsc -b` 提供多个配置文件路径（例如 `tsc -b src test`）。
就像 `tsc -p` 一样，如果配置文件名为 `tsconfig.json`，则无需显式指定文件名本身。

### `tsc -b` 命令行

你可以指定任意数量的配置文件：

```shell
 > tsc -b                            # Use the tsconfig.json in the current directory
 > tsc -b src                        # Use src/tsconfig.json
 > tsc -b foo/prd.tsconfig.json bar  # Use foo/prd.tsconfig.json and bar/tsconfig.json
```

无需担心在命令行上传递文件的先后顺序——`tsc` 会在需要时重新对它们进行排序，以确保依赖项始终最先构建。

`tsc -b` 还提供了一些专属标志：

- [`--verbose`](/tsconfig#verbose)：打印详细日志以解释当前正在发生的事情（可与其他任何标志组合使用）
- `--dry`：显示将要执行的操作，但实际上不构建任何内容
- `--clean`：删除指定项目的输出文件（可与 `--dry` 组合使用）
- [`--force`](/tsconfig#force)：将所有项目均视为已过期并重新构建
- `--watch`：监听模式（除 [`--verbose`](/tsconfig#verbose) 外，不可与其他任何标志组合使用）

## 注意事项

通常情况下，除非开启了 [`noEmitOnError`](/tsconfig#noEmitOnError)，否则即使存在语法或类型错误，`tsc` 也会生成输出产物（`.js` 和 `.d.ts`）。
在增量构建系统中这样做会非常糟糕——如果某个过期的依赖项出现了新的错误，你将只能看到该错误*一次*，因为后续构建会跳过构建当前已是最新状态的项目。
出于这个原因，`tsc -b` 的实际行为相当于对所有项目都默认启用了 [`noEmitOnError`](/tsconfig#noEmitOnError)。

如果你将任何构建输出（`.js`、`.d.ts`、`.d.ts.map` 等）检入了版本控制，则在执行某些版本控制操作后，可能需要运行一次 [`--force`](/tsconfig#force) 构建，具体取决于你的版本控制工具是否会在本地副本和远程副本之间保留时间戳。

## MSBuild

如果你使用的是 msbuild 项目，可以在项目文件中添加：

```xml
    <TypeScriptBuildMode>true</TypeScriptBuildMode>
```

来启用构建模式。这将开启自动增量构建以及清理功能。

请注意，与 `tsconfig.json` / `-p` 一样，原有的 TypeScript 项目属性将不再生效——所有设置都应通过 tsconfig 文件进行管理。

一些团队构建了基于 msbuild 的工作流，其中 tsconfig 文件与其配对的托管项目具有相同的*隐式*图顺序。
如果你的解决方案正是如此，你可以继续将 `msbuild`、`tsc -p` 与项目引用结合使用；它们之间完全互操作。

## 使用指南

### 整体结构

随着 `tsconfig.json` 文件的增多，你通常会希望使用[配置文件继承](/docs/handbook/tsconfig-json.html)来集中管理通用的编译器选项。
这样你只需修改一个文件中的设置，而无需逐一编辑多个文件。

另一个良好的实践是创建一个“解决方案（solution）”`tsconfig.json` 文件，它仅包含指向所有叶子节点项目的 [`references`](/tsconfig#references)，并将 [`files`](/tsconfig#files) 设置为空数组（否则该解决方案文件会导致文件被重复编译）。请注意，从 3.0 开始，如果 `tsconfig.json` 文件中包含至少一个 `reference`，那么 [`files`](/tsconfig#files) 数组为空不再被视为错误。

这提供了一个简洁的入口点；例如在 TypeScript 自身的代码仓库中，我们只需运行 `tsc -b src` 即可构建所有端点目标，因为我们在 `src/tsconfig.json` 中列出了所有的子项目。

你可以在 TypeScript 代码仓库中看到这些模式——以 `src/tsconfig-base.json`、`src/tsconfig.json` 和 `src/tsc/tsconfig.json` 作为核心示例。

### 针对相对模块的项目结构设计

通常情况下，迁移使用相对模块的代码仓库并不需要做太多调整。
只需在指定父目录的每个子目录中放置一个 `tsconfig.json` 文件，并向这些配置文件添加 `reference` 以匹配程序预期的分层结构即可。
你需要将 [`outDir`](/tsconfig#outDir) 设置为输出目录的显式子文件夹，或者将 [`rootDir`](/tsconfig#rootDir) 设置为所有项目文件夹的公共根目录。

### 针对 outFile 的项目结构设计

使用 [`outFile`](/tsconfig#outFile) 进行编译时的结构布局更加灵活，因为相对路径不再那么重要。
TypeScript 仓库本身就是一个很好的参考——我们拥有若干“库（library）”项目和若干“端点（endpoint）”项目；端点项目保持尽可能轻量，仅引入它们所需的库。

<!--
### Structuring for monorepos

TODO: Experiment more and figure this out. Rush and Lerna seem to have different models that imply different things on our end
-->
