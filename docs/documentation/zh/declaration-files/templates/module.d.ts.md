---
title: 模块 .d.ts
layout: docs
permalink: /zh/docs/handbook/declaration-files/templates/module-d-ts.html
---

## 将 JavaScript 与 DTS 示例进行对比

## 常见的 CommonJS 模式

使用 CommonJS 模式的模块通过 `module.exports` 来描述导出的值。例如，以下模块导出了一个函数和一个数值常量：

```js
const maxInterval = 12

function getArrayLength(arr) {
  return arr.length
}

module.exports = {
  getArrayLength,
  maxInterval,
}
```

可以用如下 `.d.ts` 来描述它：

```ts
export function getArrayLength(arr: any[]): number
export const maxInterval: 12
```

TypeScript Playground 可以展示 JavaScript 代码对应的 `.d.ts`。你可以在此处[亲自尝试](/play?useJavaScript=true#code/GYVwdgxgLglg9mABAcwKZQIICcsEMCeAMqmMlABYAUuOAlIgN6IBQiiW6IWSNWAdABsSZcswC+zCAgDOURAFtcADwAq5GKUQBeRAEYATM2by4AExBC+qJQAc4WKNO2NWKdNjxFhFADSvFquqk4sxAA)。

`.d.ts` 语法特意设计得与 [ES 模块](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)语法相似。
ES 模块由 TC39 于 2015 年作为 ES2015 (ES6) 的一部分正式批准，在此之前也已经通过转译器使用了很长一段时间。如果你的 JavaScript 代码库使用的是 ES 模块：

```js
export function getArrayLength(arr) {
  return arr.length
}
```

它对应的 `.d.ts` 如下：

```ts
export function getArrayLength(arr: any[]): number
```

### 默认导出

在 CommonJS 中，你可以将任意值作为默认导出。例如，这是一个正则表达式模块：

```js
module.exports = /hello( world)?/
```

可以用如下 `.d.ts` 来描述：

```ts
declare const helloWorld: RegExp
export = helloWorld
```

或者一个数字：

```js
module.exports = 3.142
```

```ts
declare const pi: number
export = pi
```

CommonJS 中的一种导出风格是导出函数。
由于函数也是对象，因此可以向其添加额外的属性并一同导出。

```js
function getArrayLength(arr) {
  return arr.length
}
getArrayLength.maxInterval = 12

module.exports = getArrayLength
```

可以描述为：

```ts
declare function getArrayLength(arr: any[]): number
declare namespace getArrayLength {
  declare const maxInterval: 12
}

export = getArrayLength
```

有关其工作原理的详细信息，请参阅[模块函数](/docs/handbook/declaration-files/templates/module-function-d-ts.html)以及[模块参考](/docs/handbook/modules.html)页面。

## 处理多种消费导入方式

在现代使用方代码中，导入模块的方式有很多种：

```ts
const fastify = require('fastify')
const { fastify } = require('fastify')
import fastify = require('fastify')
import * as Fastify from 'fastify'
import { fastify, FastifyInstance } from 'fastify'
import fastify from 'fastify'
import fastify, { FastifyInstance } from 'fastify'
```

要覆盖所有这些情况，需要 JavaScript 代码在实际中支持所有这些模式。
为了支持其中的多种模式，CommonJS 模块通常需要类似如下写法：

```js
class FastifyInstance {}

function fastify() {
  return new FastifyInstance()
}

fastify.FastifyInstance = FastifyInstance

// 支持 { fastify }
fastify.fastify = fastify
// 支持严格的 ES 模块支持
fastify.default = fastify
// 设置默认导出
module.exports = fastify
```

## 模块中的类型

你可能希望为 JavaScript 代码中不存在的结构提供类型：

```js
function getArrayMetadata(arr) {
  return {
    length: getArrayLength(arr),
    firstObject: arr[0],
  }
}

module.exports = {
  getArrayMetadata,
}
```

可以描述为：

```ts
export type ArrayMetadata = {
  length: number
  firstObject: any | undefined
}
export function getArrayMetadata(arr: any[]): ArrayMetadata
```

这个示例非常适合[使用泛型](/docs/handbook/generics.html#generic-types)来提供更丰富的类型信息：

```ts
export type ArrayMetadata<ArrType> = {
  length: number
  firstObject: ArrType | undefined
}

export function getArrayMetadata<ArrType>(
  arr: ArrType[],
): ArrayMetadata<ArrType>
```

现在，数组的类型将传递给 `ArrayMetadata` 类型。

导出的类型随后可以通过 TypeScript 代码中的 `import` 或 `import type`，或者 [JSDoc 导入](/docs/handbook/jsdoc-supported-types.html#import-types)被模块的使用方复用。

### 模块代码中的命名空间

尝试描述 JavaScript 代码的运行时关系可能会比较棘手。
当类 ES 模块的语法不足以描述导出项时，你可以使用 `namespace`（命名空间）。

例如，你需要描述的类型可能足够复杂，以至于选择在 `.d.ts` 中使用命名空间对其进行组织：

```ts
// 这表示在运行时可用的 JavaScript 类
export class API {
  constructor(baseURL: string)
  getInfo(opts: API.InfoRequest): API.InfoResponse
}

// 该命名空间与 API 类合并，允许使用者以及本文件
// 拥有嵌套在各自独立部分中的类型。
declare namespace API {
  export interface InfoRequest {
    id: string
  }

  export interface InfoResponse {
    width: number
    height: number
  }
}
```

要了解命名空间在 `.d.ts` 文件中的工作机制，请阅读 [`.d.ts` 深入解析](/docs/handbook/declaration-files/deep-dive.html)。

### 可选的全局用法

你可以使用 `export as namespace` 声明你的模块在 UMD 上下文中可以在全局作用域中使用：

```ts
export as namespace moduleName
```

## 参考示例

为了让你了解所有这些部分是如何组合在一起的，以下是一个在创建新模块时可作为起点的参考 `.d.ts`：

```ts
// Type definitions for [~库名称~] [~可选版本号~]
// Project: [~项目名称~]
// Definitions by: [~你的姓名~] <[~你的 URL~]>

/*~ 这是模块模板文件。你应该将其重命名为 index.d.ts
 *~ 并放置在与该模块同名的文件夹中。
 *~ 例如，如果你正在为 "super-greeter" 编写文件，
 *~ 该文件应为 'super-greeter/index.d.ts'
 */

/*~ 如果该模块是一个 UMD 模块，在模块加载器环境之外加载时
 *~ 暴露了一个全局变量 'myLib'，请在此声明该全局变量。
 *~ 否则，请删除此声明。
 */
export as namespace myLib

/*~ 如果该模块导出函数，请像这样声明它们。
 */
export function myFunction(a: string): string
export function myOtherFunction(a: number): number

/*~ 你可以声明通过导入该模块即可使用的类型 */
export interface SomeType {
  name: string
  length: number
  extras?: string[]
}

/*~ 你可以使用 const、let 或 var 声明模块的属性 */
export const myField: number
```

### 库文件结构

声明文件的结构应当与库本身的结构保持一致。

一个库可以由多个模块组成，例如：

```
myLib
  +---- index.js
  +---- foo.js
  +---- bar
         +---- index.js
         +---- baz.js
```

它们可以像这样导入：

```js
var a = require('myLib')
var b = require('myLib/foo')
var c = require('myLib/bar')
var d = require('myLib/bar/baz')
```

因此你的声明文件应当是：

```
@types/myLib
  +---- index.d.ts
  +---- foo.d.ts
  +---- bar
         +---- index.d.ts
         +---- baz.d.ts
```

### 测试你的类型

如果你计划将这些更改提交到 DefinitelyTyped 供大家使用，我们建议你：

> 1. 在 `node_modules/@types/[libname]` 中创建一个新文件夹
> 2. 在该文件夹中创建 `index.d.ts`，并将示例复制进去
> 3. 查看模块的使用在哪些地方报错，并开始补全 `index.d.ts`
> 4. 满意后，克隆 [DefinitelyTyped/DefinitelyTyped](https://github.com/DefinitelyTyped) 并按照 README 中的说明操作。

否则：

> 1. 在源码树根目录下创建一个新文件：`[libname].d.ts`
> 2. 添加 `declare module "[libname]" {  }`
> 3. 将模板添加到 `declare module` 的大括号内，并查看你的使用在何处报错
