---
title: TypeScript 2.8
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-2-8.html
oneline: TypeScript 2.8 发布说明
---

## 条件类型

TypeScript 2.8 引入了**条件类型**，它能够表达非均匀的类型映射。
条件类型基于一个类型关系测试条件，从两个可能的类型中选择其一：

```ts
T extends U ? X : Y
```

上述类型的含义是：当 `T` 可赋值给 `U` 时，类型为 `X`，否则类型为 `Y`。

条件类型 `T extends U ? X : Y` 要么被**解析**为 `X` 或 `Y`，要么因条件依赖于一个或多个类型变量而被**推迟**。
是否解析或推迟，按如下规则确定：

- 首先，给定 `T` 和 `U` 的实例化版本 `T'` 和 `U'`（其中所有类型参数都被替换为 `any`）：若 `T'` 不可赋值给 `U'`，则条件类型被解析为 `Y`。直觉上，如果 `T` 最宽松的实例化版本都无法赋值给 `U` 最宽松的实例化版本，那么任何实例化版本都不行，直接解析为 `Y`。
- 接着，对于 `U` 中每个由 `infer` 声明（下文详述）引入的类型变量，通过从 `T` 到 `U` 的推断（使用与泛型函数类型推断相同的算法）收集候选类型集合。对于给定的 `infer` 类型变量 `V`：若存在从协变位置推断的候选类型，则 `V` 的推断类型为这些候选类型的联合；若存在从逆变位置推断的候选类型，则 `V` 的推断类型为这些候选类型的交叉；否则，`V` 的推断类型为 `never`。
- 然后，给定 `T` 的实例化版本 `T''`（其中所有 `infer` 类型变量被替换为上一步推断的类型）：若 `T''` 可**确定地赋值**给 `U`，则条件类型被解析为 `X`。"确定地可赋值"与常规可赋值关系相同，但不考虑类型变量约束。直觉上，当一个类型确定地可赋值给另一个类型时，我们知道对于这两种类型的**所有实例化**都成立。
- 否则，条件依赖于一个或多个类型变量，条件类型被推迟。

##### 示例

```ts
type TypeName<T> = T extends string
  ? 'string'
  : T extends number
    ? 'number'
    : T extends boolean
      ? 'boolean'
      : T extends undefined
        ? 'undefined'
        : T extends Function
          ? 'function'
          : 'object'

type T0 = TypeName<string> // "string"
type T1 = TypeName<'a'> // "string"
type T2 = TypeName<true> // "boolean"
type T3 = TypeName<() => void> // "function"
type T4 = TypeName<string[]> // "object"
```

## 分布式条件类型

被检查类型为裸类型参数的条件类型称为**分布式条件类型**。
分布式条件类型在实例化时会自动分布到联合类型的各成员上。
例如，将类型参数 `A | B | C` 代入 `T extends U ? X : Y` 中的 `T`，会被解析为 `(A extends U ? X : Y) | (B extends U ? X : Y) | (C extends U ? X : Y)`。

##### 示例

```ts
type T10 = TypeName<string | (() => void)> // "string" | "function"
type T12 = TypeName<string | string[] | undefined> // "string" | "object" | "undefined"
type T11 = TypeName<string[] | number[]> // "object"
```

在分布式条件类型 `T extends U ? X : Y` 的实例化中，条件类型内部对 `T` 的引用会被解析为联合类型的各个成员（即 `T` 指向条件类型分布到联合类型**之后**的各个成员）。
此外，`X` 内部对 `T` 的引用还附带了额外的类型参数约束 `U`（即在 `X` 中，`T` 被视为可赋值给 `U`）。

##### 示例

```ts
type BoxedValue<T> = { value: T }
type BoxedArray<T> = { array: T[] }
type Boxed<T> = T extends any[] ? BoxedArray<T[number]> : BoxedValue<T>

type T20 = Boxed<string> // BoxedValue<string>;
type T21 = Boxed<number[]> // BoxedArray<number>;
type T22 = Boxed<string | number[]> // BoxedValue<string> | BoxedArray<number>;
```

注意，在 `Boxed<T>` 的 true 分支中，`T` 附带了额外约束 `any[]`，因此可以用 `T[number]` 引用数组的元素类型。另外注意最后一个示例中条件类型是如何分布到联合类型上的。

