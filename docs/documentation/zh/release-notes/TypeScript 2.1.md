---
title: TypeScript 2.1
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-2-1.html
oneline: TypeScript 2.1 发布说明
---

## `keyof` 与查找类型

在 JavaScript 中，将属性名作为参数传入 API 是很常见的模式，但以往在 TypeScript 中无法表达这类 API 所涉及的类型关系。

索引类型查询（即 `keyof`）解决了这个问题。`keyof T` 会产生 `T` 所有合法属性名构成的类型，并被视为 `string` 的子类型。

##### 示例

```ts
interface Person {
  name: string
  age: number
  location: string
}

type K1 = keyof Person // "name" | "age" | "location"
type K2 = keyof Person[] // "length" | "push" | "pop" | "concat" | ...
type K3 = keyof { [x: string]: Person } // string
```

与之对应的是**索引访问类型**，也称为**查找类型**。其语法与元素访问相同，但写成类型形式：

##### 示例

```ts
type P1 = Person['name'] // string
type P2 = Person['name' | 'age'] // string | number
type P3 = string['charAt'] // (pos: number) => string
type P4 = string[]['push'] // (...items: string[]) => number
type P5 = string[][0] // string
```

结合类型系统的其他特性，这一模式可以实现类型安全的属性访问：

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key] // 推断类型为 T[K]
}

function setProperty<T, K extends keyof T>(obj: T, key: K, value: T[K]) {
  obj[key] = value
}

let x = { foo: 10, bar: 'hello!' }

let foo = getProperty(x, 'foo') // number
let bar = getProperty(x, 'bar') // string

let oops = getProperty(x, 'wargarbl') // 错误！"wargarbl" 不是 "foo" | "bar"

setProperty(x, 'foo', 'string') // 错误！应为 number 类型，传入了 string
```

## 映射类型

将现有类型的所有属性变为可选是一项常见需求。以 `Person` 为例：

```ts
interface Person {
  name: string
  age: number
  location: string
}
```

其对应的可选版本为：

```ts
interface PartialPerson {
  name?: string
  age?: number
  location?: string
}
```

借助映射类型，`PartialPerson` 可以写成对 `Person` 类型的通用变换：

```ts
type Partial<T> = {
  [P in keyof T]?: T[P]
}

type PartialPerson = Partial<Person>
```

映射类型通过遍历字面量类型的联合，为新对象类型生成一组属性。它类似于 [Python 中的列表推导式](https://docs.python.org/2/tutorial/datastructures.html#nested-list-comprehensions)，区别在于它生成的是类型的属性，而非列表中的元素。

除 `Partial` 外，映射类型还能表达许多有用的类型变换：

```ts
// 保持类型不变，但将每个属性设为只读。
type Readonly<T> = {
  readonly [P in keyof T]: T[P]
}

// 属性名称相同，但将值转换为 Promise 而不是具体值
type Deferred<T> = {
  [P in keyof T]: Promise<T[P]>
}

// 为 T 的属性包装 Proxy
type Proxify<T> = {
  [P in keyof T]: { get(): T[P]; set(v: T[P]): void }
}
```

## `Partial`、`Readonly`、`Record` 与 `Pick`

如上所述，`Partial` 和 `Readonly` 都是非常实用的工具类型，常用于描述典型的 JS 操作，例如：

```ts
function assign<T>(obj: T, props: Partial<T>): void
function freeze<T>(obj: T): Readonly<T>
```

因此，它们现已默认包含在标准库中。

此外，我们还新增了两个工具类型：`Record` 和 `Pick`。

```ts
// 从 T 中选取一组属性 K
declare function pick<T, K extends keyof T>(obj: T, ...keys: K[]): Pick<T, K>

const nameAndAgeOnly = pick(person, 'name', 'age') // { name: string, age: number }
```

```ts
// 对于类型 T 的每个属性 K，将其转换为 U
function mapObject<K extends string, T, U>(
  obj: Record<K, T>,
  f: (x: T) => U,
): Record<K, U>

