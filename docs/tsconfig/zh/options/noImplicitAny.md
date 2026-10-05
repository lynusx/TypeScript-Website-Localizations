---
display: 'No Implicit Any'
oneline: '为具有隐式 `any` 类型的表达式和声明启用错误报告。'
---

在某些缺少类型注解的情况下，如果 TypeScript 无法推断变量的类型，它将回退为 `any` 类型。

这可能会导致遗漏某些错误，例如：

```ts twoslash
// @noImplicitAny: false
function fn(s) {
  // No error?
  console.log(s.subtr(3))
}
fn(42)
```

启用 `noImplicitAny` 后，每当 TypeScript 准备推断为 `any` 时，都会发出错误提示：

```ts twoslash
// @errors: 7006
function fn(s) {
  console.log(s.subtr(3))
}
```
