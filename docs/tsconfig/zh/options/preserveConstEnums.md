---
display: 'Preserve Const Enums'
oneline: '禁止在生成的代码中擦除 `const enum` 声明。'
---

不在生成的代码中擦除 `const enum` 声明。`const enum` 通过直接输出枚举值而非引用，提供了一种减少应用程序运行时总内存占用的方式。

例如以下 TypeScript 代码：

```ts twoslash
const enum Album {
  JimmyEatWorldFutures = 1,
  TubRingZooHypothesis = 2,
  DogFashionDiscoAdultery = 3,
}

const selectedAlbum = Album.JimmyEatWorldFutures
if (selectedAlbum === Album.JimmyEatWorldFutures) {
  console.log('That is a great choice.')
}
```

`const enum` 的默认行为是将任何 `Album.Something` 转换为对应的数值字面量，并从 JavaScript 中完全移除对该枚举的引用。

```ts twoslash
// @showEmit
const enum Album {
  JimmyEatWorldFutures = 1,
  TubRingZooHypothesis = 2,
  DogFashionDiscoAdultery = 3,
}

const selectedAlbum = Album.JimmyEatWorldFutures
if (selectedAlbum === Album.JimmyEatWorldFutures) {
  console.log('That is a great choice.')
}
```

将 `preserveConstEnums` 设置为 `true` 后，`enum` 会在运行时保留，并且数值仍会被输出。

```ts twoslash
// @preserveConstEnums: true
// @showEmit
const enum Album {
  JimmyEatWorldFutures = 1,
  TubRingZooHypothesis = 2,
  DogFashionDiscoAdultery = 3,
}

const selectedAlbum = Album.JimmyEatWorldFutures
if (selectedAlbum === Album.JimmyEatWorldFutures) {
  console.log('That is a great choice.')
}
```

这实质上使此类 `const enum` 仅作为源码特性存在，在运行时不留痕迹。
