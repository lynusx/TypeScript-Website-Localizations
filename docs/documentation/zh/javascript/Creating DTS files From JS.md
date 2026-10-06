---
title: 从 JS 文件生成 DTS 文件
layout: docs
permalink: /zh/docs/handbook/declaration-files/dts-from-js.html
oneline: '如何在 JavaScript 项目中生成 d.ts 文件'
translatable: true
---

[自 TypeScript 3.7 起](/docs/handbook/release-notes/typescript-3-7.html#--declaration-and---allowjs)，TypeScript 增加了对使用 JSDoc 语法从 JavaScript 生成 .d.ts 文件的支持。

这种配置意味着你无需将项目迁移到 TypeScript，也无需在代码库中维护 .d.ts 文件，就能拥有由 TypeScript 驱动的编辑器体验。
TypeScript 支持大多数 JSDoc 标签，你可以在[此处查看参考文档](/docs/handbook/type-checking-javascript-files.html#supported-jsdoc)。

## 配置项目以生成 .d.ts 文件

要在项目中添加 .d.ts 文件的生成，最多只需完成以下四个步骤：

- 将 TypeScript 添加到开发依赖中
- 添加 `tsconfig.json` 来配置 TypeScript
- 运行 TypeScript 编译器，为 JS 文件生成对应的 d.ts 文件
- （可选）编辑 package.json 以引用这些类型

### 添加 TypeScript

你可以在我们的[安装页面](/download)中了解具体操作方法。

### TSConfig

TSConfig 是一个 jsonc 文件，它既用于配置编译器标志，也用于声明在何处查找文件。
在本场景中，你需要一个如下所示的文件：

```jsonc tsconfig
{
  // 修改此处以匹配你的项目
  "include": ["src/**/*"],

  "compilerOptions": {
    // 告知 TypeScript 读取 JS 文件，因为
    // 它们在通常情况下不会被作为源文件处理
    "allowJs": true,
    // 生成 d.ts 文件
    "declaration": true,
    // 本次编译器运行应
    // 仅输出 d.ts 文件
    "emitDeclarationOnly": true,
    // 类型文件应输出到此目录。
    // 移除此项会将 .d.ts 文件
    // 放置在 .js 文件的旁边
    "outDir": "dist",
    // 在 VSCode 中使用类似“跳转到定义”等 IDE 功能时
    // 跳转到 js 文件
    "declarationMap": true,
  },
}
```

你可以在 [tsconfig 参考文档](/tsconfig)中了解有关这些选项的更多信息。
除了使用 TSConfig 文件外，也可以使用 CLI，以下 CLI 命令具有相同的行为：

```sh
npx -p typescript tsc src/**/*.js --declaration --allowJs --emitDeclarationOnly --outDir types
```

## 运行编译器

你可以在我们的[安装页面](/download)中了解如何运行编译器。
如果这些生成的文件被包含在项目的 `.gitignore` 中，你需要确保在打包发布时包含它们。

## 编辑 package.json

TypeScript 复用了 Node 在 `package.json` 中的模块解析机制，并增加了一个查找 .d.ts 文件的额外步骤。
大致来说，解析过程会先检查可选的 `types` 字段，然后检查 `"main"` 字段，最后尝试根目录下的 `index.d.ts`。

| Package.json              | 默认 .d.ts 的位置              |
| :------------------------ | :----------------------------- |
| 无 "types" 字段           | 检查 "main"，然后是 index.d.ts |
| "types": "main.d.ts"      | main.d.ts                      |
| "types": "./dist/main.js" | ./dist/main.d.ts               |

如果未指定，则使用 "main"：

| Package.json             | 默认 .d.ts 的位置 |
| :----------------------- | :---------------- |
| 无 "main" 字段           | index.d.ts        |
| "main":"index.js"        | index.d.ts        |
| "main":"./dist/index.js" | ./dist/index.d.ts |

## 提示

如果你想为 .d.ts 文件编写测试，可以尝试使用 [tsd](https://github.com/SamVerschueren/tsd) 或 [TSTyche](https://github.com/tstyche/tstyche)。
