---
display: 'Remove Comments'
oneline: '禁止生成注释。'
---

在转换为 JavaScript 时删除 TypeScript 文件中的所有注释。默认值为 `false`。

例如，这是一个带有 JSDoc 注释的 TypeScript 文件：

```ts
/** 'Hello world' 的葡萄牙语翻译 */
export const helloWorldPTBR = 'Olá Mundo'
```

当 `removeComments` 设置为 `true` 时：

```ts twoslash
// @showEmit
// @removeComments: true
/** 'Hello world' 的葡萄牙语翻译 */
export const helloWorldPTBR = 'Olá Mundo'
```

未设置 `removeComments` 或将其设置为 `false` 时：

```ts twoslash
// @showEmit
// @removeComments: false
/** 'Hello world' 的葡萄牙语翻译 */
export const helloWorldPTBR = 'Olá Mundo'
```

这意味着注释会保留在生成的 JavaScript 代码中。