const names = { foo: 'hello', bar: 'world', baz: 'bye' }
const lengths = mapObject(names, (s) => s.length) // { foo: number, bar: number, baz: number }
```

## 对象展开与剩余

TypeScript 2.1 新增了对 [ESnext 展开与剩余](https://github.com/sebmarkbage/ecmascript-rest-spread)的支持。

类似于数组展开，对象展开可以方便地获取对象的浅拷贝：

```ts
let copy = { ...original }
```

同样，也可以合并多个对象。下例中，`merged` 将拥有来自 `foo`、`bar` 和 `baz` 的所有属性：

```ts
let merged = { ...foo, ...bar, ...baz }
```

还可以覆盖已有属性或添加新属性：

```ts
let obj = { x: 1, y: 'string' }
var newObj = { ...obj, z: 3, y: 4 } // { x: number, y: number, z: number }
```

展开操作的顺序决定了最终对象的属性值：后面的展开会覆盖前面的同名属性。

对象剩余是对象展开的反向操作——在解构时，它可以收集所有未被显式提取的剩余属性：

```ts
let obj = { x: 1, y: 1, z: 1 }
let { z, ...obj1 } = obj
obj1 // {x: number, y:number};
```

## 向下编译的异步函数

此特性在 TypeScript 2.1 之前已存在，但仅支持以 ES6/ES2015 为编译目标。TypeScript 2.1 将此能力扩展至 ES3 和 ES5 运行时，使你在任何环境下都可以自由使用异步函数。

> 注意：首先，我们需要确保运行时全局提供符合 ECMAScript 规范的 `Promise`。
> 这可能涉及引入 `Promise` 的 [polyfill](https://github.com/stefanpenner/es6-promise)，或依赖目标运行时内置的实现。
> 此外，我们还需要通过将 [`lib`](/tsconfig#lib) 选项配置为诸如 `"dom", "es2015"` 或 `"dom", "es2015.promise", "es5"`，以确保 TypeScript 能够感知 `Promise` 的存在。

##### 示例

##### tsconfig.json

```json tsconfig
{
  "compilerOptions": {
    "lib": ["dom", "es2015.promise", "es5"]
  }
}
```

##### dramaticWelcome.ts

```ts
function delay(milliseconds: number) {
  return new Promise<void>((resolve) => {
    setTimeout(resolve, milliseconds)
  })
}

async function dramaticWelcome() {
  console.log('Hello')

  for (let i = 0; i < 3; i++) {
    await delay(500)
    console.log('.')
  }

  console.log('World!')
}

dramaticWelcome()
```

编译并运行输出结果，在 ES3/ES5 引擎上应能得到正确的行为。

## 支持外部辅助函数库（`tslib`）

TypeScript 会注入少量辅助函数，例如用于继承的 `__extends`、用于对象字面量和 JSX 元素中展开运算符的 `__assign`，以及用于异步函数的 `__awaiter`。

以往有两种处理方式：

1.  在**每个**需要辅助函数的文件中注入这些函数；
2.  通过 [`noEmitHelpers`](/tsconfig#noEmitHelpers) 完全不生成辅助函数。

这两种方案都不够理想：在每个文件中重复注入会让包体积变大；而不包含辅助函数则意味着用户必须自行维护辅助函数库。

TypeScript 2.1 允许将这些辅助函数集中到项目的一个单独模块中，编译器会按需生成对它们的导入语句。

首先，安装 [`tslib`](https://github.com/Microsoft/tslib) 工具库：

```sh
npm install tslib
```

然后，使用 [`importHelpers`](/tsconfig#importHelpers) 进行编译：

```sh
tsc --module commonjs --importHelpers a.ts
```

对于如下输入，生成的 `.js` 文件将包含对 `tslib` 的导入，并使用其中的 `__assign` 辅助函数，而非内联实现：

```ts
export const o = { a: 1, name: 'o' }
export const copy = { ...o }
```

```js
'use strict'
var tslib_1 = require('tslib')
exports.o = { a: 1, name: 'o' }
exports.copy = tslib_1.__assign({}, exports.o)
```

## 无类型导入

TypeScript 历来对模块导入方式要求严格，以便避免拼写错误和防止模块被错误使用。

然而很多时候，你只是想导入一个现有模块，而该模块可能没有对应的 `.d.ts` 文件。以前，这会直接报错；从 TypeScript 2.1 起，这变得简单多了。

TypeScript 2.1 允许在没有类型声明的情况下导入 JavaScript 模块。如果类型声明文件（如 `declare module "foo" { ... }` 或 `node_modules/@types/foo`）存在，它仍具有更高优先级。

在 [`noImplicitAny`](/tsconfig#noImplicitAny) 开启时，导入没有声明文件的模块仍会报错。

##### 示例

```ts
// 如果 `node_modules/asdf/index.js` 存在则成功
import { x } from 'asdf'
```

## 支持 `--target ES2016`、`--target ES2017` 与 `--target ESNext`

TypeScript 2.1 新增了三个编译目标值：`--target ES2016`、`--target ES2017` 和 `--target ESNext`。

使用 `--target ES2016` 时，编译器不会转换 ES2016 专有特性，例如 `**` 运算符。

类似地，`--target ES2017` 时，编译器不会转换 ES2017 专有特性，如 `async`/`await`。

`--target ESNext` 对应最新支持的 [ES 提案特性](https://github.com/tc39/proposals)。

## 改进的 `any` 类型推断

以前，当 TypeScript 无法确定变量类型时，会直接选择 `any` 类型。

```ts
let x // 隐式为 'any'
let y = [] // 隐式为 'any[]'

