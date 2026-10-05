---
display: 'Suppress Implicit Any Index Errors'
oneline: '在对缺少索引签名的对象进行索引时，抑制 [`noImplicitAny`](#noImplicitAny) 错误。'
---

开启 `suppressImplicitAnyIndexErrors` 会抑制对对象进行索引时报告的隐式 any 错误，如下例所示：

```ts twoslash
// @noImplicitAny: true
// @suppressImplicitAnyIndexErrors: false
// @strict: true
// @errors: 7053
const obj = { x: 10 }
console.log(obj['foo'])
```

使用 `suppressImplicitAnyIndexErrors` 是一种相当激进的做法。建议改用 `@ts-ignore` 注释：

```ts twoslash
// @noImplicitAny: true
// @strict: true
const obj = { x: 10 }
// @ts-ignore
console.log(obj['foo'])
```
