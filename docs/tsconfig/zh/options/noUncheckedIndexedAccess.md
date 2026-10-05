---
display: 'No Unchecked Indexed Access'
oneline: '当使用索引访问时，将 `undefined` 添加到类型中。'
---

TypeScript 提供了一种通过索引签名（index signatures）来描述键未知但值类型已知的对象的方法。

```ts twoslash
interface EnvironmentVars {
  NAME: string
  OS: string

  // Unknown properties are covered by this index signature.
  [propName: string]: string
}

declare const env: EnvironmentVars

// Declared as existing
const sysName = env.NAME
const os = env.OS
//    ^?

// Not declared, but because of the index
// signature, then it is considered a string
const nodeEnv = env.NODE_ENV
//    ^?
```

启用 `noUncheckedIndexedAccess` 会在类型中为任何未声明的字段添加 `undefined`。

```ts twoslash
interface EnvironmentVars {
  NAME: string
  OS: string

  // Unknown properties are covered by this index signature.
  [propName: string]: string
}
// @noUncheckedIndexedAccess
// ---cut---
declare const env: EnvironmentVars

// Declared as existing
const sysName = env.NAME
const os = env.OS
//    ^?

// Not declared, but because of the index
// signature, then it is considered a string
const nodeEnv = env.NODE_ENV
//    ^?
```
