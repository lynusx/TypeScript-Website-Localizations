---
title: 索引访问类型
layout: docs
permalink: /zh/docs/handbook/2/indexed-access-types.html
oneline: "使用 Type['a'] 语法访问类型的子集。"
---

我们可以使用 _索引访问类型_ 来查找另一个类型上的特定属性：

```ts twoslash
type Person = { age: number; name: string; alive: boolean }
type Age = Person['age']
//   ^?
```

用于索引的类型本身也是一个类型，所以我们完全可以使用联合、`keyof` 或其他类型：

```ts twoslash
type Person = { age: number; name: string; alive: boolean }
// ---cut---
type I1 = Person['age' | 'name']
//   ^?

type I2 = Person[keyof Person]
//   ^?

type AliveOrName = 'alive' | 'name'
type I3 = Person[AliveOrName]
//   ^?
```

如果你试索索引一个不存在的属性，甚至会看到错误：

```ts twoslash
// @errors: 2339
type Person = { age: number; name: string; alive: boolean }
// ---cut---
type I1 = Person['alve']
```

另一个用任意类型进行索引的例子，是使用 `number` 来获取数组元素的类型。我们可以将其与 `typeof` 结合，方便地捕获数组字面量的元素类型：

```ts twoslash
const MyArray = [
  { name: 'Alice', age: 15 },
  { name: 'Bob', age: 23 },
  { name: 'Eve', age: 38 },
]

type Person = (typeof MyArray)[number]
//   ^?
type Age = (typeof MyArray)[number]['age']
//   ^?
// Or
type Age2 = Person['age']
//   ^?
```

索引时只能使用类型，这意味着你不能使用 `const` 变量引用来进行索引：

```ts twoslash
// @errors: 2538 2749
type Person = { age: number; name: string; alive: boolean }
// ---cut---
const key = 'age'
type Age = Person[key]
```

不过，你可以使用类型别名来进行类似风格的重构：

```ts twoslash
type Person = { age: number; name: string; alive: boolean }
// ---cut---
type key = 'age'
type Age = Person[key]
```
