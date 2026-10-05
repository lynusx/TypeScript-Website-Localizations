---
display: 'Source Map'
oneline: '为生成的 JavaScript 文件创建 source map 文件。'
---

启用 [source map 文件](https://developer.mozilla.org/docs/Tools/Debugger/How_to/Use_a_source_map)的生成。
这些文件使调试器及其他工具在实际处理生成的 JavaScript 文件时，能够展示原始的 TypeScript 源代码。
Source map 文件会作为 `.js.map`（或 `.jsx.map`）文件输出在对应的 `.js` 输出文件旁边。

随后，`.js` 文件中将包含一条 source map 注释，用以向外部工具指示文件的位置，例如：

```ts
// helloWorld.ts
export declare const helloWorld = 'hi'
```

在 `sourceMap` 设置为 `true` 的情况下进行编译，将生成如下 JavaScript 文件：

```js
// helloWorld.js
'use strict'
Object.defineProperty(exports, '__esModule', { value: true })
exports.helloWorld = 'hi'
//# sourceMappingURL=// helloWorld.js.map
```

同时还会生成如下 JSON map 文件：

```json
// helloWorld.js.map
{
  "version": 3,
  "file": "ex.js",
  "sourceRoot": "",
  "sources": ["../ex.ts"],
  "names": [],
  "mappings": ";;AAAa,QAAA,UAAU,GAAG,IAAI,CAAA"
}
```
