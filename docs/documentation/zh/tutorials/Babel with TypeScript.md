---
title: 结合 TypeScript 使用 Babel
layout: docs
permalink: /zh/docs/handbook/babel-with-typescript.html
oneline: 如何创建 Babel + TypeScript 混合项目
translatable: true
---

## 针对 TypeScript 的 Babel 与 `tsc` 对比

在构建现代 JavaScript 项目时，你可能会问自己：将文件从 TypeScript 转换为 JavaScript 的正确方式是什么？

很多时候，答案是*“视情况而定”_，或者取决于项目本身，_“可能已经有人替你做好了决定”*。如果你使用现有的框架构建项目，例如 [tsdx](https://tsdx.io)、[Angular](https://angular.io/)、[NestJS](https://nestjs.com/) 或[快速入门](/docs)中提到的任何框架，那么这个选择已经由框架为你代劳了。

不过，一个实用的经验法则是：

- 你的构建产物是否与源输入文件基本一致？使用 `tsc`
- 你是否需要一个具有多种潜在输出的构建管线？使用 `babel` 进行转译，使用 `tsc` 进行类型检查

## 使用 Babel 进行转译，使用 `tsc` 进行类型检查

对于那些拥有既有构建基础设施、且可能是从 JavaScript 代码库迁移到 TypeScript 的项目来说，这是一种常见的模式。

这种技术属于混合方案：使用 Babel 的 [preset-typescript](https://babeljs.io/docs/en/babel-preset-typescript) 生成 JS 文件，然后使用 TypeScript 进行类型检查并生成 `.d.ts` 文件。

利用 Babel 对 TypeScript 的支持，你可以与现有的构建管线协同工作，并且很可能获得更快的 JS 生成速度，因为 Babel 不会对代码进行类型检查。

#### 类型检查与 d.ts 文件生成

使用 Babel 的缺点在于，从 TS 转换为 JS 的过程中不会进行类型检查。这意味着你在编辑器中遗漏的类型错误可能会悄悄溜进生产环境代码中。

此外，Babel 无法为你的 TypeScript 代码生成 `.d.ts` 文件；如果你的项目是一个库，这会让项目的使用变得更加困难。

为了解决这些问题，你可能需要配置一条命令来使用 TSC 对项目进行类型检查。这通常意味着需要将部分 Babel 配置同步到对应的 [`tsconfig.json`](/tsconfig) 中，并确保启用了以下配置项：

```json tsconfig
"compilerOptions": {
  // Ensure that .d.ts files are created by tsc, but not .js files
  "declaration": true,
  "emitDeclarationOnly": true,
  // Ensure that Babel can safely transpile files in the TypeScript project
  "isolatedModules": true
}
```

关于这些标志的更多信息：

- [`isolatedModules`](/tsconfig#isolatedModules)
- [`declaration`](/tsconfig#declaration), [`emitDeclarationOnly`](/tsconfig#emitDeclarationOnly)
