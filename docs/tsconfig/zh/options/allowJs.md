---
display: 'Allow JS'
oneline: '允许 JavaScript 文件成为程序的一部分。使用 `checkJS` 选项可从这些文件中获取报错信息。'
---

允许在项目中导入 JavaScript 文件，而不仅限于 `.ts` 和 `.tsx` 文件。例如，以下 JS 文件：

```js twoslash
// @filename: card.js
export const defaultCardDeck = 'Heart'
```

当导入到 TypeScript 文件时会产生报错：

```ts twoslash
// @errors: 2307
// @filename: card.js
module.exports.defaultCardDeck = 'Heart'
// ---cut---
// @filename: index.ts
import { defaultCardDeck } from './card'

console.log(defaultCardDeck)
```

启用 `allowJs` 后即可正常导入：

```ts twoslash
// @filename: card.js
module.exports.defaultCardDeck = 'Heart'
// ---cut---
// @allowJs
// @filename: index.ts
import { defaultCardDeck } from './card'

console.log(defaultCardDeck)
```

该标志可用于逐步将 TypeScript 文件引入 JS 项目，使 `.ts` 和 `.tsx` 文件能与现有 JavaScript 文件共存。

它还可以与 [`declaration`](#declaration) 和 [`emitDeclarationOnly`](#emitDeclarationOnly) 配合使用，为 JS 文件[生成声明文件](/docs/handbook/declaration-files/dts-from-js.html)。