条件类型的分布式特性可以方便地用于**过滤**联合类型：

```ts
type Diff<T, U> = T extends U ? never : T // 从 T 中移除可赋值给 U 的类型
type Filter<T, U> = T extends U ? T : never // 从 T 中移除不可赋值给 U 的类型

type T30 = Diff<'a' | 'b' | 'c' | 'd', 'a' | 'c' | 'f'> // "b" | "d"
type T31 = Filter<'a' | 'b' | 'c' | 'd', 'a' | 'c' | 'f'> // "a" | "c"
type T32 = Diff<string | number | (() => void), Function> // string | number
type T33 = Filter<string | number | (() => void), Function> // () => void

type NonNullable<T> = Diff<T, null | undefined> // 从 T 中移除 null 和 undefined

type T34 = NonNullable<string | number | undefined> // string | number
type T35 = NonNullable<string | string[] | null | undefined> // string | string[]

function f1<T>(x: T, y: NonNullable<T>) {
  x = y // 正常
  y = x // 错误
}

function f2<T extends string | undefined>(x: T, y: NonNullable<T>) {
  x = y // 正常
  y = x // 错误
  let s1: string = x // 错误
  let s2: string = y // 正常
}
```

条件类型与映射类型结合使用时特别有用：

```ts
type FunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? K : never
}[keyof T]
type FunctionProperties<T> = Pick<T, FunctionPropertyNames<T>>

type NonFunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? never : K
}[keyof T]
type NonFunctionProperties<T> = Pick<T, NonFunctionPropertyNames<T>>

interface Part {
  id: number
  name: string
  subparts: Part[]
  updatePart(newName: string): void
}

type T40 = FunctionPropertyNames<Part> // "updatePart"
type T41 = NonFunctionPropertyNames<Part> // "id" | "name" | "subparts"
type T42 = FunctionProperties<Part> // { updatePart(newName: string): void }
type T43 = NonFunctionProperties<Part> // { id: number, name: string, subparts: Part[] }
```

与联合类型和交叉类型类似，条件类型不允许递归引用自身。
例如，以下代码会报错：

##### 示例

```ts
type ElementType<T> = T extends any[] ? ElementType<T[number]> : T // 错误
```

## 条件类型中的类型推断

在条件类型的 `extends` 子句中，现在可以使用 `infer` 声明来引入一个待推断的类型变量。
这些被推断的类型变量可以在条件类型的 true 分支中被引用。
同一个类型变量可以在多个位置使用 `infer`。

例如，以下代码提取了函数类型的返回类型：

```ts
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any
```

条件类型可以嵌套，形成按顺序匹配的模式序列：

```ts
type Unpacked<T> = T extends (infer U)[]
  ? U
  : T extends (...args: any[]) => infer U
    ? U
    : T extends Promise<infer U>
      ? U
      : T

type T0 = Unpacked<string> // string
type T1 = Unpacked<string[]> // string
type T2 = Unpacked<() => string> // string
type T3 = Unpacked<Promise<string>> // string
type T4 = Unpacked<Promise<string>[]> // Promise<string>
type T5 = Unpacked<Unpacked<Promise<string>[]>> // string
```

以下示例展示了在协变位置上对同一类型变量存在多个候选类型时，如何推断出联合类型：

```ts
type Foo<T> = T extends { a: infer U; b: infer U } ? U : never
type T10 = Foo<{ a: string; b: string }> // string
type T11 = Foo<{ a: string; b: number }> // string | number
```

类似地，在逆变位置上对同一类型变量存在多个候选类型时，会推断出交叉类型：

```ts
type Bar<T> = T extends { a: (x: infer U) => void; b: (x: infer U) => void }
  ? U
  : never
type T20 = Bar<{ a: (x: string) => void; b: (x: string) => void }> // string
type T21 = Bar<{ a: (x: string) => void; b: (x: number) => void }> // string & number
```

当从具有多个调用签名的类型（如重载函数的类型）进行推断时，推断基于**最后一个**签名（该签名通常是最宽泛的通用情况）。
无法基于参数类型列表执行重载解析。

```ts
declare function foo(x: string): number
declare function foo(x: number): string
declare function foo(x: string | number): string | number
type T30 = ReturnType<typeof foo> // string | number
```

`infer` 声明不能用于普通类型参数的约束子句中：

