---
title: 库结构
layout: docs
permalink: /zh/docs/handbook/declaration-files/library-structures.html
oneline: 如何组织你的 d.ts 文件结构
---

大体上讲，声明文件的*组织结构*取决于该库的使用方式。
在 JavaScript 中，提供可供使用的库有许多种方式，你需要编写与之相匹配的声明文件。
本指南介绍了如何识别常见的库模式，以及如何编写与这些模式相对应的声明文件。

每种主要的库结构模式在[模板](/docs/handbook/declaration-files/templates.html)部分中都有对应的文件。
你可以参考这些模板以更快地上手。

## 识别库的类型

首先，我们将梳理 TypeScript 声明文件所能表示的各类库。
我们将简要展示每种库是如何被*使用*的、如何被*编写*的，并列举一些现实中的示例库。

识别库的结构是编写其声明文件的第一步。
我们将提供基于*使用方式*以及基于*代码实现*来识别结构的线索。
根据库的文档和组织结构，其中一种方式可能会比另一种更容易。
建议你选择对自己更方便的方式。

### 你应该关注什么？

在为某个库编写类型定义时，可以先问自己以下几个问题：

1. 你是如何获取该库的？

   例如，它是*只能*通过 npm 获取，还是只能通过 CDN 获取？

2. 你会如何导入它？

   它是否会添加全局对象？它使用的是 `require` 还是 `import`/`export` 语句？

### 不同类型库的简要示例

### 模块化库

几乎所有现代 Node.js 库都属于模块体系。
这类库只能在具备模块加载器的 JS 环境中工作。
例如，`express` 仅能在 Node.js 中运行，并且必须使用 CommonJS 的 `require` 函数加载。

ECMAScript 2015（亦称 ES2015、ECMAScript 6 和 ES6）、CommonJS 以及 RequireJS 都有类似的*导入模块*的概念。
例如，在 JavaScript CommonJS（Node.js）中，你会写：

```js
var fs = require('fs')
```

在 TypeScript 或 ES6 中，`import` 关键字具有相同的用途：

```ts
import * as fs from 'fs'
```

在模块化库的文档中，你通常会看到包含以下某一行代码：

```js
var someLib = require('someLib')
```

或者

```js
define(..., ['someLib'], function(someLib) {

});
```

