---
title: 类型声明
layout: docs
permalink: /zh/docs/handbook/2/type-declarations.html
oneline: 'TypeScript 如何为没有类型的 JavaScript 代码提供类型。'
---

在前面的章节中，我们一直使用所有 JavaScript 运行时内置的函数来演示 TypeScript 的基本概念。
然而，如今几乎所有的 JavaScript 项目都会引入许多第三方库来完成常见任务。
为你应用程序中*非自己编写的代码*部分提供类型，将极大提升你的 TypeScript 开发体验。
那么，这些类型是从何而来的呢？

## 类型声明长什么样？

假设你写了这样一段代码：

```ts twoslash
// @errors: 2339
const k = Math.max(5, 6)
const j = Math.mix(7, 8)
```

尽管 `Math` 的具体实现并不属于你编写的代码，TypeScript 又是如何知道 `max` 存在而 `mix` 不存在的呢？

答案在于：存在用来描述这些内置对象的*声明文件*（declaration files）。
声明文件提供了一种方法，可以在不提供实际实现的情况下，*声明*某些类型或值的存在。

## `.d.ts` 文件

TypeScript 主要有两种类型的文件。
`.ts` 文件是*实现*文件，包含类型和可执行代码。
这些文件会编译输出 `.js` 文件，也是你平时编写代码的地方。

`.d.ts` 文件是*声明*文件，*仅*包含类型信息。
这些文件不会生成 `.js` 输出，它们仅用于类型检查。
后续我们还会深入学习如何编写自定义声明文件。

## 内置类型定义

TypeScript 为 JavaScript 运行时中所有标准内置 API 都提供了声明文件。
这包括像 `string` 或 `function` 等内置类型的方法和属性、`Math` 和 `Object` 等顶层全局对象，以及它们关联的类型。
默认情况下，TypeScript 还包含了在浏览器环境下运行时的可用类型，例如 `window` 和 `document`，这些统称为 DOM API。

TypeScript 以 `lib.[something].d.ts` 的命名规则来命名这些声明文件。
如果你在编辑器中跳转到了带有此类名称的文件，就会知道自己正在查看平台的内置部分，而不是用户代码。

### `target` 配置项

代码中可用的方法、属性和函数，实际上取决于代码所运行的 JavaScript _版本_。
例如，字符串的 `startsWith` 方法仅从被称为 _ECMAScript 6_ 的 JavaScript 版本开始提供。

清楚代码最终运行在哪个 JavaScript 版本上非常重要，因为你不希望使用比部署目标平台版本更高的 API。
这也正是 [`target`](/tsconfig#target) 编译器配置项的作用之一。

TypeScript 通过根据你的 [`target`](/tsconfig#target) 配置项默认包含不同的 `lib` 文件来解决这个问题。
例如，如果 [`target`](/tsconfig#target) 设置为 `ES5`，当你尝试使用 `startsWith` 方法时就会报错，因为该方法仅在 `ES6` 或更高版本中可用。

### `lib` 配置项

[`lib`](/tsconfig#lib) 配置项允许你对程序中可用的内置声明文件进行更细粒度的控制。
有关详细信息，请参阅 [`lib`](/tsconfig#lib) 文档页面。

## 外部定义

对于非内置 API，有多种途径可以获取声明文件。
具体采用哪种方式取决于你正在为哪个库获取类型。

### 内置捆绑类型（Bundled Types）

如果你使用的库作为 npm 包发布，它可能已经在分发包中包含了类型声明文件。
你可以查阅该项目的文档了解，或者直接尝试导入该包，看看 TypeScript 能否自动为你解析类型。

如果你是一名包作者并打算随包捆绑类型定义，可以阅读我们的[捆绑类型定义指南](/docs/handbook/declaration-files/publishing.html#including-declarations-in-your-npm-package)。

### DefinitelyTyped / `@types`

[DefinitelyTyped 仓库](https://github.com/DefinitelyTyped/DefinitelyTyped/)是一个集中式代码仓库，收录了成千上万个库的声明文件。
绝大多数常用库都可以在 DefinitelyTyped 上找到声明文件。

DefinitelyTyped 上的类型定义还会自动发布到 npm 的 `@types` 作用域下。
类型包的名称始终与对应库本身的名称一致。
例如，如果你安装了 `react` npm 包，可以通过运行以下命令来安装对应的类型：

```sh
npm install --save-dev @types/react
```

TypeScript 会自动在 `node_modules/@types` 下查找类型定义，因此无需任何额外步骤即可在程序中使用这些类型。

### 自定义定义

在极少数情况下，如果某个库既没有自带类型，也没有在 DefinitelyTyped 上提供定义，你可以自己编写声明文件。
相关指导请参阅附录[编写声明文件](/docs/handbook/declaration-files/introduction.html)。

如果你只是想消除关于某个模块的警告而不打算编写完整的声明文件，也可以在项目的 `.d.ts` 文件中为该模块添加一个空声明，将其快速声明为 `any` 类型。
例如，如果你想使用名为 `some-untyped-module` 的无类型模块，可以这样写：

```ts twoslash
declare module 'some-untyped-module'
```
