---
display: 'Allow Unreachable Code'
oneline: '禁用无法访问的代码的错误报告。'
---

可选值：

- `undefined`（默认值）：向编辑器提供建议性质的警告
- `true`：忽略无法访问的代码
- `false`：对无法访问的代码抛出编译器错误

这些警告仅针对因 JavaScript 语法使用而明确可判定为无法访问的代码，例如：

```ts
function fn(n: number) {
  if (n > 5) {
    return true
  } else {
    return false
  }
  return true
}
```

设置 `"allowUnreachableCode": false` 时：

```ts twoslash
// @errors: 7027
// @allowUnreachableCode: false
function fn(n: number) {
  if (n > 5) {
    return true
  } else {
    return false
  }
  return true
}
```

这不会影响那些因类型分析而*看起来*无法访问的代码所产生的错误。
