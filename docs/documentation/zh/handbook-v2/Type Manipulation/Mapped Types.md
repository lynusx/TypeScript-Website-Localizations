---
title: 映射类型
layout: docs
permalink: /zh/docs/handbook/2/mapped-types.html
oneline: '通过复用现有类型生成新类型。'
---

当你不想重复编写重复代码时，有时一个类型需要基于另一个类型来定义。

映射类型建立在索引签名语法之上，而索引签名用于声明尚未提前声明的属性类型：

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

映射类型是一种泛型类型，它使用 `PropertyKey` 的联合（通常[通过 `keyof`](/docs/handbook/2/indexed-access-types.html) 创建）来遍历键以创建新类型：

```ts twoslash
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean
}
```

在这个示例中，`OptionsFlags` 会获取类型 `Type` 中的所有属性，并将其值类型都转换为布尔值。

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

在映射过程中，还可以应用两个额外的修饰符：`readonly` 和 `?`，它们分别影响属性的可变性与可选性。

你可以通过添加 `-` 或 `+` 前缀来移除或添加这些修饰符。如果不添加前缀，则默认为 `+`。

```ts twoslash
// Removes 'readonly' attributes from a type's properties
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
// Removes 'optional' attributes from a type's properties
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

## 通过 `as` 进行键名重映射

在 TypeScript 4.1 及更高版本中，你可以通过映射类型中的 `as` 子句来重新映射键名：

```ts
type MappedTypeWithNewProperties<Type> = {
  [Properties in keyof Type as NewKeyType]: Type[Properties]
}
```

你可以借助[模板字面量类型](/docs/handbook/2/template-literal-types.html)等特性，基于已有的属性名创建新的属性名：

```ts twoslash
type Getters<Type> = {
  [
    Property in keyof Type as `get${Capitalize<string & Property>}`
  ]: () => Type[Property]
}

interface Person {
  name: string
  age: number
  location: string
}

type LazyPerson = Getters<Person>
//   ^?
```

你可以通过条件类型生成 `never` 来过滤掉特定键：

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

你不仅可以映射 `string | number | symbol` 的联合类型，还可以对任意类型的联合进行映射：

```ts twoslash
type EventConfig<Events extends { kind: string }> = {
  [E in Events as E['kind']]: (event: E) => void
}

type SquareEvent = { kind: 'square'; x: number; y: number }
type CircleEvent = { kind: 'circle'; radius: number }

type Config = EventConfig<SquareEvent | CircleEvent>
//   ^?
```

### 进阶探索

映射类型可以与本章类型操作中的其他特性完美配合。例如，下面是一个[使用条件类型的映射类型](/docs/handbook/2/conditional-types.html)，它根据对象的 `pii` 属性是否设置为字面量 `true`，来返回 `true` 或 `false`：

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
