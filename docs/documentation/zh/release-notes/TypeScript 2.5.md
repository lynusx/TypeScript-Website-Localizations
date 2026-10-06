---
title: TypeScript 2.5
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-2-5.html
oneline: TypeScript 2.5 发布说明
---

## `catch` 子句中的可选变量

感谢 [@tinganho](https://github.com/tinganho) 的贡献，TypeScript 2.5 实现了一项新的 ECMAScript 特性，允许用户省略 `catch` 子句中的变量声明。例如，在使用 `JSON.parse` 时，你可能需要将调用包裹在 `try`/`catch` 中，但当输入有误时抛出的 `SyntaxError` 却不一定需要用到。

```ts
let input = '...'
try {
  JSON.parse(input)
} catch {
  // ^ Notice that our `catch` clause doesn't declare a variable.
  console.log('Invalid JSON given\n\n' + input)
}
```

## 在 `checkJs`/`@ts-check` 模式下的类型断言/类型转换语法

TypeScript 2.5 引入了在纯 JavaScript 项目中[对表达式进行类型断言](https://github.com/Microsoft/TypeScript/issues/5158)的能力。语法为在括号表达式前加上 `/** @type {...} */` 注解注释，用于重新评估该表达式的类型。例如：

```ts
var x = /** @type {SomeType} */ AnyParenthesizedExpression
```

## 包的去重与重定向

在 TypeScript 2.5 中使用 `Node` 模块解析策略时，编译器现在会检查文件是否来自"相同"的包。若某个文件来自的包，其 `package.json` 中的 `name` 和 `version` 字段与此前已遇到的包完全一致，TypeScript 会将其重定向到最顶层的那个包。这有助于解决两个包可能包含相同类声明，但因含有 `private` 成员而导致结构不兼容的问题。

额外的好处是，这还可以减少编译器和语言服务的内存占用与运行时开销，因为来自重复包的 `.d.ts` 文件不会被重复加载。

## `--preserveSymlinks` 编译器标志

TypeScript 2.5 引入了 [`preserveSymlinks`](/tsconfig#preserveSymlinks) 标志，其行为与 [Node.js 的 `--preserve-symlinks` 标志](https://nodejs.org/api/cli.html#cli_preserve_symlinks)一致。该标志的行为与 Webpack 的 `resolve.symlinks` 选项相反（即 TypeScript 的 [`preserveSymlinks`](/tsconfig#preserveSymlinks) 设为 `true` 相当于 Webpack 的 `resolve.symlinks` 设为 `false`，反之亦然）。

在此模式下，对模块和包的引用（例如 `import` 和 `/// <reference type="..." />` 指令）均相对于符号链接文件所在的位置解析，而非相对于符号链接所指向的实际路径解析。具体示例请参阅 [Node.js 网站的相关文档](https://nodejs.org/api/cli.html#cli_preserve_symlinks)。
