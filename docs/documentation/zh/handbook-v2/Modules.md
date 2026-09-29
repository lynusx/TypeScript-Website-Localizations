---
title: 模块
layout: docs
permalink: /zh/docs/handbook/2/modules.html
oneline: 'JavaScript 如何处理跨文件边界的通信。'
---

JavaScript 有着漫长的、以各种方式处理代码模块化的历史。TypeScript 自 2012 年问世以来，已经实现了对其中许多格式的支持，但随着时间的推移，社区和 JavaScript 规范已经统一到一种称为 ES 模块（ES Modules，或 ES6 模块）的格式上。你可能更熟悉它的 `import`/`export` 语法。

ES 模块于 2015 年被加入 JavaScript 规范，到 2020 年已在大多数 Web 浏览器和 JavaScript 运行时中得到广泛支持。

为了突出重点，本手册将同时介绍 ES 模块及其流行的前身 CommonJS 的 `module.exports =` 语法；你可以在 [模块](/docs/handbook/modules.html) 下的参考部分找到其他模块模式的信息。

## JavaScript 模块是如何定义的

在 TypeScript 中，与 ECMAScript 2015 一样，任何包含顶层 `import` 或 `export` 的文件都被视为模块。

反之，没有任何顶层 import 或 export 声明的文件会被视为一个脚本，其内容在全局作用域中可见（因此对模块也可见）。

模块在自己的作用域内执行，而不是在全局作用域内。这意味着在模块中声明的变量、函数、类等在模块外部不可见，除非使用某种导出形式将它们显式导出。相反，要使用从另一个模块导出的变量、函数、类、接口等，必须使用某种导入形式将其导入。

## 非模块

在开始之前，重要的是要理解 TypeScript 如何界定一个模块。JavaScript 规范声明：任何没有 `import` 声明、`export` 或顶层 `await` 的 JavaScript 文件都应被视为脚本而不是模块。