与全局模块类似，你也可能会在 [UMD](#umd) 模块的文档中看到这些示例，因此请务必检查其代码或文档。

#### 从代码识别模块化库

模块化库通常至少具备以下特征之一：

- 无条件调用 `require` 或 `define`
- 类似 `import * as a from 'b';` 或 `export c;` 的声明
- 对 `exports` 或 `module.exports` 的赋值

它们极少会包含：

- 对 `window` 或 `global` 属性的赋值

#### 模块模板

针对模块有 4 个可用的模板：
[`module.d.ts`](/docs/handbook/declaration-files/templates/module-d-ts.html)、[`module-class.d.ts`](/docs/handbook/declaration-files/templates/module-class-d-ts.html)、[`module-function.d.ts`](/docs/handbook/declaration-files/templates/module-function-d-ts.html) 和 [`module-plugin.d.ts`](/docs/handbook/declaration-files/templates/module-plugin-d-ts.html)。

建议你先阅读 [`module.d.ts`](/docs/handbook/declaration-files/templates/module-d-ts.html)，以了解它们整体的工作方式。

如果你的模块可以像函数一样被*调用*，请使用模板 [`module-function.d.ts`](/docs/handbook/declaration-files/templates/module-function-d-ts.html)：

```js
const x = require('foo')
// Note: calling 'x' as a function
const y = x(42)
```

如果你的模块可以使用 `new` 进行*构造*，请使用模板 [`module-class.d.ts`](/docs/handbook/declaration-files/templates/module-class-d-ts.html)：

```js
const x = require('bar')
// Note: using 'new' operator on the imported variable
const y = new x('hello')
```

如果你有一个在导入时会对其他模块进行修改的模块，请使用模板 [`module-plugin.d.ts`](/docs/handbook/declaration-files/templates/module-plugin-d-ts.html)：

```js
const jest = require('jest')
require('jest-matchers-files')
```

### 全局库

*全局*库是指可以直接从全局作用域访问的库（即不需要使用任何形式的 `import`）。
许多库仅暴露一个或多个全局变量供使用。
例如，如果你使用 [jQuery](https://jquery.com/)，只需直接引用 `$` 变量即可：

```ts
$(() => {
  console.log('hello!')
})
```

在全局库的文档中，你通常会看到关于如何在 HTML 的 script 标签中使用该库的指南：

```html
<script src="http://a.great.cdn.for/someLib.js"></script>
```

如今，大多数流行的全局可用库实际上都是作为 UMD 库编写的（见下文）。
UMD 库的文档与全局库的文档很难区分。
在编写全局声明文件之前，请确保该库实际上不是 UMD 库。

#### 从代码识别全局库

全局库的代码通常极其简单。
一个全局的 "Hello, world" 库可能长成这样：

```js
function createGreeting(s) {
  return 'Hello, ' + s
}
```

或者像这样：

```js
// Web
window.createGreeting = function (s) {
  return 'Hello, ' + s
}

// Node
global.createGreeting = function (s) {
  return 'Hello, ' + s
}

// Potentially any runtime
globalThis.createGreeting = function (s) {
  return 'Hello, ' + s
}
```

查看全局库的代码时，你通常会看到：

- 顶层的 `var` 语句或 `function` 声明
- 对 `window.someName` 的一次或多次赋值
- 假定存在 `document` 或 `window` 等 DOM 原生对象

你*不会*看到：

- 对 `require` 或 `define` 等模块加载器的检测或使用
- CommonJS/Node.js 风格的导入，如 `var fs = require("fs");`
- 对 `define(...)` 的调用
- 说明如何 `require` 或导入该库的文档

#### 全局库示例

由于将全局库转换为 UMD 库通常很容易，因此极少有流行的库仍以全局风格编写。
不过，某些小型且依赖 DOM（或者*没有任何*依赖）的库可能仍是全局库。

#### 全局库模板

模板文件 [`global.d.ts`](/docs/handbook/declaration-files/templates/global-d-ts.html) 定义了一个示例库 `myLib`。
请务必阅读[“防止名称冲突”脚注](#preventing-name-conflicts)。

### _UMD_

_UMD_ 模块是指*既*可以作为模块使用（通过导入），*又*可以作为全局库使用（在没有模块加载器的环境中运行时）。
许多流行的库（例如 [Moment.js](https://momentjs.com/)）都是以这种方式编写的。
例如，在 Node.js 中或使用 RequireJS 时，你会写：

```ts
import moment = require('moment')
console.log(moment.format())
```

而在原生浏览器环境中，你会写：

```js
console.log(moment.format())
```

#### 识别 UMD 库

[UMD 模块](https://github.com/umdjs/umd) 会检测是否存在模块加载器环境。
这是一种很容易发现的模式，大致如下所示：

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

如果你在库的代码中看到对 `typeof define`、`typeof window` 或 `typeof module` 的检测，尤其是位于文件顶部时，它几乎必定是一个 UMD 库。

UMD 库的文档通常还会演示一个使用 `require` 的“在 Node.js 中使用”示例，以及一个使用 `<script>` 标签加载脚本的“在浏览器中使用”示例。

#### UMD 库示例

大多数流行的库现在都以 UMD 包的形式提供。
例如 [jQuery](https://jquery.com/)、[Moment.js](https://momentjs.com/)、[lodash](https://lodash.com/) 等等。

#### 模板

请使用 [`module-plugin.d.ts`](/docs/handbook/declaration-files/templates/module-plugin-d-ts.html) 模板。

## 消费依赖

你的库可能会依赖几种不同类型的依赖项。
本节将展示如何将它们引入到声明文件中。

### 对全局库的依赖

如果你的库依赖某个全局库，请使用 `/// <reference types="..." />` 指令：

```ts
/// <reference types="someLib" />

function getThing(): someLib.thing
```

### 对模块的依赖

如果你的库依赖某个模块，请使用 `import` 语句：

```ts
import * as moment from 'moment'

function getThing(): moment
```

### 对 UMD 库的依赖

#### 来自全局库

如果你的全局库依赖某个 UMD 模块，请使用 `/// <reference types` 指令：

```ts
/// <reference types="moment" />

function getThing(): moment
```

#### 来自模块或 UMD 库

如果你的模块或 UMD 库依赖某个 UMD 库，请使用 `import` 语句：

```ts
import * as someLib from 'someLib'
```

*切勿*使用 `/// <reference` 指令来声明对 UMD 库的依赖！

## 脚注

### 防止名称冲突

请注意，在编写全局声明文件时，可以在全局作用域中定义许多类型。
我们强烈不建议这样做，因为当项目中存在多个声明文件时，这可能会导致无法解决的命名冲突。

一个简单的遵循原则是：仅在库所定义的全局变量所对应的*命名空间*下声明类型。
例如，如果该库定义了全局值 'cats'，你应该这样写：

```ts
declare namespace cats {
  interface KittySettings {}
}
```

而*不要*这样写：

```ts
// at top-level
interface CatsKittySettings {}
```

该指南还可以确保该库在未来迁移到 UMD 时，不会破坏声明文件使用者的现有代码。

### ES6 对模块调用签名的影响

许多流行的库（例如 Express）在导入时将其自身暴露为一个可调用的函数。
例如，典型的 Express 用法如下所示：

```ts
import exp = require('express')
var app = exp()
```

在符合 ES6 规范的模块加载器中，顶层对象（此处导入为 `exp`）只能拥有属性；
顶层模块对象*绝不可能*是可调用的。

最常见的解决方案是为可调用/可构造的对象定义一个 `default` 导出；
模块加载器通常会自动检测这种情况，并将顶层对象替换为 `default` 导出。
如果你在 tsconfig.json 中启用了 [`"esModuleInterop": true`](/tsconfig/#esModuleInterop)，TypeScript 也可以为你处理这种情况。
