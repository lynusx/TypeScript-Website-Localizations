---
title: 模块
layout: docs
permalink: /zh/docs/handbook/2/modules.html
oneline: 'JavaScript 如何处理跨文件边界的代码通信。'
---

JavaScript 在处理代码模块化方面有着悠久的历史，曾出现过多种不同的方案。
TypeScript 自 2012 年诞生以来，实现了对其中许多模块格式的支持，但随着时间的推移，社区和 JavaScript 规范最终汇聚到了一种名为 ES Modules（或 ES6 模块）的格式上。你可能更熟悉它的 `import`/`export` 语法。

ES Modules 于 2015 年被正式纳入 JavaScript 规范，并在 2020 年左右获得了绝大多数 Web 浏览器和 JavaScript 运行时环境的广泛支持。

为了聚焦重点，本手册将涵盖 ES Modules 及其广泛流行的前身 CommonJS 的 `module.exports =` 语法；你可以在参考部分的[模块](/docs/handbook/modules.html)章节中找到关于其他模块模式的详细信息。

## JavaScript 模块是如何定义的

在 TypeScript 中，与 ECMAScript 2015 一样，任何包含顶层 `import` 或 `export` 的文件都被视为模块。

相反，没有任何顶层 import 或 export 声明的文件会被视为脚本，其内容在全局作用域中可用（因此对各个模块也是可见的）。

模块在其自身的作用域内执行，而不是在全局作用域内。
这意味着在模块中声明的变量、函数、类等，除非使用某种导出语法显式导出，否则在模块外部是不可见的。
反之，要想使用从其他模块导出的变量、函数、类、接口等，必须使用某种导入语法将其导入。

## 非模块

在开始之前，了解 TypeScript 将什么视作模块非常重要。
JavaScript 规范规定，任何不包含 `import` 声明、`export` 或顶层 `await` 的 JavaScript 文件都应被视为脚本，而非模块。

