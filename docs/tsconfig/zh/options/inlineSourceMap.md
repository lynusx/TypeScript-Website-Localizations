---
display: 'Inline Source Map'
oneline: '在生成的 JavaScript 文件中内联包含 sourcemap 内容。'
---

设置后，TypeScript 不会输出单独的 `.js.map` 文件来提供源映射（source map），而是会将 source map 内容直接内嵌到生成的 `.js` 文件中。
尽管这会导致 JS 文件体积变大，但在某些场景下非常方便。
例如，你可能希望在不允许提供 `.map` 文件访问的 Web 服务器上调试 JS 文件。

与 [`sourceMap`](#sourceMap) 互斥。

例如，如下 TypeScript 代码：

```ts
const helloWorld = 'hi'
console.log(helloWorld)
```

转换为如下 JavaScript 代码：

```ts twoslash
// @showEmit
const helloWorld = 'hi'
console.log(helloWorld)
```

接着，在启用 `inlineSourceMap` 进行构建时，文件底部会包含一段注释，其中包含了该文件的 source map：

```ts twoslash
// @inlineSourceMap
// @showEmit
const helloWorld = 'hi'
console.log(helloWorld)
```
