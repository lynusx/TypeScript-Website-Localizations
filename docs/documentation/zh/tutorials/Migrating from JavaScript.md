---
title: 从 JavaScript 迁移
layout: docs
permalink: /zh/docs/handbook/migrating-from-javascript.html
oneline: 如何从 JavaScript 迁移到 TypeScript
---

TypeScript 并不是凭空诞生的。
它在设计之初就充分考虑了 JavaScript 生态系统，而如今现存的 JavaScript 代码量也非常庞大。
将 JavaScript 代码库转换为 TypeScript 虽然可能有些繁琐，但通常并不困难。
在本教程中，我们将介绍如何开启迁移之旅。
在此之前，我们假设你已经阅读了足够多的手册内容，能够编写新的 TypeScript 代码。

如果你打算转换 React 项目，建议先阅读 [React 转换指南](https://github.com/Microsoft/TypeScript-React-Conversion-Guide#typescript-react-conversion-guide)。

## 配置目录结构

如果你编写的是纯 JavaScript 代码，很可能是直接运行 JavaScript，
即 `.js` 文件位于 `src`、`lib` 或 `dist` 目录中，然后按需直接运行。

如果是这种情况，你编写的文件将作为 TypeScript 的输入，而你运行的将是它生成的输出文件。
在从 JS 迁移到 TS 的过程中，我们需要将输入文件分离开来，以防止 TypeScript 覆盖它们。
如果你的输出文件需要存放在特定目录中，那么该目录就是你的输出目录。

你可能还会对 JavaScript 执行某些中间构建步骤，例如打包或使用 Babel 等转译器。
在这种情况下，你可能已经搭建好了类似的文件夹结构。

从现在开始，我们假设你的目录结构大致如下：

```
projectRoot
├── src
│   ├── file1.js
│   └── file2.js
├── built
└── tsconfig.json
```

如果你的 `src` 目录外还有一个 `tests` 文件夹，你可能在 `src` 中有一个 `tsconfig.json`，在 `tests` 中也有一个。

## 编写配置文件

TypeScript 使用名为 `tsconfig.json` 的文件来管理项目配置，例如要包含哪些文件以及要执行哪些类型的检查。
让我们为项目创建一个最基础的配置文件：

```json
{
  "compilerOptions": {
    "outDir": "./built",
    "allowJs": true,
    "target": "es5"
  },
  "include": ["./src/**/*"]
}
```

这里我们向 TypeScript 指定了几项配置：

1. 读取 `src` 目录下它能识别的所有文件（通过 [`include`](/tsconfig#include)）。
2. 接受 JavaScript 文件作为输入（通过 [`allowJs`](/tsconfig#allowJs)）。
3. 将所有输出文件生成到 `built` 目录中（通过 [`outDir`](/tsconfig#outDir)）。
4. 将较新的 JavaScript 语法特性降级转换为较旧的版本，例如 ECMAScript 5（使用 [`target`](/tsconfig#target)）。

此时，如果你在项目根目录下尝试运行 `tsc`，应该会在 `built` 目录中看到输出文件。
`built` 中的文件布局应该与 `src` 中的布局完全一致。
现在，TypeScript 已经可以在你的项目中正常工作了。

## 早期收益

即使仅做到这一步，只要 TypeScript 能理解你的项目，你就能获得很多显著的好处。
如果你打开 [VS Code](https://code.visualstudio.com) 或 [Visual Studio](https://visualstudio.com) 等编辑器，就会发现通常能获得诸如代码补全等工具支持。
你还可以通过以下配置选项来捕获某些错误：

- [`noImplicitReturns`](/tsconfig#noImplicitReturns)：防止你在函数末尾遗漏 `return` 语句。
- [`noFallthroughCasesInSwitch`](/tsconfig#noFallthroughCasesInSwitch)：如果你不想遗漏 `switch` 块中各个 `case` 之间的 `break` 语句，这个选项会很有帮助。

TypeScript 还会对无法访问的代码和标签发出警告，你可以分别使用 [`allowUnreachableCode`](/tsconfig#allowUnreachableCode) 和 [`allowUnusedLabels`](/tsconfig#allowUnusedLabels) 来禁用这些警告。

## 与构建工具集成

你的流水线中可能还有其他构建步骤。
也许你需要向每个文件中拼接某些内容。
虽然每个构建工具各不相同，但我们会尽力介绍其核心要点。

### Gulp

如果你以某种方式使用了 Gulp，我们准备了一篇关于在 TypeScript 中[使用 Gulp](/docs/handbook/gulp.html) 并与 Browserify、Babelify 和 Uglify 等常见构建工具集成的教程。
你可以前往阅读更多内容。

### Webpack

与 Webpack 的集成非常简单。
你可以使用 TypeScript 加载器 `ts-loader`，并结合 `source-map-loader` 来简化调试。
只需运行：

```shell
npm install ts-loader source-map-loader
```

然后将以下选项合并到你的 `webpack.config.js` 文件中：

```js
module.exports = {
  entry: './src/index.ts',
  output: {
    filename: './dist/bundle.js',
  },

  // 启用 sourcemap 以调试 webpack 的输出。
  devtool: 'source-map',

  resolve: {
    // 将 '.ts' 和 '.tsx' 添加为可解析的扩展名。
    extensions: ['', '.webpack.js', '.web.js', '.ts', '.tsx', '.js'],
  },

  module: {
    rules: [
      // 所有带有 '.ts' 或 '.tsx' 扩展名的文件都将由 'ts-loader' 处理。
      { test: /\.tsx?$/, loader: 'ts-loader' },

      // 所有输出的 '.js' 文件的 sourcemap 都将由 'source-map-loader' 重新处理。
      { test: /\.js$/, loader: 'source-map-loader' },
    ],
  },

  // 其他配置项...
}
```

需要注意的是，ts-loader 必须在处理 `.js` 文件的任何其他 loader 之前运行。

你可以在我们的 [React 与 Webpack 教程](/docs/handbook/react-&-webpack.html)中查看使用 Webpack 的示例。

## 迁移到 TypeScript 文件

到了这一步，你可能已经准备好开始使用 TypeScript 文件了。
第一步是将你的某个 `.js` 文件重命名为 `.ts`。
如果该文件使用了 JSX，则需要将其重命名为 `.tsx`。

完成这一步了吗？
太棒了！
你已经成功将一个文件从 JavaScript 迁移到了 TypeScript！

当然，你可能会觉得哪里不太对劲。
如果在支持 TypeScript 的编辑器中打开该文件（或者运行 `tsc --pretty`），你可能会在某些行看到红色波浪线。
你可以像看待 Microsoft Word 等文字处理器中的红色波浪线一样看待它们。
即使存在这些提示，TypeScript 依然会转译你的代码，就像 Word 依然允许你打印文档一样。

如果觉得这样过于宽松，你可以收紧这种行为。
例如，如果你_不希望_ TypeScript 在出现错误时仍然编译为 JavaScript，可以使用 [`noEmitOnError`](/tsconfig#noEmitOnError) 选项。
从这个角度来说，TypeScript 的严格程度就像一个刻度盘，你可以根据需要调得尽可能高。

如果你打算使用更严格的可用设置，最好现在就开启（参见下方的[获取更严格的检查](#getting-stricter-checks)）。
例如，如果你绝不希望 TypeScript 在没有明确声明的情况下静默将类型推断为 `any`，可以在开始修改文件之前使用 [`noImplicitAny`](/tsconfig#noImplicitAny)。
虽然这在一开始可能让人有些不知所措，但长远收益会很快显现出来。

### 排查与消除错误

正如我们前面提到的，转换后出现错误信息是完全正常的。
重要的是逐一排查这些错误，并决定如何处理它们。
通常这些错误确实是潜在的 bug，但有时你只需要向 TypeScript 更清楚地解释你的意图。

#### 从模块导入

起初你可能会遇到大量形如 `Cannot find name 'require'.` 和 `Cannot find name 'define'.` 的错误。
在这些情况下，很可能是你使用了模块。
虽然你可以通过声明以下内容来让 TypeScript 认可它们的存在：

```ts
// 适用于 Node/CommonJS
declare function require(path: string): any
```

或者：

```ts
// 适用于 RequireJS/AMD
declare function define(...args: any[]): any
```

但更好的做法是去除这些调用，改用 TypeScript 语法进行导入。

首先，你需要通过设置 TypeScript 的 [`module`](/tsconfig#module) 选项来启用某种模块系统。
有效选项包括 `commonjs`、`amd`、`system` 和 `umd`。

如果你有以下 Node/CommonJS 代码：

```js
var foo = require('foo')

foo.doStuff()
```

或者以下 RequireJS/AMD 代码：

```js
define(['foo'], function (foo) {
  foo.doStuff()
})
```

那么你可以将其改写为如下 TypeScript 代码：

```ts
import foo = require('foo')

foo.doStuff()
```

#### 获取声明文件

如果你已经开始改用 TypeScript 导入，可能会遇到诸如 `Cannot find module 'foo'.` 之类的错误。
这里的问题在于你可能缺少描述该库的_声明文件_。
幸运的是，解决起来非常简单。
如果 TypeScript 对类似 `lodash` 的包报错，你只需运行：

```shell
npm install -S @types/lodash
```

如果你使用的模块选项不是 `commonjs`，则需要将 [`moduleResolution`](/tsconfig#moduleResolution) 选项设置为 `node`。

完成之后，你就可以顺利导入 lodash，并获得准确的代码补全。

#### 从模块导出

通常，从模块中导出内容涉及向 `exports` 或 `module.exports` 等对象添加属性。
TypeScript 允许你使用顶层的 export 语句。
例如，如果你像这样导出了一个函数：

```js
module.exports.feedPets = function (pets) {
  // ...
}
```

你可以将其改写如下：

```ts
export function feedPets(pets) {
  // ...
}
```

有时你可能会完全重写 exports 对象。
这是一种常见的模式，用于让模块可以被直接调用，如下面这段代码所示：

```js
var express = require('express')
var app = express()
```

以前你可能这样编写代码：

```js
function foo() {
  // ...
}
module.exports = foo
```

在 TypeScript 中，你可以使用 `export =` 语法来表示这种模式：

```ts
function foo() {
  // ...
}
export = foo
```

#### 参数过多或过少

有时你可能会发现调用函数时传入的参数过多或过少。
通常情况下这是一个 bug，但在某些情况下，你可能声明了一个使用 `arguments` 对象而非显式声明任何形参的函数：

```js
function myCoolFunction() {
  if (arguments.length == 2 && !Array.isArray(arguments[1])) {
    var f = arguments[0]
    var arr = arguments[1]
    // ...
  }
  // ...
}

myCoolFunction(
  function (x) {
    console.log(x)
  },
  [1, 2, 3, 4],
)
myCoolFunction(
  function (x) {
    console.log(x)
  },
  1,
  2,
  3,
  4,
)
```

在这种情况下，我们需要使用 TypeScript 的函数重载来告知所有调用者 `myCoolFunction` 的调用方式。

```ts
function myCoolFunction(f: (x: number) => void, nums: number[]): void
function myCoolFunction(f: (x: number) => void, ...nums: number[]): void
function myCoolFunction() {
  if (arguments.length == 2 && !Array.isArray(arguments[1])) {
    var f = arguments[0]
    var arr = arguments[1]
    // ...
  }
  // ...
}
```

我们为 `myCoolFunction` 添加了两个重载签名。
第一个签名声明 `myCoolFunction` 接受一个函数（该函数接受一个 `number`），以及一个 `number` 列表。
第二个签名声明它同样接受一个函数，并使用剩余参数（`...nums`）表明其后任意数量的参数都必须是 `number`。

#### 连续添加属性

有些人觉得先创建一个对象紧接着为其添加属性更加美观，如下所示：

```js
var options = {}
options.color = 'red'
options.volume = 11
```

TypeScript 会提示你不能给 `color` 和 `volume` 赋值，因为它推断出 `options` 的初始类型是 `{}`，而该类型没有任何属性。
如果你改为直接在对象字面量内部声明这些属性，就不会报错：

```ts
let options = {
  color: 'red',
  volume: 11,
}
```

你也可以定义 `options` 的类型，并在对象字面量上添加类型断言：

```ts
interface Options {
  color: string
  volume: number
}

let options = {} as Options
options.color = 'red'
options.volume = 11
```

或者，你也可以直接将 `options` 的类型指定为 `any`，这是最省事的做法，但能带来的收益也最少。

#### `any`、`Object` 与 `{}`

你可能会倾向于使用 `Object` 或 `{}` 来表示某个值可以拥有任意属性，因为在大多数情况下，`Object` 是最宽泛的类型。
然而，在这些场景下**你真正应该使用的是 `any`**，因为它是最_灵活_的类型。

例如，如果某个值的类型被标注为 `Object`，你将无法在其上调用 `toLowerCase()` 等方法。
更宽泛的类型通常意味着你能对其进行的操作更少，但 `any` 的特殊之处在于：它既是最通用的类型，同时又允许你对其执行任何操作。
这意味着你可以调用它、构造它、访问其属性等。
但请记住，一旦使用 `any`，你就会失去 TypeScript 提供的绝大部分错误检查和编辑器支持。

如果必须在 `Object` 和 `{}` 之间做出抉择，你应该优先选择 `{}`。
虽然两者大致相同，但在某些极少数的特殊情况下，技术上 `{}` 是比 `Object` 更宽泛的类型。

### 获取更严格的检查

TypeScript 自带了一些检查功能，可以为你的程序提供更高的安全性和更深入的分析。
一旦你将代码库转换为 TypeScript，就可以开始启用这些检查以提升安全性。

#### 禁用隐式 `any`

在某些情况下，TypeScript 无法推断出某些类型的具体值。
为了尽可能宽松，它会默认采用 `any` 类型。
虽然这非常有利于平滑迁移，但使用 `any` 意味着你无法获得任何类型安全保障，也无法享受在其他地方能获得的工具支持。
你可以通过 [`noImplicitAny`](/tsconfig#noImplicitAny) 选项来让 TypeScript 标出这些位置并报错。

#### 严格的 `null` 与 `undefined` 检查

默认情况下，TypeScript 假定 `null` 和 `undefined` 处于每个类型的值域中。
这意味着任何声明为 `number` 类型的值都可以是 `null` 或 `undefined`。
由于 `null` 和 `undefined` 在 JavaScript 和 TypeScript 中是极其常见的 bug 来源，因此 TypeScript 提供了 [`strictNullChecks`](/tsconfig#strictNullChecks) 选项，让你免受这些问题的困扰。

当启用 [`strictNullChecks`](/tsconfig#strictNullChecks) 时，`null` 和 `undefined` 会拥有各自独立的类型，分别称为 `null` 和 `undefined`。
当任何值_可能_为 `null` 时，你可以将其与原始类型组成联合类型。
例如，如果某个值可能是 `number` 或 `null`，你可以将类型写为 `number | null`。

如果你遇到某个 TypeScript 认为可能为 `null`/`undefined` 的值，而你明确知道它不可能是，可以使用后缀 `!` 操作符明确告知编译器：

```ts
declare var foo: string[] | null

foo.length // 错误 - 'foo' 可能为 'null'

foo!.length // 正常 - 'foo!' 的类型为 'string[]'
```

需要提醒的是，当使用 [`strictNullChecks`](/tsconfig#strictNullChecks) 时，你的依赖库可能也需要更新以支持 [`strictNullChecks`](/tsconfig#strictNullChecks)。

#### 对 `this` 禁用隐式 `any`

当你在类之外使用 `this` 关键字时，它默认具有 `any` 类型。
例如，假设有一个 `Point` 类，以及一个我们希望作为方法添加的函数：

```ts
class Point {
  constructor(
    public x,
    public y,
  ) {}
  getDistance(p: Point) {
    let dx = p.x - this.x
    let dy = p.y - this.y
    return Math.sqrt(dx ** 2 + dy ** 2)
  }
}
// ...

// 重新打开接口定义。
interface Point {
  distanceFromOrigin(): number
}
Point.prototype.distanceFromOrigin = function () {
  return this.getDistance({ x: 0, y: 0 })
}
```

这会出现我们前面提到的同样问题——我们可能会拼错 `getDistance` 却收不到任何错误提示。
出于这个原因，TypeScript 提供了 [`noImplicitThis`](/tsconfig#noImplicitThis) 选项。
设置该选项后，当在没有显式（或推断出）类型的情况下使用 `this` 时，TypeScript 将会报错。
解决方法是在接口或函数本身中使用 `this` 参数来提供显式类型：

```ts
Point.prototype.distanceFromOrigin = function (this: Point) {
  return this.getDistance({ x: 0, y: 0 })
}
```
