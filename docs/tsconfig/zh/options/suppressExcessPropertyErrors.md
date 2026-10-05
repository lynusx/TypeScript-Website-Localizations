---
display: 'Suppress Excess Property Errors'
oneline: '禁用在创建对象字面量时报告额外属性错误。'
---

该选项禁用对额外属性错误的报告，如下例所示：

```ts twoslash
// @errors: 2322
type Point = { x: number; y: number }
const p: Point = { x: 1, y: 3, m: 10 }
```

添加该标志是为了帮助开发者迁移到 [TypeScript 1.6](/docs/handbook/release-notes/typescript-1-6.html#stricter-object-literal-assignment-checks) 中对新对象字面量更严格的检查。

我们不建议在现代代码库中使用此标志；如果遇到个别需要抑制报错的情况，可以使用 `// @ts-ignore`。