在脚本文件中，变量和类型被声明在共享的全局作用域中，通常假定你要么使用 [`outFile`](/tsconfig#outFile) 编译器选项将多个输入文件合并为一个输出文件，要么在 HTML 中使用多个 `<script>` 标签来加载这些文件（且顺序要正确！）。

如果你有一个目前没有任何 `import` 或 `export`、但希望它被当作模块处理，请添加这样一行：

```ts twoslash
export {}
```

这会将该文件变为一个不导出任何内容的模块。无论你的模块编译目标是什么，该语法都有效。

## TypeScript 中的模块

<blockquote class='bg-reading'>
   <p>扩展阅读：<br />
   <a href='https://exploringjs.com/impatient-js/ch_modules.html#overview-syntax-of-ecmascript-modules'>Impatient JS (Modules)</a><br/>
   <a href='https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Modules'>MDN: JavaScript 模块</a><br/>
   </p>
</blockquote>

在 TypeScript 中编写基于模块的代码时，需要考虑三个主要问题：

- **语法**：我想使用什么语法来导入和导出内容？
- **模块解析**：模块名（或路径）与磁盘上的文件之间是什么关系？
- **模块输出目标**：我生成的 JavaScript 模块应该是什么样子？

### ES 模块语法

文件可以通过 `export default` 声明一个主导出：

```ts twoslash
// @filename: hello.ts
export default function helloWorld() {
  console.log('Hello, world!')
}
```

然后这样导入它：

```ts twoslash
// @filename: hello.ts
export default function helloWorld() {
  console.log('Hello, world!')
}
// @filename: index.ts
// ---cut---
import helloWorld from './hello.js'
helloWorld()
```

除了默认导出之外，你还可以省略 `default`，通过 `export` 导出多个变量和函数：

```ts twoslash
// @filename: maths.ts
export var pi = 3.14
export let squareTwo = 1.41
export const phi = 1.61

export class RandomNumberGenerator {}

export function absolute(num: number) {
  if (num < 0) return num * -1
  return num
}
```

这些导出可以通过 `import` 语法在另一个文件中使用：

```ts twoslash
// @filename: maths.ts
export var pi = 3.14
export let squareTwo = 1.41
export const phi = 1.61
export class RandomNumberGenerator {}
export function absolute(num: number) {
  if (num < 0) return num * -1
  return num
}
// @filename: app.ts
// ---cut---
import { pi, phi, absolute } from './maths.js'

console.log(pi)
const absPhi = absolute(phi)
//    ^?
```

### 其他导入语法

可以使用 `import {old as new}` 这样的格式对导入进行重命名：

```ts twoslash
// @filename: maths.ts
export var pi = 3.14
// @filename: app.ts
// ---cut---
import { pi as π } from './maths.js'

console.log(π)
//          ^?
```

你可以将上述语法混合到一个 `import` 中：

```ts twoslash
// @filename: maths.ts
export const pi = 3.14
export default class RandomNumberGenerator {}

// @filename: app.ts
import RandomNumberGenerator, { pi as π } from './maths.js'

RandomNumberGenerator
// ^?

console.log(π)
//          ^?
```

你可以使用 `* as name` 将所有导出的对象放入一个单独的命名空间中：

```ts twoslash
// @filename: maths.ts
export var pi = 3.14
export let squareTwo = 1.41
export const phi = 1.61

export function absolute(num: number) {
  if (num < 0) return num * -1
  return num
}
// ---cut---
// @filename: app.ts
import * as math from './maths.js'

console.log(math.pi)
const positivePhi = math.absolute(math.phi)
//    ^?
```

你可以通过 `import "./file"` 导入一个文件，而 _不_ 将任何变量引入当前模块：

```ts twoslash
// @filename: maths.ts
export var pi = 3.14
// ---cut---
// @filename: app.ts
import './maths.js'

console.log('3.14')
```

在这种情况下，`import` 什么也不做。但是，`maths.ts` 中的所有代码都会被执行，这可能触发影响其他对象的副作用。

#### TypeScript 特有的 ES 模块语法

Types can be exported and imported using the same syntax as JavaScript values:

```ts twoslash
// @filename: animal.ts
export type Cat = { breed: string; yearOfBirth: number }

export interface Dog {
  breeds: string[]
  yearOfBirth: number
}

// @filename: app.ts
import { Cat, Dog } from './animal.js'
type Animals = Cat | Dog
```

TypeScript 扩展了 import 语法，引入了两个用于声明类型导入的概念：

###### `import type`

这是一种 _只能_ 导入类型的导入语句：

```ts twoslash
// @filename: animal.ts
export type Cat = { breed: string; yearOfBirth: number }
export type Dog = { breeds: string[]; yearOfBirth: number }
export const createCatName = () => 'fluffy'

// @filename: valid.ts
import type { Cat, Dog } from './animal.js'
export type Animals = Cat | Dog

// @filename: app.ts
// @errors: 1361
import type { createCatName } from './animal.js'
const name = createCatName()
```

###### 内联 `type` 导入

TypeScript 4.5 还允许给单个导入加上 `type` 前缀，以表明被导入的引用是一个类型：

```ts twoslash
// @filename: animal.ts
export type Cat = { breed: string; yearOfBirth: number }
export type Dog = { breeds: string[]; yearOfBirth: number }
export const createCatName = () => 'fluffy'
// ---cut---
// @filename: app.ts
import { createCatName, type Cat, type Dog } from './animal.js'

export type Animals = Cat | Dog
const name = createCatName()
```

这些特性加在一起，可以让 Babel、swc 或 esbuild 这样的非 TypeScript 转译器知道哪些导入可以安全地移除。

#### 与 CommonJS 行为对应的 ES 模块语法

TypeScript 有一种 ES 模块语法，与 CommonJS 和 AMD 的 `require` _直接_ 对应。在 _大多数情况下_，使用 ES 模块的导入与那些环境中的 `require` 是相同的，但这种语法可以确保你的 TypeScript 文件与 CommonJS 输出保持一对一的对应关系：

```ts twoslash
/// <reference types="node" />
// @module: commonjs
// ---cut---
import fs = require('fs')
const code = fs.readFileSync('hello.ts', 'utf8')
```

你可以在[模块参考页](/docs/handbook/modules.html#export--and-import--require)了解更多关于此语法的信息。

## CommonJS 语法

CommonJS 是 npm 上大多数模块的发布格式。即使你使用上面的 ES 模块语法编写代码，简单了解 CommonJS 语法的工作原理也能让你更轻松地进行调试。

#### 导出

标识符通过在一个名为 `module` 的全局对象上设置 `exports` 属性来导出。

```ts twoslash
/// <reference types="node" />
// ---cut---
function absolute(num: number) {
  if (num < 0) return num * -1
  return num
}

module.exports = {
  pi: 3.14,
  squareTwo: 1.41,
  phi: 1.61,
  absolute,
}
```

然后这些文件可以通过 `require` 语句导入：

```ts twoslash
// @module: commonjs
// @filename: maths.ts
/// <reference types="node" />
function absolute(num: number) {
  if (num < 0) return num * -1
  return num
}

module.exports = {
  pi: 3.14,
  squareTwo: 1.41,
  phi: 1.61,
  absolute,
}
// @filename: index.ts
// ---cut---
const maths = require('./maths')
maths.pi
//    ^?
```

或者，你可以使用 JavaScript 中的解构特性稍作简化：

```ts twoslash
// @module: commonjs
// @filename: maths.ts
/// <reference types="node" />
function absolute(num: number) {
  if (num < 0) return num * -1
  return num
}

module.exports = {
  pi: 3.14,
  squareTwo: 1.41,
  phi: 1.61,
  absolute,
}
// @filename: index.ts
// ---cut---
const { squareTwo } = require('./maths')
squareTwo
// ^?
```

### CommonJS 与 ES 模块互操作

CommonJS 和 ES 模块在特性上存在不匹配，主要体现在默认导入与模块命名空间对象导入的区别上。TypeScript 提供了一个编译器标志 [`esModuleInterop`](/tsconfig#esModuleInterop)，以减少这两套不同约束之间的冲突。

## TypeScript 的模块解析选项

模块解析是获取 `import` 或 `require` 语句中的字符串，并确定该字符串所指代文件的过程。

TypeScript 包含两种解析策略：Classic 和 Node。Classic 是编译器选项 [`module`](/tsconfig#module) 不是 `commonjs` 时的默认策略，它的存在是为了向后兼容。
Node 策略复刻了 Node.js 在 CommonJS 模式下的工作方式，并额外检查 `.ts` 和 `.d.ts`。

有许多 TSConfig 标志会影响 TypeScript 中的模块策略：[`moduleResolution`](/tsconfig#moduleResolution)、[`baseUrl`](/tsconfig#baseUrl)、[`paths`](/tsconfig#paths)、[`rootDirs`](/tsconfig#rootDirs)。

关于这些策略工作原理的完整细节，请参阅 [模块解析](/docs/handbook/modules/reference.html#the-moduleresolution-compiler-option) 参考页面。

## TypeScript 的模块输出选项

有两个选项会影响输出的 JavaScript：

- [`target`](/tsconfig#target) 决定哪些 JS 特性会被降级（转换为可在较旧的 JavaScript 运行时中运行），哪些保持不变。
- [`module`](/tsconfig#module) 决定模块之间使用什么代码进行交互。

使用哪种 [`target`](/tsconfig#target) 取决于你预期运行 TypeScript 代码的 JavaScript 运行时所支持的特性。它可能是你支持的最旧的 Web 浏览器、你预期运行的最低版本的 Node.js，或者来自运行时的特定约束 —— 比如 Electron。

模块之间的所有通信均通过模块加载器进行，编译器选项 [`module`](/tsconfig#module) 决定使用哪一个。在运行时，模块加载器负责在执行模块之前，定位并执行该模块的所有依赖项。

例如，下面是一个使用 ES 模块语法的 TypeScript 文件，展示了 [`module`](/tsconfig#module) 的几个不同选项：

```ts twoslash
// @filename: constants.ts
export const valueOfPi = 3.142
// @filename: index.ts
// ---cut---
import { valueOfPi } from './constants.js'

export const twoPi = valueOfPi * 2
```

#### `ES2020`

```ts twoslash
// @showEmit
// @module: es2020
// @noErrors
import { valueOfPi } from './constants.js'

export const twoPi = valueOfPi * 2
```

#### `CommonJS`

```ts twoslash
// @showEmit
// @module: commonjs
// @noErrors
import { valueOfPi } from './constants.js'

export const twoPi = valueOfPi * 2
```

#### `UMD`

```ts twoslash
// @showEmit
// @module: umd
// @noErrors
import { valueOfPi } from './constants.js'

export const twoPi = valueOfPi * 2
```

> 注意，ES2020 的输出实际上与原始的 `index.ts` 相同。

你可以在 [`module` 的 TSConfig 参考](/tsconfig/#module) 中查看所有可用选项以及它们生成的 JavaScript 代码长什么样。

## TypeScript 命名空间

TypeScript 有自己的模块格式，称为 `namespaces`（命名空间），它早于 ES 模块标准。这种语法在创建复杂的定义文件方面有许多有用的特性，并且仍[在 DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) 中被积极使用。虽然命名空间并未被弃用，但它的绝大多数特性在 ES 模块中都存在，我们建议你使用 ES 模块以与 JavaScript 的发展方向保持一致。你可以在[命名空间参考页](/docs/handbook/namespaces.html)了解更多关于命名空间的信息。