let z: any // 显式为 'any'。
```

TypeScript 2.1 改变了这一行为：它不再直接选择 `any`，而是根据后续赋值来推断类型。

此功能仅在 [`noImplicitAny`](/tsconfig#noImplicitAny) 开启时生效。

##### 示例

```ts
let x

// 你仍然可以给 'x' 赋任意值。
x = () => 42

// 在上次赋值后，TypeScript 2.1 已知 'x' 的类型为 '() => number'。
let y = x()

// 正因如此，它现在会提示你不能将数字与函数相加！
console.log(x + y)
//          ~~~~~
// 错误！运算符 '+' 不能应用于类型 '() => number' 和 'number'。

// TypeScript 仍然允许给 'x' 赋任意值。
x = 'Hello world!'

// 但现在它也知道 'x' 是一个 'string'！
x.toLowerCase()
```

同样的追踪机制也适用于空数组。

声明时无类型注解且初始值为 `[]` 的变量被视为隐式 `any[]`。但每次执行 `x.push(value)`、`x.unshift(value)` 或 `x[n] = value` 操作后，变量的类型都会随着添加的元素而**演化**。

```ts
function f1() {
  let x = []
  x.push(5)
  x[1] = 'hello'
  x.unshift(true)
  return x // (string | number | boolean)[]
}

function f2() {
  let x = null
  if (cond()) {
    x = []
    while (cond()) {
      x.push('hello')
    }
  }
  return x // string[] | null
}
```

## 隐式 any 错误

这一改进带来的一大好处是：在开启 [`noImplicitAny`](/tsconfig#noImplicitAny) 时，隐式 `any` 错误会大幅减少。只有当编译器在没有类型注解的情况下确实无法推断变量类型时，才会报告此错误。

##### 示例

```ts
function f3() {
  let x = [] // 错误：在某些无法确定类型的变量位置，变量 'x' 隐式具有类型 'any[]'。
  x.push(5)
  function g() {
    x // 错误：变量 'x' 隐式具有 'any[]' 类型。
  }
}
```

## 更精确的字面量类型推断

字符串、数字和布尔字面量类型（如 `"abc"`、`1` 和 `true`）以前只有在显式类型注解存在时才会被推断出来。从 TypeScript 2.1 起，`const` 变量和 `readonly` 属性**始终**会被推断为字面量类型。

对于没有类型注解的 `const` 变量或 `readonly` 属性，推断类型为其字面量初始值的类型。对于 `let` 变量、`var` 变量、参数或非 `readonly` 属性（有初始值但无类型注解），推断类型为其初始值字面量类型经过**拓宽**后的类型——字符串字面量类型拓宽为 `string`，数字字面量类型拓宽为 `number`，`true` 或 `false` 拓宽为 `boolean`，枚举字面量类型拓宽为其所在的枚举类型。

##### 示例

```ts
const c1 = 1 // 类型为 1
const c2 = c1 // 类型为 1
const c3 = 'abc' // 类型为 "abc"
const c4 = true // 类型为 true
const c5 = cond ? 1 : 'abc' // 类型为 1 | "abc"