```ts
type ReturnType<T extends (...args: any[]) => infer R> = R // 错误，不支持
```

但是，可以通过在约束中去掉类型变量、改用条件类型来达到相同的效果：

```ts
type AnyFunction = (...args: any[]) => any
type ReturnType<T extends AnyFunction> = T extends (...args: any[]) => infer R
  ? R
  : any
```

## 预定义条件类型

TypeScript 2.8 在 `lib.d.ts` 中新增了几个预定义的条件类型：

- `Exclude<T, U>` —— 从 `T` 中排除可赋值给 `U` 的类型。
- `Extract<T, U>` —— 从 `T` 中提取可赋值给 `U` 的类型。
- `NonNullable<T>` —— 从 `T` 中排除 `null` 和 `undefined`。
- `ReturnType<T>` —— 获取函数类型的返回类型。
- `InstanceType<T>` —— 获取构造函数类型的实例类型。

##### 示例

```ts
type T00 = Exclude<'a' | 'b' | 'c' | 'd', 'a' | 'c' | 'f'> // "b" | "d"
type T01 = Extract<'a' | 'b' | 'c' | 'd', 'a' | 'c' | 'f'> // "a" | "c"

type T02 = Exclude<string | number | (() => void), Function> // string | number
type T03 = Extract<string | number | (() => void), Function> // () => void

type T04 = NonNullable<string | number | undefined> // string | number
type T05 = NonNullable<(() => string) | string[] | null | undefined> // (() => string) | string[]

function f1(s: string) {
  return { a: 1, b: s }
}

class C {
  x = 0
  y = 0
}

type T10 = ReturnType<() => string> // string
type T11 = ReturnType<(s: string) => void> // void
type T12 = ReturnType<<T>() => T> // {}
type T13 = ReturnType<<T extends U, U extends number[]>() => T> // number[]
type T14 = ReturnType<typeof f1> // { a: number, b: string }
type T15 = ReturnType<any> // any
type T16 = ReturnType<never> // any
type T17 = ReturnType<string> // 错误
type T18 = ReturnType<Function> // 错误

type T20 = InstanceType<typeof C> // C
type T21 = InstanceType<any> // any
type T22 = InstanceType<never> // any
type T23 = InstanceType<string> // 错误
type T24 = InstanceType<Function> // 错误
```

