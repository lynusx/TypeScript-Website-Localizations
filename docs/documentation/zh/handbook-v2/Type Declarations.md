---
title: 类型声明
layout: docs
permalink: /zh/docs/handbook/2/type-declarations.html
oneline: 'TypeScript 如何为无类型的 JavaScript 提供类型。'
---

在前面阅读的章节中，我们一直使用所有 JavaScript 运行时中都存在的内置函数来演示基本的 TypeScript 概念。然而，如今几乎所有的 JavaScript 项目都会引入许多库来完成常规任务。为应用程序中 _不是_ 你编写的代码提供类型，将极大地提升 TypeScript 的开发体验。这些类型从何而来？

## 类型声明长什么样？

假设你编写了如下代码：

```ts twoslash
// @errors: 2339
const k = Math.max(5, 6)
const j = Math.mix(7, 8)
```

尽管 `Math` 的实现并不是你代码的一部分，TypeScript 是如何知道 `max` 存在而 `mix` 不存在的呢？

答案是，存在描述这些内置对象的 _声明文件_。声明文件提供了一种方式，可以 _声明_ 某些类型或值的存在，而无需提供这些值的具体实现。

## `.d.ts` 文件

TypeScript 主要有两种类型的文件。`.ts` 文件是包含类型和可执行代码的 _实现_ 文件。这些文件会产生 `.js` 输出，也是你平时编写代码的地方。

`.d.ts` 文件是 _仅_ 包含类型信息的 _声明文件_。这些文件不会产生 `.js` 输出；它们仅用于类型检查。稍后我们将进一步学习如何编写自己的声明文件。

## 内置类型定义

TypeScript 为 JavaScript 运行时中所有标准化的内置 API 提供了声明文件。这包括像 `string` 或 `function` 这样的内置类型的方法和属性、像 `Math` 和 `Object` 这样的顶层名称，以及它们关联的类型。默认情况下，TypeScript 还包含在浏览器中运行时可用的类型，例如 `window` 和 `document`；它们被统称为 DOM API。

TypeScript 以 `lib.[something].d.ts` 的模式命名这些声明文件。如果你打开一个这种名字的文件，就可以知道你正在查看的是平台的某个内置部分，而不是用户代码。

### `target` 设置

实际上，你能使用哪些方法、属性和函数，取决于运行代码的 JavaScript 版本。例如，字符串的 `startsWith` 方法仅从被称为 _ECMAScript 6_ 的 JavaScript 版本才开始提供。

清楚你的代码最终运行在哪个版本的 JavaScript 上非常重要，因为你不希望使用的 API 版本高于要部署的目标平台。这就是 [`target`](/tsconfig#target) 编译器设置的功能之一。

TypeScript 会根据你的 [`target`](/tsconfig#target) 设置，改变默认包含哪些 `lib` 文件来帮助解决这个问题。例如，如果 [`target`](/tsconfig#target) 是 `ES5`，你在尝试使用 `startsWith` 方法时会看到错误，因为该方法仅在 `ES6` 或更高版本中可用。

### `lib` 设置

[`lib`](/tsconfig#lib) 设置允许更细粒度地控制哪些内置声明文件在程序中可用。请参阅 [`lib`](/tsconfig#lib) 文档页面以获取更多信息。

## 外部类型定义

对于非内置的 API，获取声明文件的方法有很多。具体做法取决于你要为哪个库获取类型。

### 捆绑的类型

如果你使用的库作为 npm 包发布，它可能已经将类型声明文件包含在其分发包中了。你可以通过阅读项目文档来确认，或者直接尝试导入该包，看 TypeScript 能否自动解析类型。如果你是包作者，正考虑将类型定义与包捆绑在一起，可以阅读我们的 [捆绑类型定义](/docs/handbook/declaration-files/publishing.html#including-declarations-in-your-npm-package) 指南。

### DefinitelyTyped / `@types`

[DefinitelyTyped 仓库](https://github.com/DefinitelyTyped/DefinitelyTyped/) 是一个集中存储数千个库声明文件的仓库。绝大多数常用库在 DefinitelyTyped 上都有可用的声明文件。

DefinitelyTyped 上的定义还会自动以 `@types` 作用域发布到 npm。类型包的名称始终与底层包本身的名称相同。例如，如果你安装了 `react` 这个 npm 包，你可以通过运行以下命令安装相应的类型：

```sh
npm install --save-dev @types/react
```

TypeScript 会自动查找 `node_modules/@types` 下的类型定义，因此无需额外步骤即可让这些类型在程序中生效。

### 自己编写定义

如果遇到罕见情况：某个库既没有捆绑自己的类型，在 DefinitelyTyped 上也没有定义，你可以自己编写声明文件。请参阅附录 [编写声明文件](/docs/handbook/declaration-files/introduction.html) 获取相关指南。

如果你只想在不编写声明文件的情况下消除关于某个特定模块的警告，也可以在项目的 `.d.ts` 文件中为该模块放置一个空声明，快速将其声明为 `any` 类型。例如，如果你想在没有定义的情况下使用名为 `some-untyped-module` 的模块，你可以这样写：

```ts twoslash
declare module 'some-untyped-module'
```