let v1 = 1 // 类型为 number
let v2 = c2 // 类型为 number
let v3 = c3 // 类型为 string
let v4 = c4 // 类型为 boolean
let v5 = c5 // 类型为 number | string
```

字面量类型的拓宽行为可通过显式类型注解来控制。具体而言，当一个字面量类型的表达式在没有类型注解的 `const` 声明处被推断时，该 `const` 变量会得到一个可拓宽的字面量类型；而当 `const` 声明处存在显式字面量类型注解时，变量则会得到一个不可拓宽的字面量类型。

##### 示例

```ts
const c1 = 'hello' // 拓宽类型 "hello"
let v1 = c1 // 类型为 string

const c2: 'hello' = 'hello' // 类型为 "hello"
let v2 = c2 // 类型为 "hello"
```

## 将 super 调用的返回值用作 `this`

在 ES2015 中，若构造函数返回一个对象，该对象会隐式替换所有调用 `super()` 的子类中的 `this`。因此，必须捕获 `super()` 可能的返回值并用其替换 `this`。这一变更使得与[自定义元素（Custom Elements）](https://www.w3.org/TR/custom-elements/)的集成成为可能——自定义元素正是利用这一机制，通过用户编写的构造函数来初始化浏览器分配的元素。

##### 示例

```ts
class Base {
  x: number
  constructor() {
    // 返回不同于 `this` 的新对象
    return {
      x: 1,
    }
  }
}

class Derived extends Base {
  constructor() {
    super()
    this.x = 2
  }
}
```

生成代码：

```js
var Derived = (function (_super) {
  __extends(Derived, _super)
  function Derived() {
    var _this = _super.call(this) || this
    _this.x = 2
    return _this
  }
  return Derived
})(Base)
```

> 注意：该变更会对以 ES5/ES3 为目标时继承 `Error`、`Array`、`Map` 等内置类的行为带来破坏性改动。详情请参阅[继承内置类破坏性变更说明文档](https://github.com/Microsoft/TypeScript-wiki/blob/master/Breaking-Changes.md#extending-built-ins-like-error-array-and-map-may-no-longer-work)。

## 配置继承

项目通常有多个输出目标，例如 `ES5` 与 `ES2015`、调试版与生产版，或 `CommonJS` 与 `System`。这些目标之间往往只有少数配置项不同，维护多个 `tsconfig.json` 文件会相当繁琐。

TypeScript 2.1 通过 `extends` 支持配置继承，规则如下：

- `extends` 是 `tsconfig.json` 中新增的顶级属性（与 `compilerOptions`、[`files`](/tsconfig#files)、[`include`](/tsconfig#include) 和 [`exclude`](/tsconfig#exclude) 并列）。
- `extends` 的值必须是一个字符串，包含指向被继承配置文件的路径。
- 基础文件中的配置会先被加载，随后被继承文件中的配置覆盖。
- 配置文件之间不允许存在循环引用。
- 继承文件中的 [`files`](/tsconfig#files)、[`include`](/tsconfig#include) 和 [`exclude`](/tsconfig#exclude) 会**覆盖**基础文件中的对应配置。
- 配置文件中所有相对路径均相对于其所在配置文件解析。

##### 示例

`configs/base.json`：

```json tsconfig
{
  "compilerOptions": {
    "noImplicitAny": true,
    "strictNullChecks": true
  }
}
```

`tsconfig.json`：

```json tsconfig
{
  "extends": "./configs/base",
  "files": ["main.ts", "supplemental.ts"]
}
```

`tsconfig.nostrictnull.json`：

```json tsconfig
{
  "extends": "./tsconfig",
  "compilerOptions": {
    "strictNullChecks": false
  }
}
```

## 新增 `--alwaysStrict`

使用 [`alwaysStrict`](/tsconfig#alwaysStrict) 选项调用编译器会产生以下效果：

1. 以严格模式解析所有代码。
2. 在每个生成文件的顶部写入 `"use strict";` 指令。

模块代码会自动以严格模式解析，此新选项主要推荐用于非模块代码。