在脚本文件中，变量和类型都声明在共享的全局作用域内，此时通常预期你将通过 [`outFile`](/tsconfig#outFile) 编译器选项将多个输入文件合并为一个输出文件，或者在 HTML 中使用多个 `<script>` 标签来加载这些文件（且必须保证正确的加载顺序！）。

如果你手头有一个当前没有任何 `import` 或 `export` 的文件，但希望它被视为模块，只需添加这一行：

```ts twoslash
export {}
```

这会将该文件转变为一个不导出任何内容的模块。无论你的模块目标（module target）设置如何，此语法都能生效。

## TypeScript 中的模块

<blockquote class='bg-reading'>
   <p>延伸阅读：<br />
   <a href='https://exploringjs.com/impatient-js/ch_modules.html#overview-syntax-of-ecmascript-modules'>Impatient JS (Modules)</a><br/>
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules'>MDN: JavaScript 模块</a><br/>
   </p>
</blockquote>

在 TypeScript 中编写基于模块的代码时，主要需要考虑三个方面：

- **语法（Syntax）**：我想使用什么语法来导入和导出内容？
- **模块解析（Module Resolution）**：模块名称（或路径）与磁盘上的文件之间存在什么对应关系？
- **模块输出目标（Module Output Target）**：生成的 JavaScript 模块代码应该是什么样子的？

### ES 模块语法

一个文件可以通过 `export default` 声明一个默认导出：

```ts twoslash
// @filename: hello.ts
export default function helloWorld() {
  console.log('Hello, world!')
}
```

然后通过以下方式导入：

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

除了默认导出外，你还可以通过省略 `default` 的 `export` 导出多个变量和函数：

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

这些内容可以在另一个文件中通过 `import` 语法引入并使用：

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

可以使用形如 `import {old as new}` 的格式对导入项进行重命名：

```ts twoslash
// @filename: maths.ts
export var pi = 3.14
// @filename: app.ts
// ---cut---
import { pi as π } from './maths.js'

console.log(π)
//          ^?
```

你可以将上述语法混合搭配在单个 `import` 中：

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

你可以使用 `* as name` 将所有导出的对象整合到一个命名空间中：

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

你可以通过 `import "./file"` 仅导入文件而*不*向当前模块中引入任何变量：

```ts twoslash
// @filename: maths.ts
export var pi = 3.14
// ---cut---
// @filename: app.ts
import './maths.js'

console.log('3.14')
```

在这种情况下，`import` 并不会引入任何绑定。然而，`maths.ts` 中的所有代码都会被执行，这可能会触发影响其他对象的副作用（side-effects）。

#### TypeScript 特有的 ES 模块语法

类型可以使用与 JavaScript 值相同的语法进行导出和导入：

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

TypeScript 扩展了 `import` 语法，提供了两种声明类型导入的概念：

###### `import type`

这是一种*只能*导入类型的导入语句：

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

TypeScript 4.5 还允许为单个导入项添加 `type` 前缀，以指明导入的引用是一个类型：

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

结合这些语法，像 Babel、swc 或 esbuild 这样的非 TypeScript 转译器就能清楚地知道哪些导入可以安全地移除。

#### 具有 CommonJS 行为特性的 ES 模块语法

TypeScript 提供了一种与 CommonJS 和 AMD 的 `require` *直接*对应的 ES 模块语法。在*大多数情况下*，使用 ES 模块导入与这些环境下的 `require` 行为相同，但该语法可确保你的 TypeScript 文件与 CommonJS 输出形成一一对应的关系：

```ts twoslash
/// <reference types="node" />
// @module: commonjs
// ---cut---
import fs = require('fs')
const code = fs.readFileSync('hello.ts', 'utf8')
```

你可以在[模块参考页面](/docs/handbook/modules.html#export--and-import--require)中了解关于该语法的更多信息。

## CommonJS 语法

CommonJS 是 npm 上绝大多数模块采用的分发格式。即使你使用的是上述 ES 模块语法进行编写，简要了解 CommonJS 语法的工作原理也将有助于你更轻松地排查和调试问题。

#### 导出

标识符是通过在全局对象 `module` 的 `exports` 属性上赋值来进行导出的。

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

随后，这些文件可以通过 `require` 语句进行导入：

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

或者你可以使用 JavaScript 的解构特性将其稍作简化：

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

### CommonJS 与 ES 模块互操作性

在默认导入与模块命名空间对象导入之间的差异上，CommonJS 与 ES 模块在特性上存在不匹配。TypeScript 提供了一个编译器标志 [`esModuleInterop`](/tsconfig#esModuleInterop)，以减少两套不同约束体系之间的摩擦与冲突。

## TypeScript 的模块解析选项

模块解析（Module resolution）是指从 `import` 或 `require` 语句中提取字符串，并确定该字符串所引用的文件的过程。

TypeScript 包含两种解析策略：Classic 和 Node。当编译器选项 [`module`](/tsconfig#module) 不是 `commonjs` 时，默认使用 Classic 策略，保留它是为了向后兼容。
Node 策略则复现了 Node.js 在 CommonJS 模式下的工作方式，并针对 `.ts` 和 `.d.ts` 文件增加了额外的检查。

TypeScript 中有许多 TSConfig 配置项会影响模块策略：[`moduleResolution`](/tsconfig#moduleResolution)、[`baseUrl`](/tsconfig#baseUrl)、[`paths`](/tsconfig#paths)、[`rootDirs`](/tsconfig#rootDirs)。

关于这些策略工作机制的完整详情，你可以查阅[模块解析](/docs/handbook/modules/reference.html#the-moduleresolution-compiler-option)参考页面。

## TypeScript 的模块输出选项

有两个选项会影响生成的 JavaScript 输出：

- [`target`](/tsconfig#target)：决定哪些 JS 特性会被降级（转换为可在较旧的 JavaScript 运行时中运行的代码），哪些特性会原样保留
- [`module`](/tsconfig#module)：决定模块之间相互交互时所采用的代码格式

使用哪种 [`target`](/tsconfig#target) 取决于你期望运行 TypeScript 代码的 JavaScript 运行时所支持的特性。这可能取决于：你支持的最旧的 Web 浏览器、你期望运行的最低 Node.js 版本，或者来自运行时的独特约束——例如 Electron。

模块之间的所有通信都是通过模块加载器完成的，编译器选项 [`module`](/tsconfig#module) 决定了使用哪一个加载器。
在运行时，模块加载器负责在执行某个模块之前定位并执行该模块的所有依赖项。

例如，下面是一个使用 ES 模块语法的 TypeScript 文件，展示了 [`module`](/tsconfig#module) 的几种不同配置选项：

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

> 请注意，ES2020 的输出实际上与原本的 `index.ts` 完全一致。

你可以在 [`module` 的 TSConfig 参考](/tsconfig#module)中查看所有可用选项以及它们生成的 JavaScript 代码。

## TypeScript 命名空间

TypeScript 拥有自己的模块格式，称为命名空间（`namespaces`），它的出现早于 ES 模块标准。这种语法在创建复杂的类型声明文件时有许多实用的特性，并且至今仍[在 DefinitelyTyped 中](https://github.com/DefinitelyTyped/DefinitelyTyped)被广泛使用。尽管命名空间并未被废弃，但其绝大多数特性在 ES 模块中都已具备，我们推荐你使用 ES 模块以与 JavaScript 的发展方向保持一致。你可以在[命名空间参考页面](/docs/handbook/namespaces.html)中了解更多关于命名空间的内容。
