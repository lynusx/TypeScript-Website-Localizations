---
display: 'Strict Property Initialization'
oneline: '检查类中已声明但在构造函数中未明确初始化的属性。'
---

设置为 true 时，如果类的属性已被声明但未在构造函数中显式赋值，TypeScript 将会报错。

```ts twoslash
// @errors: 2564
class UserAccount {
  name: string
  accountType = 'user'

  email: string
  address: string | undefined

  constructor(name: string) {
    this.name = name
    // 注意未对 this.email 进行赋值
  }
}
```

在上述情况下：

- `this.name` 经过了显式赋值。
- `this.accountType` 具有默认初始值。
- `this.email` 未被赋值，因此会报错。
- `this.address` 被声明为可能为 `undefined`，这意味着它不必被赋值。
