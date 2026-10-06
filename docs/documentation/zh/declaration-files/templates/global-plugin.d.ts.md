---
title: '全局插件 .d.ts'
layout: docs
permalink: /zh/docs/handbook/declaration-files/templates/global-plugin-d-ts.html
---

## _UMD_

_UMD_ 模块既可以作为模块使用（通过导入），也可以作为全局对象使用（在没有模块加载器的环境中运行时）。
许多流行的库，例如 [Moment.js](http://momentjs.com/)，都是以此种方式编写的。
例如，在 Node.js 中或使用 RequireJS 时，你可以这样写：

```ts
import moment = require('moment')
console.log(moment.format())
```

而在原生浏览器环境中，你可以这样写：

```js
console.log(moment.format())
```

### 识别 UMD 库

[UMD 模块](https://github.com/umdjs/umd)会检查是否存在模块加载器环境。
这是一种很容易识别的模式，通常类似于：

```js
(function (root, factory) {
    if (typeof define === "function" && define.amd) {
        define(["libName"], factory);
    } else if (typeof module === "object" && module.exports) {
        module.exports = factory(require("libName"));
    } else {
        root.returnExports = factory(root.libName);
    }
}(this, function (b) {
```

如果你在某个库的代码中看到针对 `typeof define`、`typeof window` 或 `typeof module` 的检测（特别是在文件顶部），那么它几乎肯定是一个 UMD 库。

UMD 库的文档通常还会提供一个展示 `require` 的“在 Node.js 中使用”示例，
以及一个展示如何通过 `<script>` 标签加载脚本的“在浏览器中使用”示例。

### UMD 库示例

大多数流行的库现在都以 UMD 包的形式提供。
例如 [jQuery](https://jquery.com/)、[Moment.js](http://momentjs.com/)、[lodash](https://lodash.com/) 等等。

### 模板

模块有三个可用模板：
[`module.d.ts`](/docs/handbook/declaration-files/templates/module-d-ts.html)、[`module-class.d.ts`](/docs/handbook/declaration-files/templates/module-class-d-ts.html) 和 [`module-function.d.ts`](/docs/handbook/declaration-files/templates/module-function-d-ts.html)。

如果你的模块可以像函数一样被_调用_，请使用 [`module-function.d.ts`](/docs/handbook/declaration-files/templates/module-function-d-ts.html)：

```js
var x = require('foo')
// 注意：将 'x' 作为函数调用
var y = x(42)
```

请务必阅读[尾注“ES6 对模块调用签名的影响”](#the-impact-of-es6-on-module-plugins)。

如果你的模块可以使用 `new` 进行_构造_，请使用 [`module-class.d.ts`](/docs/handbook/declaration-files/templates/module-class-d-ts.html)：

```js
var x = require('bar')
// 注意：对导入的变量使用 'new' 运算符
var y = new x('hello')
```

相同的[尾注](#the-impact-of-es6-on-module-plugins)也适用于此类模块。

如果你的模块既不可调用也不可构造，请使用 [`module.d.ts`](/docs/handbook/declaration-files/templates/module-d-ts.html) 文件。

## _模块插件_ 或 _UMD 插件_

_模块插件_会修改另一个模块（无论是 UMD 还是普通模块）的形态。
例如在 Moment.js 中，`moment-range` 为 `moment` 对象添加了一个新的 `range` 方法。

就编写声明文件而言，无论被修改的模块是普通模块还是 UMD 模块，你所编写的代码都是相同的。

### 模板

使用 [`module-plugin.d.ts`](/docs/handbook/declaration-files/templates/module-plugin-d-ts.html) 模板。

## _全局插件_

_全局插件_是一种修改某些全局对象形态的全局代码。
与_全局修改模块_一样，它们带来了运行时冲突的可能性。

例如，某些库会向 `Array.prototype` 或 `String.prototype` 添加新函数。

### 识别全局插件

全局插件通常很容易从其文档中识别出来。

你会看到类似于这样的示例：

```js
var x = 'hello, world'
// 在内置类型上创建新方法
console.log(x.startsWithHello())

var y = [1, 2, 3]
// 在内置类型上创建新方法
console.log(y.reverseAndSort())
```

### 模板

使用 [`global-plugin.d.ts`](/docs/handbook/declaration-files/templates/global-plugin-d-ts.html) 模板。

## _全局修改模块_

_全局修改模块_在被导入时会修改全局作用域中的现有值。
例如，可能存在一个库在导入时向 `String.prototype` 添加新成员。
由于可能引发运行时冲突，这种模式存在一定风险，
但我们仍然可以为其编写声明文件。

### 识别全局修改模块

全局修改模块通常很容易从其文档中识别出来。
通常来说，它们类似于全局插件，但需要通过 `require` 调用来使其生效。

你可能会看到类似这样的文档：

```js
// 不使用其返回值的 'require' 调用
var unused = require('magic-string-time')
/* 或者 */
require('magic-string-time')

var x = 'hello, world'
// 在内置类型上创建新方法
console.log(x.startsWithHello())

var y = [1, 2, 3]
// 在内置类型上创建新方法
console.log(y.reverseAndSort())
```

### 模板

使用 [`global-modifying-module.d.ts`](/docs/handbook/declaration-files/templates/global-modifying-module-d-ts.html) 模板。

## 使用依赖项

你的库可能会有几种不同类型的依赖项。
本节将展示如何在声明文件中引入它们。

### 对全局库的依赖

如果你的库依赖于某个全局库，请使用 `/// <reference types="..." />` 指令：

```ts
/// <reference types="someLib" />

function getThing(): someLib.thing
```

### 对模块的依赖

如果你的库依赖于某个模块，请使用 `import` 语句：

```ts
import * as moment from 'moment'

function getThing(): moment
```

### 对 UMD 库的依赖

#### 在全局库中

如果你的全局库依赖于某个 UMD 模块，请使用 `/// <reference types` 指令：

```ts
/// <reference types="moment" />

function getThing(): moment
```

#### 在模块或 UMD 库中

如果你的模块或 UMD 库依赖于某个 UMD 库，请使用 `import` 语句：

```ts
import * as someLib from 'someLib'
```

请_不要_使用 `/// <reference` 指令来声明对 UMD 库的依赖！

## 尾注

### 防止名称冲突

请注意，在编写全局声明文件时，完全可以在全局作用域中定义许多类型。
我们强烈不建议这样做，因为当项目中存在多个声明文件时，这可能会导致无法解决的命名冲突。

一个简单的遵循原则是：仅通过该库所定义的全局变量在命名空间（_namespaced_）内部声明类型。
例如，如果该库定义了全局值 'cats'，你应该这样写：

```ts
declare namespace cats {
  interface KittySettings {}
}
```

而_不要_这样写：

```ts
// 顶层作用域
interface CatsKittySettings {}
```

这条指导原则还能确保该库在未来迁移到 UMD 时，不会破坏该声明文件使用者的代码。

### ES6 对模块插件的影响

某些插件会向现有模块添加或修改顶层导出项。
虽然这在 CommonJS 和其他加载器中是合法的，但 ES6 模块被视为不可变的，因此这种模式将无法使用。
由于 TypeScript 与模块加载器无关，所以在编译期并不会强制执行这一策略，但打算迁移到 ES6 模块加载器的开发者应当注意这一点。

### ES6 对模块调用签名的影响

许多流行的库（例如 Express）在被导入时会将自身暴露为一个可调用函数。
例如，典型的 Express 用法如下所示：

```ts
import exp = require('express')
var app = exp()
```

在 ES6 模块加载器中，顶层对象（此处导入为 `exp`）只能拥有属性；
顶层模块对象_绝不能_直接作为函数调用。
最常见的解决方案是为可调用/可构造的对象定义一个 `default` 导出；
某些模块加载器的 shim 会自动检测到这种情况，并将顶层对象替换为 `default` 导出。

### 库的文件结构

声明文件的结构应当反映出库的文件结构。

一个库可能由多个模块组成，例如：

```
myLib
  +---- index.js
  +---- foo.js
  +---- bar
         +---- index.js
         +---- baz.js
```

它们可能会被这样导入：

```js
var a = require('myLib')
var b = require('myLib/foo')
var c = require('myLib/bar')
var d = require('myLib/bar/baz')
```

因此，你的声明文件结构应当为：

```
@types/myLib
  +---- index.d.ts
  +---- foo.d.ts
  +---- bar
         +---- index.d.ts
         +---- baz.d.ts
```

```ts
// Type definitions for [~库名称~] [~可选版本号~]
// Project: [~项目名称~]
// Definitions by: [~你的姓名~] <[~你的 URL~]>

/*~ 此模板展示了如何编写全局插件。 */

/*~ 为原始类型编写声明并添加新成员。
 *~ 例如，这里为内置的 number 类型添加了带有重载的
 *~ 'toBinaryString' 方法。
 */
interface Number {
  toBinaryString(opts?: MyLibrary.BinaryFormatOptions): string

  toBinaryString(
    callback: MyLibrary.BinaryFormatCallback,
    opts?: MyLibrary.BinaryFormatOptions,
  ): string
}

/*~ 如果需要声明多个类型，请将它们放在命名空间内，
 *~ 以避免向全局命名空间添加过多内容。
 */
declare namespace MyLibrary {
  type BinaryFormatCallback = (n: number) => string
  interface BinaryFormatOptions {
    prefix?: string
    padding: number
  }
}
```
