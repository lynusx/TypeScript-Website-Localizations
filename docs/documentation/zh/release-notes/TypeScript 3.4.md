---
title: TypeScript 3.4
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-3-4.html
oneline: TypeScript 3.4 发布说明
---

## 使用 `--incremental` 标志加快后续构建

TypeScript 3.4 引入了一个名为 [`incremental`](/tsconfig#incremental) 的新标志，用于指示 TypeScript 保存上次编译的项目图（project graph）信息。
下次使用 [`incremental`](/tsconfig#incremental) 调用 TypeScript 时，它将利用这些信息以开销最小的方式对项目进行类型检查并输出改动。

```jsonc tsconfig
// tsconfig.json
{
  "compilerOptions": {
    "incremental": true,
    "outDir": "./lib",
  },
  "include": ["./src"],
}
```

在使用上述配置时，默认情况下当我们运行 `tsc`，TypeScript 会在输出目录（`./lib`）中查找名为 `.tsbuildinfo` 的文件。
如果 `./lib/.tsbuildinfo` 不存在，它会被自动生成。
但如果它已存在，`tsc` 将尝试利用该文件进行增量类型检查并更新输出文件。

这些 `.tsbuildinfo` 文件可以安全地删除，并且在运行时对代码没有任何影响——它们纯粹用于加快编译速度。
我们还可以随意为其命名，并通过 [`tsBuildInfoFile`](/tsconfig#tsBuildInfoFile) 选项将其放置在任何位置。

```jsonc tsconfig
// front-end.tsconfig.json
{
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": "./buildcache/front-end",
    "outDir": "./lib",
  },
  "include": ["./src"],
}
```

### 复合项目

复合项目（即将 [`composite`](/tsconfig#composite) 设置为 `true` 的 `tsconfig.json`）的部分初衷在于使不同项目之间的引用能够以增量方式构建。
因此，复合项目**始终**会生成 `.tsbuildinfo` 文件。

### `outFile`

当使用 [`outFile`](/tsconfig#outFile) 时，构建信息文件的名称将基于输出文件的名称确定。
例如，如果输出的 JavaScript 文件是 `./output/foo.js`，那么在 [`incremental`](/tsconfig#incremental) 标志下，TypeScript 将生成 `./output/foo.tsbuildinfo` 文件。
与前面一样，这可以通过 [`tsBuildInfoFile`](/tsconfig#tsBuildInfoFile) 选项进行控制。

## 泛型函数的更高阶类型推断

TypeScript 3.4 现在可以在从其他泛型函数推断产生用于推断的自由类型变量时，生成泛型函数类型。
这意味着在 3.4 中，许多函数组合模式的表现得到了显著改善。

更具体地说，为了说明其动机，让我们考虑以下 `compose` 函数：

```ts
function compose<A, B, C>(f: (arg: A) => B, g: (arg: B) => C): (arg: A) => C {
  return (x) => g(f(x))
}
```

`compose` 接收另外两个函数：

- `f`：接收某个参数（类型为 `A`），并返回类型为 `B` 的值
- `g`：接收类型为 `B`（即 `f` 返回的类型）的参数，并返回类型为 `C` 的值

接着 `compose` 返回一个函数，该函数将接收的参数先传给 `f`，再将结果传给 `g`。

当调用该函数时，TypeScript 将通过称为*类型参数推断*（type argument inference）的过程尝试推导出 `A`、`B` 和 `C` 的类型。
这种推断过程通常运行良好：

```ts
interface Person {
  name: string
  age: number
}

function getDisplayName(p: Person) {
  return p.name.toLowerCase()
}

function getLength(s: string) {
  return s.length
}

// has type '(p: Person) => number'
const getDisplayNameLength = compose(getDisplayName, getLength)

// works and returns the type 'number'
getDisplayNameLength({ name: 'Person McPersonface', age: 42 })
```

这里的推断过程相当直接，因为 `getDisplayName` 和 `getLength` 使用的都是易于引用的具体类型。
然而，在 TypeScript 3.3 及更早版本中，当向类似 `compose` 这样的泛型函数传入其他泛型函数时，效果并不理想。

```ts
interface Box<T> {
  value: T
}

function makeArray<T>(x: T): T[] {
  return [x]
}

function makeBox<U>(value: U): Box<U> {
  return { value }
}

// has type '(arg: {}) => Box<{}[]>'
const makeBoxedArray = compose(makeArray, makeBox)

makeBoxedArray('hello!').value[0].toUpperCase()
//                                ~~~~~~~~~~~
// error: Property 'toUpperCase' does not exist on type '{}'.
```

在旧版本中，当从其他类型变量（如 `T` 和 `U`）进行推断时，TypeScript 会推断为空对象类型（`{}`）。

在 TypeScript 3.4 的类型参数推断过程中，对于返回函数类型的泛型函数调用，TypeScript _会_在适当时将泛型函数参数中的类型参数传递到结果函数类型中。

换句话说，TypeScript 3.4 不再生成如下类型：

```ts
;(arg: {}) => Box<{}[]>
```

而是生成如下类型：

```ts
;<T>(arg: T) => Box<T[]>
```

注意，`T` 已从 `makeArray` 传递到结果类型的类型参数列表中。
这意味着来自 `compose` 参数的泛型特性得到了保留，我们的 `makeBoxedArray` 示例即可正常工作！

```ts
interface Box<T> {
  value: T
}

function makeArray<T>(x: T): T[] {
  return [x]
}

function makeBox<U>(value: U): Box<U> {
  return { value }
}

// has type '<T>(arg: T) => Box<T[]>'
const makeBoxedArray = compose(makeArray, makeBox)

// works with no problem!
makeBoxedArray('hello!').value[0].toUpperCase()
```

更多详细信息，请参阅[原始变更 PR](https://github.com/Microsoft/TypeScript/pull/30215)。

## `ReadonlyArray` 与 `readonly` 元组的改进

TypeScript 3.4 让只读类数组类型的使用变得更加轻松。

### `ReadonlyArray` 的新语法

`ReadonlyArray` 类型用于描述只能读取的 `Array`。
任何持有 `ReadonlyArray` 引用的变量都无法添加、删除或替换数组中的任何元素。

```ts
function foo(arr: ReadonlyArray<string>) {
  arr.slice() // okay
  arr.push('hello!') // error!
}
```

虽然在不打算修改数组时使用 `ReadonlyArray` 替代 `Array` 是一种良好的实践，但鉴于常规数组具有更简洁的语法，此前使用它往往有些繁琐。
具体而言，`number[]` 是 `Array<number>` 的简写形式，正如 `Date[]` 是 `Array<Date>` 的简写一样。

TypeScript 3.4 针对数组类型引入了全新的 `readonly` 修饰符，为 `ReadonlyArray` 提供了新的简写语法。

```ts
function foo(arr: readonly string[]) {
  arr.slice() // okay
  arr.push('hello!') // error!
}
```

### `readonly` 元组

TypeScript 3.4 还引入了对 `readonly` 元组的支持。
我们可以像使用数组简写语法一样，在任何元组类型前添加 `readonly` 关键字将其声明为 `readonly` 元组。
正如你所预期的那样，常规元组的各个槽位是可写的，而 `readonly` 元组只允许从这些位置读取。

```ts
function foo(pair: readonly [string, string]) {
  console.log(pair[0]) // okay
  pair[1] = 'hello!' // error
}
```

正如常规元组是继承自 `Array` 的类型——元素类型为 <code>T<sub>1</sub></code>、<code>T<sub>2</sub></code>、……<code>T<sub>n</sub></code> 的元组继承自 <code>Array&lt; T<sub>1</sub> \| T<sub>2</sub> \| ... T<sub>n</sub> &gt;</code>——`readonly` 元组则是继承自 `ReadonlyArray` 的类型。因此，元素类型为 <code>T<sub>1</sub></code>、<code>T<sub>2</sub></code>、……<code>T<sub>n</sub></code> 的 `readonly` 元组继承自 <code>ReadonlyArray< T<sub>1</sub> \| T<sub>2</sub> | ... T<sub>n</sub></code>。

### `readonly` 映射类型修饰符与 `readonly` 数组

在 TypeScript 的早期版本中，我们对映射类型进行了泛化，使其能够对类数组类型进行差异化处理。
这意味着像 `Boxify` 这样的映射类型既可以作用于数组，也可以作用于元组。

```ts
interface Box<T> {
  value: T
}

type Boxify<T> = {
  [K in keyof T]: Box<T[K]>
}

// { a: Box<string>, b: Box<number> }
type A = Boxify<{ a: string; b: number }>

// Array<Box<number>>
type B = Boxify<number[]>

// [Box<string>, Box<number>]
type C = Boxify<[string, boolean]>
```

遗憾的是，像 `Readonly` 工具类型这样的映射类型此前在数组和元组类型上实际上是空操作（no-op）。

```ts
// lib.d.ts
type Readonly<T> = {
  readonly [K in keyof T]: T[K]
}

// How code acted *before* TypeScript 3.4

// { readonly a: string, readonly b: number }
type A = Readonly<{ a: string; b: number }>

// number[]
type B = Readonly<number[]>

// [string, boolean]
type C = Readonly<[string, boolean]>
```

在 TypeScript 3.4 中，映射类型中的 `readonly` 修饰符会自动将类数组类型转换为对应的 `readonly` 版本。

```ts
// How code acts now *with* TypeScript 3.4

// { readonly a: string, readonly b: number }
type A = Readonly<{ a: string; b: number }>

// readonly number[]
type B = Readonly<number[]>

// readonly [string, boolean]
type C = Readonly<[string, boolean]>
```

类似地，你可以编写一个像 `Writable` 这样的映射类型来移除 `readonly` 特性，从而将 `readonly` 数组容器转换回可变版本。

```ts
type Writable<T> = {
  -readonly [K in keyof T]: T[K]
}

// { a: string, b: number }
type A = Writable<{
  readonly a: string
  readonly b: number
}>

// number[]
type B = Writable<readonly number[]>

// [string, boolean]
type C = Writable<readonly [string, boolean]>
```

### 注意事项

需要注意的是，尽管形式如此，`readonly` 类型修饰符仅可用于数组类型和元组类型的语法中。
它并不是通用的类型操作符。

```ts
let err1: readonly Set<number> // error!
let err2: readonly Array<boolean> // error!

let okay: readonly boolean[] // works fine
```

更多详情可参阅[该 Pull Request](https://github.com/Microsoft/TypeScript/pull/29435)。

## `const` 断言

TypeScript 3.4 为字面量值引入了一种称为 _`const`_ 断言的新结构。
其语法为以 `const` 替代类型名称的类型断言（例如 `123 as const`）。
当使用 `const` 断言构造新的字面量表达式时，我们可以向语言提示：

- 该表达式中的字面量类型不应被拓宽（例如不会从 `"hello"` 拓宽为 `string`）
- 对象字面量将获得 `readonly` 属性
- 数组字面量将变为 `readonly` 元组

```ts
// Type '"hello"'
let x = 'hello' as const

// Type 'readonly [10, 20]'
let y = [10, 20] as const

// Type '{ readonly text: "hello" }'
let z = { text: 'hello' } as const
```

在 `.tsx` 文件之外，也可以使用尖括号断言语法。

```ts
// Type '"hello"'
let x = <const>'hello'

// Type 'readonly [10, 20]'
let y = <const>[10, 20]

// Type '{ readonly text: "hello" }'
let z = <const>{ text: 'hello' }
```

这项特性意味着，原本仅用于向编译器提示不可变性的类型注解往往可以省略。

```ts
// Works with no types referenced or declared.
// We only needed a single const assertion.
function getShapes() {
  let result = [
    { kind: 'circle', radius: 100 },
    { kind: 'square', sideLength: 50 },
  ] as const

  return result
}

for (const shape of getShapes()) {
  // Narrows perfectly!
  if (shape.kind === 'circle') {
    console.log('Circle radius', shape.radius)
  } else {
    console.log('Square side length', shape.sideLength)
  }
}
```

注意，上述代码无需任何类型注解。
`const` 断言使 TypeScript 能够为表达式推导出最精确的类型。

如果你选择不使用 TypeScript 的 `enum` 结构，这甚至可以用于在纯 JavaScript 代码中实现类似 `enum` 的模式。

```ts
export const Colors = {
  red: 'RED',
  blue: 'BLUE',
  green: 'GREEN',
} as const

// or use an 'export default'

export default {
  red: 'RED',
  blue: 'BLUE',
  green: 'GREEN',
} as const
```

### 注意事项

需要注意的一点是，`const` 断言只能直接应用于简单的字面量表达式。

```ts
// Error! A 'const' assertion can only be applied
// to a string, number, boolean, array, or object literal.
let a = (Math.random() < 0.5 ? 0 : 1) as const
let b = (60 * 60 * 1000) as const

// Works!
let c = Math.random() < 0.5 ? (0 as const) : (1 as const)
let d = 3_600_000 as const
```

另一点需要记住的是，`const` 上下文并不会立即将表达式转换为完全不可变的状态。

```ts
let arr = [1, 2, 3, 4]

let foo = {
  name: 'foo',
  contents: arr,
} as const

foo.name = 'bar' // error!
foo.contents = [] // error!

foo.contents.push(5) // ...works!
```

更多详情请参阅[对应的 Pull Request](https://github.com/Microsoft/TypeScript/pull/29510)。

## 对 `globalThis` 的类型检查

TypeScript 3.4 引入了对 ECMAScript 新增的 `globalThis` 的类型检查支持——顾名思义，这是一个指向全局作用域的全局变量。
与以往的解决方案不同，`globalThis` 提供了跨不同环境访问全局作用域的标准方式。

```ts
// in a global file:

var abc = 100

// Refers to 'abc' from above.
globalThis.abc = 200
```

请注意，使用 `let` 和 `const` 声明的全局变量不会挂载到 `globalThis` 上。

```ts
let answer = 42

// error! Property 'answer' does not exist on 'typeof globalThis'.
globalThis.answer = 333333
```

同样需要注意的是，当编译为较低版本的 ECMAScript 时，TypeScript 并不会转换对 `globalThis` 的引用。
因此，除非目标环境是现代常青浏览器（evergreen browsers，已原生支持 `globalThis`），否则你可能需要[使用相应的 polyfill](https://github.com/ljharb/globalThis)。

有关实现的更多详情，请参阅[该特性的 Pull Request](https://github.com/Microsoft/TypeScript/pull/29332)。
