---
display: 'No Unused Parameters'
oneline: '当函数参数未被读取时报错。'
---

在函数中存在未使用的参数时报告错误。

```ts twoslash
// @noUnusedParameters
// @errors: 6133
const createDefaultKeyboard = (modelID: number) => {
  const defaultModelID = 23
  return { type: 'keyboard', modelID: defaultModelID }
}
```

名称以下划线（`_`）开头的参数声明可以免除未使用参数的检查。例如：

```ts twoslash
// @noUnusedParameters
const createDefaultKeyboard = (_modelID: number) => {
  return { type: 'keyboard' }
}
```
