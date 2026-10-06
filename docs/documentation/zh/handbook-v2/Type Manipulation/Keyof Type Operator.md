---
title: keyof 类型运算符
layout: docs
permalink: /zh/docs/handbook/2/keyof-types.html
oneline: '在类型上下文中使用 keyof 运算符。'
---

## `keyof` 类型运算符

`keyof` 运算符接收一个对象类型，并生成其键名组成的字符串或数字字面量的联合类型。
下面的类型 `P` 与 `type P = "x" | "y"` 是相同的类型：

```ts twoslash
type Point = { x: number; y: number }
type P = keyof Point
//   ^?
```

如果该类型包含 `string` 或 `number` 索引签名，`keyof` 则会返回这些类型：

```ts twoslash
type Arrayish = { [n: number]: unknown }
type A = keyof Arrayish
//   ^?

type Mapish = { [k: string]: boolean }
type M = keyof Mapish
//   ^?
```

注意在此示例中，`M` 的类型是 `string | number` —— 这是因为 JavaScript 对象的键总是会被强制转换为字符串，因此 `obj[0]` 始终等同于 `obj["0"]`。

`keyof` 类型在与映射类型结合使用时尤其有用，我们稍后会对此进行更深入的了解。
