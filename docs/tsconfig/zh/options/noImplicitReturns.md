---
display: 'No Implicit Returns'
oneline: '对函数中未显式 return 的代码分支报错。'
---

启用后，TypeScript 将检查函数中的所有代码路径，以确保它们都返回了值。

```ts twoslash
// @errors: 2366 2322
function lookupHeadphonesManufacturer(color: 'blue' | 'black'): string {
  if (color === 'blue') {
    return 'beats'
  } else {
    ;('bose')
  }
}
```
