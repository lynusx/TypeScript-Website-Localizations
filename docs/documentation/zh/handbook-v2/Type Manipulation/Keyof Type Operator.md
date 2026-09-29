---
title: keyof 类型操作符
layout: docs
permalink: /zh/docs/handbook/2/keyof-types.html
oneline: '在类型上下文中使用 keyof 操作符。'
---

## `keyof` 类型操作符

`keyof` 操作符接受一个对象类型，并生成其键名的字符串或数字字面量联合类型。以下类型 `P` 与 `type P = "x" | "y"` 是同一类型：

```ts twoslash
type Point = { x: number; y: number }
type P = keyof Point
//   ^?
```

如果该类型具有 `string` 或 `number` 索引签名，`keyof` 将返回这些类型：

```ts twoslash
type Arrayish = { [n: number]: unknown }
type A = keyof Arrayish
//   ^?

type Mapish = { [k: string]: boolean }
type M = keyof Mapish
//   ^?
```

注意，在这个例子中，`M` 的类型是 `string | number` —— 这是因为 JavaScript 对象的键总是会被强制转换为字符串，因此 `obj[0]` 始终等同于 `obj["0"]`。

`keyof` 类型在与映射类型结合使用时会非常有用，我们将在后续章节详细探讨映射类型。
