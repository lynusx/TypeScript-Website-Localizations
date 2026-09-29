---
title: 映射类型
layout: docs
permalink: /docs/handbook/2/mapped-types.html
oneline: '通过复用现有类型来生成类型。'
---

为了避免代码重复，有时我们需要基于现有类型来创建新类型。

_映射类型_ 建立在索引签名语法的基础之上。索引签名通常用于声明那些未提前声明的属性的类型：

```ts twoslash
type Horse = {}
// ---cut---
type OnlyBoolsAndHorses = {
  [key: string]: boolean | Horse
}

const conforms: OnlyBoolsAndHorses = {
  del: true,
  rodney: false,
}
```

映射类型属于泛型，它利用 `PropertyKey` 的联合类型（通常[通过 `keyof` 获得](/docs/handbook/2/indexed-access-types.html)）来遍历键，进而构造出新类型：

```ts twoslash
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean
}
```

在这个例子中，`OptionsFlags` 提取了 `Type` 的所有属性，并将它们的类型全部改为 boolean。

```ts twoslash
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean
}
// ---cut---
type Features = {
  darkMode: () => void
  newUserProfile: () => void
}

type FeatureOptions = OptionsFlags<Features>
//   ^?
```

### 映射修饰符

在映射过程中，还可以应用两个额外的修饰符：`readonly` 和 `?`，它们分别用于控制属性的可变性和可选性。

你可以通过添加 `-` 或 `+` 前缀来移除或添加这些修饰符。如果不加前缀，则默认使用 `+`。

```ts twoslash
// 从类型的属性中移除 'readonly' 属性
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property]
}

type LockedAccount = {
  readonly id: string
  readonly name: string
}

type UnlockedAccount = CreateMutable<LockedAccount>
//   ^?
```

```ts twoslash
// 从类型的属性中移除 'optional' 属性（可选属性）
type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property]
}

type MaybeUser = {
  id: string
  name?: string
  age?: number
}

type User = Concrete<MaybeUser>
//   ^?
```

## 通过 `as` 重新映射键名

自 TypeScript 4.1 起，你可以通过映射类型中的 `as` 子句来重新映射键名：

```ts
type MappedTypeWithNewProperties<Type> = {
  [Properties in keyof Type as NewKeyType]: Type[Properties]
}
```

借助 [模板字面量类型](/docs/handbook/2/template-literal-types.html) 等特性，可以基于已有的属性名创建新的属性名：

```ts twoslash
type Getters<Type> = {
  [Property in keyof Type as `get${Capitalize<string & Property>}`]: () => Type[Property]
}

interface Person {
  name: string
  age: number
  location: string
}

type LazyPerson = Getters<Person>
//   ^?
```

借助条件类型生成 `never`，你可以过滤掉不需要的键：

```ts twoslash
// Remove the 'kind' property
type RemoveKindField<Type> = {
  [Property in keyof Type as Exclude<Property, 'kind'>]: Type[Property]
}

interface Circle {
  kind: 'circle'
  radius: number
}

type KindlessCircle = RemoveKindField<Circle>
//   ^?
```

你可以遍历映射任意的联合类型，不仅仅是 `string | number | symbol` 的联合，还可以是任何类型的联合：

```ts twoslash
type EventConfig<Events extends { kind: string }> = {
  [E in Events as E['kind']]: (event: E) => void
}

type SquareEvent = { kind: 'square'; x: number; y: number }
type CircleEvent = { kind: 'circle'; radius: number }

type Config = EventConfig<SquareEvent | CircleEvent>
//   ^?
```

### 深入探索

映射类型与本类型操作章节中的其他特性搭配使用效果极佳。例如，下面是一个 [使用条件类型的映射类型](/docs/handbook/2/conditional-types.html) 示例，它会根据对象是否将 `pii` 属性设置为字面量 `true` 来返回 `true` 或 `false`：

```ts twoslash
type ExtractPII<Type> = {
  [Property in keyof Type]: Type[Property] extends { pii: true } ? true : false
}

type DBFields = {
  id: { format: 'incrementing' }
  name: { type: string; pii: true }
}

type ObjectsNeedingGDPRDeletion = ExtractPII<DBFields>
//   ^?
```
