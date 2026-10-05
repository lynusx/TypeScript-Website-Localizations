---
display: 'Check JS'
oneline: '在进行类型检查的 JavaScript 文件中启用错误报告。'
---

与 [`allowJs`](#allowJs) 配合使用。启用 `checkJs` 后，会在 JavaScript 文件中报告错误。这相当于在项目中包含的所有 JavaScript 文件顶部都添加了 `// @ts-check`。

例如，根据 TypeScript 内置的 `parseFloat` 类型定义，以下是一段不正确的 JavaScript 代码：

```js
// parseFloat only takes a string
module.exports.pi = parseFloat(3.142)
```

当导入到 TypeScript 模块中时：

```ts twoslash
// @allowJs
// @filename: constants.js
module.exports.pi = parseFloat(3.142)

// @filename: index.ts
import { pi } from './constants'
console.log(pi)
```

你不会收到任何报错。但是，如果开启 `checkJs`，你将收到来自该 JavaScript 文件的错误提示。

```ts twoslash
// @errors: 2345
// @allowjs: true
// @checkjs: true
// @filename: constants.js
module.exports.pi = parseFloat(3.142)

// @filename: index.ts
import { pi } from './constants'
console.log(pi)
```
