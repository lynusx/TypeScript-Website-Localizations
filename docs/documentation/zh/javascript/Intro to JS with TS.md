---
title: 结合 TypeScript 编写 JavaScript 简介
layout: docs
permalink: /zh/docs/handbook/intro-to-js-ts.html
oneline: 如何使用 TypeScript 为 JavaScript 文件添加类型检查
translatable: true
---

在处理代码库时，TypeScript 的类型系统具有不同级别的严格程度：

- 仅基于 JavaScript 代码推断的类型系统
- 在 JavaScript 中[通过 JSDoc](/docs/handbook/jsdoc-supported-types.html) 进行渐进式类型标注
- 在 JavaScript 文件中使用 `// @ts-check`
- TypeScript 代码
- 启用了 [`strict`](/tsconfig#strict) 的 TypeScript

每一步都代表着向更安全的类型系统迈进，但并非每个项目都需要这种级别的校验。

## 结合 JavaScript 使用 TypeScript

这是指你所使用的编辑器利用 TypeScript 来提供自动补全、跳转到符号定义以及重命名等重构工具。
[首页](/)列出了配备 TypeScript 插件的编辑器。

## 在 JS 中通过 JSDoc 提供类型提示

在 `.js` 文件中，类型通常可以被自动推断。当无法推断类型时，可以使用 JSDoc 语法来指定类型。

位于声明之前的 JSDoc 注解将用于设定该声明的类型。例如：

```js twoslash
/** @type {number} */
var x

x = 0 // 正常
x = false // 正常？！
```

你可以在 [JSDoc 支持的类型](/docs/handbook/jsdoc-supported-types.html)中查看受支持的 JSDoc 模式的完整列表。

## `@ts-check`

上述代码示例的最后一行在 TypeScript 中会报错，但在 JS 项目中默认不会。
要在 JavaScript 文件中启用错误提示，请在 `.js` 文件的第一行添加 `// @ts-check`，让 TypeScript 将其报告为错误。

```js twoslash
// @ts-check
// @errors: 2322
/** @type {number} */
var x

x = 0 // 正常
x = false // 报错
```

如果你有大量需要开启错误检查的 JavaScript 文件，可以改用 [`jsconfig.json`](/docs/handbook/tsconfig-json.html)。
你可以通过在文件中添加 `// @ts-nocheck` 注释来跳过对特定文件的检查。

TypeScript 报告的某些错误可能并不符合你的预期，遇到这些情况时，可以通过在上一行添加 `// @ts-ignore` 或 `// @ts-expect-error` 来忽略特定行的错误。

```js twoslash
// @ts-check
/** @type {number} */
var x

x = 0 // 正常
// @ts-expect-error
x = false // 报错
```

要了解有关 TypeScript 如何解析 JavaScript 的更多信息，请阅读[在 JavaScript 文件中进行类型检查](/docs/handbook/type-checking-javascript-files.html)。