> 注意：`Exclude` 类型是[此处](https://github.com/Microsoft/TypeScript/issues/12215#issuecomment-307871458)建议的 `Diff` 类型的官方实现。我们之所以采用 `Exclude` 这个名称，是为了避免破坏现有已定义 `Diff` 的代码，而且我们认为该名称更能准确表达该类型的语义。我们并未包含 `Omit<T, K>` 类型，因为它可以很简明地编写为 `Pick<T, Exclude<keyof T, K>>`。

## 改进对映射类型修饰符的控制

映射类型支持为映射属性添加 `readonly` 或 `?` 修饰符，但此前不支持**移除**修饰符。
这在[同态映射类型](https://github.com/Microsoft/TypeScript/pull/12563)中尤为重要，因为同态映射类型默认会保留底层类型的修饰符。

TypeScript 2.8 增加了映射类型添加或移除特定修饰符的能力。
具体而言，映射类型中的 `readonly` 或 `?` 属性修饰符现在可以加上 `+` 或 `-` 前缀，分别表示添加或移除该修饰符。

#### 示例

```ts
type MutableRequired<T> = { -readonly [P in keyof T]-?: T[P] } // 移除 readonly 和 ?
type ReadonlyPartial<T> = { +readonly [P in keyof T]+?: T[P] } // 添加 readonly 和 ?
```

没有 `+` 或 `-` 前缀的修饰符等同于带 `+` 前缀的修饰符。因此，上面的 `ReadonlyPartial<T>` 类型等同于：

```ts
type ReadonlyPartial<T> = { readonly [P in keyof T]?: T[P] } // 添加 readonly 和 ?
```

利用这一能力，`lib.d.ts` 新增了 `Required<T>` 类型。
该类型会去掉 `T` 所有属性上的 `?` 修饰符，从而使所有属性变为必需。

##### 示例

```ts
type Required<T> = { [P in keyof T]-?: T[P] }
```

注意，在 [`strictNullChecks`](/tsconfig#strictNullChecks) 模式下，当同态映射类型移除底层类型某个属性上的 `?` 修饰符时，该属性类型中的 `undefined` 也会被一并移除：

##### 示例

```ts
type Foo = { a?: string } // 等同于 { a?: string | undefined }
type Bar = Required<Foo> // 等同于 { a: string }
```

## 改进 `keyof` 对交叉类型的处理

TypeScript 2.8 中，对交叉类型应用 `keyof` 会被转换为对各交叉成员分别应用 `keyof` 的联合。
换言之，`keyof (A & B)` 形式的类型会被转换为 `keyof A | keyof B`。
这一改变解决了 `keyof` 表达式推断结果不一致的问题。

##### 示例

```ts
type A = { a: string }
type B = { b: string }

type T1 = keyof (A & B) // "a" | "b"
type T2<T> = keyof (T & B) // keyof T | "b"
type T3<U> = keyof (A & U) // "a" | keyof U
type T4<T, U> = keyof (T & U) // keyof T | keyof U
type T5 = T2<A> // "a" | "b"
type T6 = T3<B> // "a" | "b"
type T7 = T4<A, B> // "a" | "b"
```

## 对 `.js` 文件中命名空间模式的更好支持

TypeScript 2.8 新增了对更多 `.js` 文件命名空间模式的识别支持。
顶层的空对象字面量声明，与函数和类一样，现在会被识别为 JavaScript 中的命名空间声明。

```js
var ns = {} // 被识别为命名空间 `ns` 的声明
ns.constant = 1 // 被识别为变量 `constant` 的声明
```

顶层的赋值语句应表现相同的行为，也就是说，不强制要求使用 `var` 或 `const` 声明：

```js
app = {} // 不需要是 `var app = {}`
app.C = class {}
app.f = function () {}
app.prop = 1
```

## IIFE 作为命名空间声明

返回函数、类或空对象字面量的 IIFE 也会被识别为命名空间：

```js
var C = (function () {
  function C(n) {
    this.p = n
  }
  return C
})()
C.staticProperty = 1
```

## 默认声明

"默认声明"允许在逻辑或运算符左侧引用被声明名称的初始化器：

```js
my = window.my || {}
my.app = my.app || {}
```

## 原型赋值

你可以将对象字面量直接赋值给 prototype 属性，单个原型赋值同样有效：

```ts
var C = function (p) {
  this.p = p
}
C.prototype = {
  m() {
    console.log(this.p)
  },
}
C.prototype.q = function (r) {
  return this.p === r
}
```

## 嵌套与合并声明

嵌套现在支持任意深度，并能在多文件间正确合并。此前这两点都不支持。

```js
var app = window.app || {}
app.C = class {}
```

## 每个文件的 JSX 工厂

TypeScript 2.8 支持通过 `@jsx dom` 编译指示在每个文件中单独配置 JSX 工厂名称。
可以通过 [`jsxFactory`](/tsconfig#jsxFactory) 编译选项（默认为 `React.createElement`）为整个编译配置 JSX 工厂。TypeScript 2.8 允许在文件开头添加注释，逐文件覆盖此配置。

##### 示例

```ts
/** @jsx dom */
import { dom } from "./renderer";
<h></h>;
```

生成：

```js
var renderer_1 = require('./renderer')
renderer_1.dom('h', null)
```

## 局部作用域的 JSX 命名空间

JSX 类型检查由 JSX 命名空间中的定义驱动，例如 `JSX.Element` 表示 JSX 元素的类型，`JSX.IntrinsicElements` 表示内置元素。
在 TypeScript 2.8 之前，`JSX` 命名空间需要位于全局命名空间中，因此一个项目中只能定义一个。
从 TypeScript 2.8 开始，`JSX` 命名空间将在 `jsxNamespace`（如 `React`）下查找，从而允许一次编译中存在多个 JSX 工厂。
为保持向后兼容性，如果工厂函数上未定义 JSX 命名空间，会回退到全局 `JSX` 命名空间。
结合每个文件的 `@jsx` 编译指示，每个文件可以使用不同的 JSX 工厂。

## 新的 `--emitDeclarationOnly`

[`emitDeclarationOnly`](/tsconfig#emitDeclarationOnly) 允许**仅**生成声明文件，使用此标志时会跳过 `.js`/`.jsx` 文件的生成。当 `.js` 文件的生成由 Babel 等其他转译器处理时，该标志非常有用。
