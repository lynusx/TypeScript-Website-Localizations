---
display: 'No Fallthrough Cases In Switch'
oneline: '对 switch 语句中贯穿（fallthrough）的 case 分支启用错误报告。'
---

报告 switch 语句中 case 分支贯穿的错误。
确保 switch 语句中任何非空的 case 分支都包含 `break`、`return` 或 `throw`。
这意味着你不会意外引入 case 贯穿的 bug。

```ts twoslash
// @noFallthroughCasesInSwitch
// @errors: 7029
const a: number = 6

switch (a) {
  case 0:
    console.log('even')
  case 1:
    console.log('odd')
    break
}
```
