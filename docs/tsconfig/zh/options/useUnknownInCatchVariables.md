---
display: 'Use Unknown In Catch Variables'
oneline: '将 catch 子句中的变量默认类型设为 `unknown` 而非 `any`。'
---

TypeScript 4.0 增加了将 catch 子句中的变量类型从 `any` 更改为 `unknown` 的支持，允许编写如下代码：

```ts twoslash
// @useUnknownInCatchVariables
try {
  // ...
} catch (err: unknown) {
  // We have to verify err is an
  // error before using it as one.
  if (err instanceof Error) {
    console.log(err.message)
  }
}
```

这种模式确保了错误处理代码更加完善，因为你无法提前保证抛出的对象就一定（_is_）是 Error 的子类。启用 `useUnknownInCatchVariables` 标志后，你不再需要显式编写附加语法（`: unknown`），也不需要通过 linter 规则来强制执行此行为。
