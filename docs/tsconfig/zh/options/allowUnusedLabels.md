---
display: 'Allow Unused Labels'
oneline: '禁用未使用的标签的错误报告。'
---

取值说明：

- `undefined`（默认值）：向编辑器提供警告建议
- `true`：忽略未使用的标签
- `false`：对未使用的标签发出编译器错误

标签在 JavaScript 中非常少见，通常表明原本想编写对象字面量：

```ts twoslash
// @errors: 7028
// @allowUnusedLabels: false
function verifyAge(age: number) {
  // 遗漏了 'return' 语句
  if (age > 18) {
    verified: true
  }
}
```
