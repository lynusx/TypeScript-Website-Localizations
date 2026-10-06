---
title: TypeScript 3.0
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-3-0.html
oneline: TypeScript 3.0 发布说明
---

## 项目引用

TypeScript 3.0 引入了全新的“项目引用”（Project References）概念。项目引用允许 TypeScript 项目依赖于其他 TypeScript 项目——具体而言，允许 `tsconfig.json` 文件引用其他的 `tsconfig.json` 文件。指定这些依赖关系可以让把代码拆分为更小的项目变得更加轻松，因为它使 TypeScript（以及相关周边工具）能够理解构建顺序和输出结构。

TypeScript 3.0 还为 tsc 引入了一种全新模式——`--build` 标志，它与项目引用紧密配合，从而实现更快的 TypeScript 构建。

更多文档请参阅[项目引用手册页面](/docs/handbook/project-references.html)。

## rest 参数与展开表达式中的元组

TypeScript 3.0 新增了多项将函数参数列表作为元组类型进行操作的新特性。
TypeScript 3.0 新增了对以下特性的支持：

- [将带有元组类型的 rest 参数展开为离散参数。](#rest-parameters-with-tuple-types)
- [将带有元组类型的展开表达式展开为离散实参。](#spread-expressions-with-tuple-types)
- [泛型 rest 参数及对应的元组类型推断。](#generic-rest-parameters)
- [元组类型中的可选元素。](#optional-elements-in-tuple-types)
- [元组类型中的 rest 元素。](#rest-elements-in-tuple-types)

借助这些特性，许多用于转换函数及其参数列表的高阶函数现在都可以实现强类型化。

## 带有元组类型的 rest 参数

当 rest 参数具有元组类型时，该元组类型会被展开为一系列离散参数。
例如，以下两个声明是等价的：

```ts
declare function foo(...args: [number, string, boolean]): void
```

```ts
declare function foo(args_0: number, args_1: string, args_2: boolean): void
```

## 带有元组类型的展开表达式

当函数调用将元组类型的展开表达式作为最后一个实参传入时，该展开表达式对应于由该元组元素类型组成的离散实参序列。

因此，以下调用是等价的：

```ts
const args: [number, string, boolean] = [42, 'hello', true]
foo(42, 'hello', true)
foo(args[0], args[1], args[2])
foo(...args)
```

## 泛型 rest 参数

rest 参数允许使用约束为数组类型的泛型类型，类型推断可以为此类泛型 rest 参数推断出元组类型。这使得捕获并展开部分参数列表的高阶操作成为可能：

##### 示例

```ts
declare function bind<T, U extends any[], V>(
  f: (x: T, ...args: U) => V,
  x: T,
): (...args: U) => V

declare function f3(x: number, y: string, z: boolean): void

const f2 = bind(f3, 42) // (y: string, z: boolean) => void
const f1 = bind(f2, 'hello') // (z: boolean) => void
const f0 = bind(f1, true) // () => void

f3(42, 'hello', true)
f2('hello', true)
f1(true)
f0()
```

在上述 `f2` 的声明中，类型推断分别将 `T`、`U` 和 `V` 推断为类型 `number`、`[string, boolean]` 和 `void`。

请注意，当从参数序列推断出元组类型并在随后展开为参数列表时（如 `U` 的情况），展开时会使用原始参数名称（不过，这些名称没有语义意义，在其他任何地方也是不可观察的）。

## 元组类型中的可选元素

元组类型现在允许在元素类型后添加 `?` 后缀，以表示该元素是可选的：

##### 示例

```ts
let t: [number, string?, boolean?]
t = [42, 'hello', true]
t = [42, 'hello']
t = [42]
```

在 [`strictNullChecks`](/tsconfig#strictNullChecks) 模式下，`?` 修饰符会自动在元素类型中包含 `undefined`，这与可选参数类似。

如果一个元素的类型带有后置 `?` 修饰符，且其右侧的所有元素也都带有 `?` 修饰符，则元组类型允许省略该元素。

当为 rest 参数推断元组类型时，源函数中的可选参数会在推断出的类型中转换为可选元组元素。

带有可选元素的元组类型的 `length` 属性是由表示可能长度的数字字面量类型构成的联合类型。
例如，元组类型 `[number, string?, boolean?]` 的 `length` 属性类型为 `1 | 2 | 3`。

### 元组类型中的 rest 元素

元组类型的最后一个元素可以是形如 `...X` 的 rest 元素，其中 `X` 是数组类型。
rest 元素表示该元组类型是开放式的，可以包含零个或多个数组元素类型的额外元素。
例如，`[number, ...string[]]` 表示以一个 `number` 元素开头，后跟任意数量 `string` 元素的元组。

##### 示例

```ts
function tuple<T extends any[]>(...args: T): T {
  return args
}

const numbers: number[] = getArrayOfNumbers()
const t1 = tuple('foo', 1, true) // [string, number, boolean]
const t2 = tuple('bar', ...numbers) // [string, ...number[]]
```

带有 rest 元素的元组类型的 `length` 属性类型为 `number`。

## 新增 `unknown` 顶类型

TypeScript 3.0 引入了全新的顶类型 `unknown`。
`unknown` 是 `any` 的类型安全对应版本。
任何类型都可以赋值给 `unknown`，但在未进行类型断言或基于控制流的类型收窄之前，`unknown` 除了赋给自身和 `any` 之外，不能赋给任何其他类型。
同样，在未先断言或收窄为更具体的类型之前，不允许对 `unknown` 执行任何操作。

##### 示例

```ts
// In an intersection everything absorbs unknown

type T00 = unknown & null // null
type T01 = unknown & undefined // undefined
type T02 = unknown & null & undefined // null & undefined (which becomes never)
type T03 = unknown & string // string
type T04 = unknown & string[] // string[]
type T05 = unknown & unknown // unknown
type T06 = unknown & any // any

// In a union an unknown absorbs everything

type T10 = unknown | null // unknown
type T11 = unknown | undefined // unknown
type T12 = unknown | null | undefined // unknown
type T13 = unknown | string // unknown
type T14 = unknown | string[] // unknown
type T15 = unknown | unknown // unknown
type T16 = unknown | any // any

// Type variable and unknown in union and intersection

type T20<T> = T & {} // T & {}
type T21<T> = T | {} // T | {}
type T22<T> = T & unknown // T
type T23<T> = T | unknown // unknown

// unknown in conditional types

type T30<T> = unknown extends T ? true : false // Deferred
type T31<T> = T extends unknown ? true : false // Deferred (so it distributes)
type T32<T> = never extends T ? true : false // true
type T33<T> = T extends never ? true : false // Deferred

// keyof unknown

type T40 = keyof any // string | number | symbol
type T41 = keyof unknown // never

// Only equality operators are allowed with unknown

function f10(x: unknown) {
  x == 5
  x !== 10
  x >= 0 // Error
  x + 1 // Error
  x * 2 // Error
  ;-x // Error
  ;+x // Error
}

// No property accesses, element accesses, or function calls

function f11(x: unknown) {
  x.foo // Error
  x[5] // Error
  x() // Error
  new x() // Error
}

// typeof, instanceof, and user defined type predicates

declare function isFunction(x: unknown): x is Function

function f20(x: unknown) {
  if (typeof x === 'string' || typeof x === 'number') {
    x // string | number
  }
  if (x instanceof Error) {
    x // Error
  }
  if (isFunction(x)) {
    x // Function
  }
}

// Homomorphic mapped type over unknown

type T50<T> = { [P in keyof T]: number }
type T51 = T50<any> // { [x: string]: number }
type T52 = T50<unknown> // {}

// Anything is assignable to unknown

function f21<T>(pAny: any, pNever: never, pT: T) {
  let x: unknown
  x = 123
  x = 'hello'
  x = [1, 2, 3]
  x = new Error()
  x = x
  x = pAny
  x = pNever
  x = pT
}

// unknown assignable only to itself and any

function f22(x: unknown) {
  let v1: any = x
  let v2: unknown = x
  let v3: object = x // Error
  let v4: string = x // Error
  let v5: string[] = x // Error
  let v6: {} = x // Error
  let v7: {} | null | undefined = x // Error
}

// Type parameter 'T extends unknown' not related to object

function f23<T extends unknown>(x: T) {
  let y: object = x // Error
}

// Anything but primitive assignable to { [x: string]: unknown }

function f24(x: { [x: string]: unknown }) {
  x = {}
  x = { a: 5 }
  x = [1, 2, 3]
  x = 123 // Error
}

// Locals of type unknown always considered initialized

function f25() {
  let x: unknown
  let y = x
}

// Spread of unknown causes result to be unknown

function f26(x: {}, y: unknown, z: any) {
  let o1 = { a: 42, ...x } // { a: number }
  let o2 = { a: 42, ...x, ...y } // unknown
  let o3 = { a: 42, ...x, ...y, ...z } // any
}

// Functions with unknown return type don't need return expressions

function f27(): unknown {}

// Rest type cannot be created from unknown

function f28(x: unknown) {
  let { ...a } = x // Error
}

// Class properties of type unknown don't need definite assignment

class C1 {
  a: string // Error
  b: unknown
  c: any
}
```

## 支持在 JSX 中使用 `defaultProps`

TypeScript 2.9 及更早版本无法在 JSX 组件中利用 [React `defaultProps`](https://reactjs.org/docs/typechecking-with-proptypes.html#default-prop-values) 声明。
用户通常不得不将属性声明为可选，并在 `render` 内部使用非空断言，或者在导出组件之前使用类型断言来修正组件的类型。

TypeScript 3.0 在 `JSX` 命名空间中新增了一个名为 `LibraryManagedAttributes` 的类型别名。
这个辅助类型在用于检查针对组件的 JSX 表达式之前，定义了对组件 `Props` 类型的转换规则；从而允许自定义诸如：如何处理传入的 prop 与推断出的 prop 之间的冲突、如何映射推断结果、如何处理可选性，以及如何合并来自不同位置的推断结果。

简而言之，利用这个通用类型，我们可以为 React 针对 `defaultProps` 以及在一定程度上对 `propTypes` 的特定行为进行建模。

```tsx
export interface Props {
  name: string
}

export class Greet extends React.Component<Props> {
  render() {
    const { name } = this.props
    return <div>Hello {name.toUpperCase()}!</div>
  }
  static defaultProps = { name: 'world' }
}

// Type-checks! No type assertions needed!
let el = <Greet />
```

## 注意事项

### `defaultProps` 上的显式类型

具有默认值的属性是从 `defaultProps` 属性类型中推断出来的。如果添加了显式类型注解，例如 `static defaultProps: Partial<Props>;`，编译器将无法识别哪些属性具有默认值（因为此时 `defaultProps` 的类型包含了 `Props` 的所有属性）。

请改用 `static defaultProps: Pick<Props, "name">;` 作为显式类型注解，或者像上面示例中那样不添加类型注解。

对于函数组件（此前称为 SFC），请使用 ES2015 默认初始值：

```tsx
function Greet({ name = 'world' }: Props) {
  return <div>Hello {name.toUpperCase()}!</div>
}
```

#### 对 `@types/React` 的更改

仍然需要在 `@types/React` 的 `JSX` 命名空间中添加对应的 `LibraryManagedAttributes` 定义变更。
请注意，目前仍存在一些限制。

## `/// <reference lib="..." />` 引用指令

TypeScript 新增了一种三斜线引用指令（`/// <reference lib="name" />`），允许文件显式引入现有的内置 _lib_ 文件。

内置 _lib_ 文件的引用方式与 _tsconfig.json_ 中的 [`lib`](/tsconfig#lib) 编译器选项相同（例如使用 `lib="es2015"` 而不是 `lib="lib.es2015.d.ts"` 等）。

对于依赖内置类型（例如 DOM API 或诸如 `Symbol`、`Iterable` 等内置 JS 运行时构造函数）的声明文件作者，推荐使用三斜线引用 lib 指令。此前，这些 .d.ts 文件必须对此类类型添加前向声明或重复声明。

##### 示例

在编译中的某个文件添加 `/// <reference lib="es2017.string" />`，等同于在编译时加上 `--lib es2017.string`。

```ts
/// <reference lib="es2017.string" />

'foo'.padStart(4)
```
