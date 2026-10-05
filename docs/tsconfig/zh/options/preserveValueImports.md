---
display: 'Preserve Value Imports'
oneline: '在 JavaScript 输出中保留未使用的导入值（否则这些值会被移除）。'
---

已废弃，请改用 [`verbatimModuleSyntax`](#verbatimModuleSyntax)。

在某些情况下，TypeScript 无法检测到你正在使用某个导入。例如，考虑以下代码：

```ts
import { Animal } from './animal.js'

eval('console.log(new Animal().isDangerous())')
```

或者使用如 Svelte、Vue 这类“编译为 HTML”的语言编写的代码。`preserveValueImports` 将阻止 TypeScript 移除该导入，即便它看起来未被使用。

当与 [`isolatedModules`](#isolatedModules) 结合使用时：导入的类型必须（_must_）被显式标记为仅类型（type-only），因为每次仅处理单个文件的编译器无法知道导入项是表面上未使用的值，还是为了避免运行时崩溃而必须移除的类型。
