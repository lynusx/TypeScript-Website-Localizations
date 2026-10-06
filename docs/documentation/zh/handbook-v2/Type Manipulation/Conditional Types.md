---
title: 条件类型
layout: docs
permalink: /zh/docs/handbook/2/conditional-types.html
oneline: '在类型系统中创建类似于 if 语句的类型。'
---

在大多数实用的程序中，我们必须根据输入做出决策。
JavaScript 程序也是如此，但由于值可以被轻松内省，这些决策也是基于输入的类型做出的。
_条件类型_（conditional types）有助于描述输入与输出类型之间的关系。

```ts twoslash
interface Animal {
  live(): void
}
interface Dog extends Animal {
  woof(): void
}

type Example1 = Dog extends Animal ? number : string
//   ^?

type Example2 = RegExp extends Animal ? number : string
//   ^?
```

条件类型的形式看起来与 JavaScript 中的条件表达式（`condition ? trueExpression : falseExpression`）有些相似：

```ts twoslash
type SomeType = any
type OtherType = any
type TrueType = any
type FalseType = any
type Stuff =
  // ---cut---
  SomeType extends OtherType ? TrueType : FalseType
```

当 `extends` 左侧的类型可以赋值给右侧的类型时，你将获得第一个分支（即“true”分支）中的类型；否则将获得后一个分支（即“false”分支）中的类型。

从上面的示例来看，条件类型可能并不会立刻显得很有用 —— 我们可以自己判断 `Dog extends Animal` 是否成立，并选择 `number` 还是 `string`！
但条件类型的真正威力在于将其与泛型结合使用。

例如，来看下面的 `createLabel` 函数：

```ts twoslash
interface IdLabel {
  id: number /* 一些字段 */
}
interface NameLabel {
  name: string /* 其他字段 */
}

function createLabel(id: number): IdLabel
function createLabel(name: string): NameLabel
function createLabel(nameOrId: string | number): IdLabel | NameLabel
function createLabel(nameOrId: string | number): IdLabel | NameLabel {
  throw 'unimplemented'
}
```

这些针对 `createLabel` 的重载描述了一个根据输入类型做出选择的单个 JavaScript 函数。请注意以下几点：

1. 如果一个库在其整个 API 中必须反复做出相同的选择，这会变得非常繁琐。
2. 我们必须创建三个重载：针对我们*确定*类型的每种情况各一个（一个用于 `string`，一个用于 `number`），以及针对最通用情况的一个（接收 `string | number`）。对于 `createLabel` 可以处理的每种新类型，重载的数量都会呈指数级增长。

相反，我们可以在条件类型中对该逻辑进行编码：

```ts twoslash
interface IdLabel {
  id: number /* 一些字段 */
}
interface NameLabel {
  name: string /* 其他字段 */
}
// ---cut---
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel
```

然后，我们可以使用该条件类型将重载简化为没有重载的单一函数。

```ts twoslash
interface IdLabel {
  id: number /* 一些字段 */
}
interface NameLabel {
  name: string /* 其他字段 */
}
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel
// ---cut---
function createLabel<T extends number | string>(idOrName: T): NameOrId<T> {
  throw 'unimplemented'
}

let a = createLabel('typescript')
//  ^?

let b = createLabel(2.8)
//  ^?

let c = createLabel(Math.random() ? 'hello' : 42)
//  ^?
```

### 条件类型约束

通常，条件类型中的检查会为我们提供一些新信息。
正如使用类型守卫进行类型收窄可以为我们提供更具体的类型一样，条件类型的 true 分支会根据我们所检查的类型进一步约束泛型。

例如，来看以下代码：

```ts twoslash
// @errors: 2536
type MessageOf<T> = T['message']
```

在这个示例中，TypeScript 会报错，因为尚不确定 `T` 是否具有名为 `message` 的属性。
我们可以对 `T` 进行约束，这样 TypeScript 就不会再报错了：

```ts twoslash
type MessageOf<T extends { message: unknown }> = T['message']

interface Email {
  message: string
}

type EmailMessageContents = MessageOf<Email>
//   ^?
```

