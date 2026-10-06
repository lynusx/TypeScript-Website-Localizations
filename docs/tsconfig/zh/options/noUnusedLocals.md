---
display: 'No Unused Locals'
oneline: '当局部变量未被读取时启用报错。'
---

对未使用的局部变量报错。

```ts twoslash
// @noUnusedLocals
// @errors: 6133
const createKeyboard = (modelID: number) => {
  const defaultModelID = 23
  return { type: 'keyboard', modelID }
}
```
