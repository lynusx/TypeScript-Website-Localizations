---
display: 'No Strict Generic Checks'
oneline: '禁用函数类型中泛型签名的严格检查。'
---

TypeScript 在比较两个泛型函数时会统一（unify）类型参数。

```ts twoslash
// @errors: 2322

type A = <T, U>(x: T, y: U) => [T, U]
type B = <S>(x: S, y: S) => [S, S]

function f(a: A, b: B) {
  b = a // Ok
  a = b // Error
}
```

可以使用此标志来移除该检查。