但是，如果我们希望 `MessageOf` 可以接收任何类型，并在 `message` 属性不可用时默认返回诸如 `never` 之类的类型呢？
我们可以通过将约束移出并引入条件类型来实现这一点：

```ts twoslash
type MessageOf<T> = T extends { message: unknown } ? T['message'] : never

interface Email {
  message: string
}

interface Dog {
  bark(): void
}

type EmailMessageContents = MessageOf<Email>
//   ^?

type DogMessageContents = MessageOf<Dog>
//   ^?
```

在 true 分支中，TypeScript 知道 `T` *一定*拥有 `message` 属性。

再举一个例子，我们还可以编写一个名为 `Flatten` 的类型，它将数组类型展平为其元素类型，而对其余类型保持原样：

```ts twoslash
type Flatten<T> = T extends any[] ? T[number] : T

// 提取出元素类型。
type Str = Flatten<string[]>
//   ^?

// 保持类型不变。
type Num = Flatten<number>
//   ^?
```

当为 `Flatten` 传入一个数组类型时，它使用 `number` 进行索引访问来获取 `string[]` 的元素类型。
否则，它只需返回传入的原始类型。

### 在条件类型内进行推断

刚才我们看到自己使用条件类型来应用约束并提取类型。
这最终成为了一种非常常见的操作，而条件类型让这一切变得更加简单。

条件类型为我们提供了一种通过 `infer` 关键字在 true 分支中从对比的类型推断类型的方法。
例如，我们可以在 `Flatten` 中直接推断元素类型，而不是通过索引访问类型“手动”提取它：

```ts twoslash
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type
```

在这里，我们使用 `infer` 关键字声明式地引入了一个名为 `Item` 的新泛型类型变量，而不是在 true 分支中指定如何检索 `Type` 的元素类型。
这使我们不必再去思考如何翻找和拆解我们感兴趣的类型的内部结构。

我们可以使用 `infer` 关键字编写一些实用的辅助类型别名。
例如，对于简单的情况，我们可以从函数类型中提取返回值类型：

```ts twoslash
type GetReturnType<Type> = Type extends (...args: never[]) => infer Return
  ? Return
  : never

type Num = GetReturnType<() => number>
//   ^?

type Str = GetReturnType<(x: string) => string>
//   ^?

type Bools = GetReturnType<(a: boolean, b: boolean) => boolean[]>
//   ^?
```

当从具有多个调用签名的类型（例如重载函数的类型）进行推断时，会根据*最后一个*签名进行推断（据推测，这是最宽松的兜底情况）。无法根据参数类型列表来执行重载解析。

```ts twoslash
declare function stringOrNum(x: string): number
declare function stringOrNum(x: number): string
declare function stringOrNum(x: string | number): string | number

type T1 = ReturnType<typeof stringOrNum>
//   ^?
```

## 分布式条件类型

当条件类型作用于泛型类型时，如果传入联合类型，它们就会变成*分布式*（distributive）的。
例如，看下面的代码：

```ts twoslash
type ToArray<Type> = Type extends any ? Type[] : never
```

如果我们将联合类型传入 `ToArray`，则条件类型将应用于该联合类型的每个成员。

```ts twoslash
type ToArray<Type> = Type extends any ? Type[] : never

type StrArrOrNumArr = ToArray<string | number>
//   ^?
```

这里发生的情况是 `ToArray` 分布在：

```ts twoslash
type StrArrOrNumArr =
  // ---cut---
  string | number
```

并映射联合类型的每个成员类型，实际上相当于：

```ts twoslash
type ToArray<Type> = Type extends any ? Type[] : never
type StrArrOrNumArr =
  // ---cut---
  ToArray<string> | ToArray<number>
```

最终得到：

```ts twoslash
type StrArrOrNumArr =
  // ---cut---
  string[] | number[]
```

通常情况下，分布式行为正是我们想要的效果。
要避免这种分布式行为，可以在 `extends` 关键字的两侧加上方括号。

```ts twoslash
type ToArrayNonDist<Type> = [Type] extends [any] ? Type[] : never

// 'ArrOfStrOrNum' 不再是联合类型。
type ArrOfStrOrNum = ToArrayNonDist<string | number>
//   ^?
```
