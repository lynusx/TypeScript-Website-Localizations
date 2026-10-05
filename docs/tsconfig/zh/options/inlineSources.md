---
display: 'Inline Sources'
oneline: '将源代码作为内联内容包含在生成的 JavaScript 中的 source map 内。'
---

设置后，TypeScript 会将 `.ts` 文件的原始内容作为嵌入字符串包含在 source map 中（使用 source map 的 `sourcesContent` 属性）。
这在与 [`inlineSourceMap`](#inlineSourceMap) 类似的场景下通常非常有用。

需要同时设置 [`sourceMap`](#sourceMap) 或 [`inlineSourceMap`](#inlineSourceMap)。

例如，对于以下 TypeScript 代码：

```ts twoslash
const helloWorld = 'hi'
console.log(helloWorld)
```

默认会转换为如下 JavaScript：

```ts twoslash
// @showEmit
const helloWorld = 'hi'
console.log(helloWorld)
```

启用 `inlineSources` 以及 [`inlineSourceMap`](#inlineSourceMap) 进行构建后，文件底部会包含一段携带 source map 的注释。
请注意，结尾部分与 [`inlineSourceMap`](#inlineSourceMap) 中的示例有所不同，因为此时 source map 还包含了原始源代码。

```ts twoslash
// @inlineSources
// @inlineSourceMap
// @showEmit
const helloWorld = 'hi'
console.log(helloWorld)
```
