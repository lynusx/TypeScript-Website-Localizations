---
display: 'Strict Null Checks'
oneline: '在进行类型检查时考虑 `null` 和 `undefined`。'
---

当 `strictNullChecks` 为 `false` 时，语言实际上会忽略 `null` 和 `undefined`。
这可能会导致运行时出现意料之外的错误。

当 `strictNullChecks` 为 `true` 时，`null` 和 `undefined` 拥有各自独立的类型。如果尝试在需要具体值的地方使用它们，将会收到类型错误。

例如在以下 TypeScript 代码中，`users.find` 并不能保证一定能找到用户，但你可以像必定能找到一样编写代码：

```ts twoslash
// @strictNullChecks: false
// @target: ES2015
declare const loggedInUsername: string

const users = [
  { name: 'Oby', age: 12 },
  { name: 'Heera', age: 32 },
]

const loggedInUser = users.find((u) => u.name === loggedInUsername)
console.log(loggedInUser.age)
```

将 `strictNullChecks` 设置为 `true` 时，若未在使用 `loggedInUser` 之前确保其存在，就会报错。

```ts twoslash
// @errors: 2339 2532 18048
// @target: ES2020
// @strictNullChecks
declare const loggedInUsername: string

const users = [
  { name: 'Oby', age: 12 },
  { name: 'Heera', age: 32 },
]

const loggedInUser = users.find((u) => u.name === loggedInUsername)
console.log(loggedInUser.age)
```

第二个示例之所以失败，是因为数组的 `find` 函数大致可以简化如下：

```ts
// 当 strictNullChecks 为 true 时
type Array = {
  find(predicate: (value: any, index: number) => boolean): S | undefined
}

// 当 strictNullChecks 为 false 时，undefined 会从类型系统中移除，
// 允许你编写假定总能找到结果的代码
type Array = {
  find(predicate: (value: any, index: number) => boolean): S
}
```
