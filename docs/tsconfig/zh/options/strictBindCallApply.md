---
display: 'Strict Bind Call Apply'
oneline: '检查 `bind`、`call` 和 `apply` 方法的参数是否与原函数相匹配。'
---

设置后，TypeScript 会检查函数的内置方法 `call`、`bind` 和 `apply` 是否传入了与底层函数相匹配的正确参数：

```ts twoslash
// @strictBindCallApply: true
// @errors: 2345

// 当 strictBindCallApply 开启时
function fn(x: string) {
  return parseInt(x)
}

const n1 = fn.call(undefined, '10')

const n2 = fn.call(undefined, false)
```

否则，这些方法将接受任意参数并返回 `any`：

```ts twoslash
// @strictBindCallApply: false

// 当 strictBindCallApply 关闭时
function fn(x: string) {
  return parseInt(x)
}

// 注意：没有报错；返回类型为 'any'
const n = fn.call(undefined, false)
```
