---
title: TypeScript 1.5
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-1-5.html
oneline: TypeScript 1.5 发布说明
---

## ES6 模块

TypeScript 1.5 支持 ECMAScript 6（ES6）模块。
ES6 模块本质上是采用了新语法的 TypeScript 外部模块：ES6 模块是独立加载的源文件，可以导入其他模块，并提供若干可供外部访问的导出项。
ES6 模块引入了数种新的导出与导入声明。
建议将 TypeScript 库和应用程序更新为使用这种新语法，但这并非硬性要求。
新的 ES6 模块语法与 TypeScript 原有的内部模块及外部模块结构共存，这些结构可以根据需要混合使用。

#### 导出声明

除了 TypeScript 原本就支持的在声明前添加 `export` 修饰之外，还可以使用独立的导出声明来导出模块成员，并可通过 `as` 子句为导出项指定不同的外部名称。

```ts
interface Stream { ... }
function writeToStream(stream: Stream, data: string) { ... }
export { Stream, writeToStream as write };  // writeToStream exported as write
```

导入声明同样可以使用 `as` 子句为导入项指定不同的本地名称。例如：

```ts
import { read, write, standardOutput as stdout } from './inout'
var s = read(stdout)
write(stdout, s)
```

作为单个导入的替代方案，可以使用命名空间导入（namespace import）来导入整个模块：

```ts
import * as io from './inout'
var s = io.read(io.standardOutput)
io.write(io.standardOutput, s)
```

#### 重新导出

通过 `from` 子句，一个模块可以将指定模块的导出项复制到当前模块，而无需引入本地名称。

```ts
export { read, write, standardOutput as stdout } from './inout'
```

`export *` 可用于重新导出另一模块的所有导出项。这在创建聚合多个其他模块导出项的模块时非常有用。

```ts
export function transform(s: string): string { ... }
export * from "./mod1";
export * from "./mod2";
```

#### 默认导出

默认导出声明（export default）指定一个表达式，该表达式将作为模块的默认导出项：

```ts
export default class Greeter {
  sayHello() {
    console.log('Greetings!')
  }
}
```

随后可以使用默认导入将其引入：

```ts
import Greeter from './greeter'
var g = new Greeter()
g.sayHello()
```

#### 裸导入

“裸导入”（Bare Import）可用于仅为执行副作用而导入模块。

```ts
import './polyfills'
```

有关模块的更多信息，请参阅 [ES6 模块支持规范](https://github.com/Microsoft/TypeScript/issues/2242)。

## 声明与赋值中的解构

TypeScript 1.5 增加了对 ES6 解构声明与解构赋值的支持。

#### 解构声明

解构声明会引入一个或多个命名变量，并使用从对象属性或数组元素中提取的值来初始化它们。

例如，以下示例声明了变量 `x`、`y` 和 `z`，并将它们分别初始化为 `getSomeObject().x`、`getSomeObject().y` 以及 `getSomeObject().z`：

```ts
var { x, y, z } = getSomeObject()
```

解构声明同样适用于从数组中提取值：

```ts
var [x, y, z = 10] = getSomeArray()
```

类似地，解构也可以用于函数形参声明中：

```ts
function drawText({ text = '', location: [x, y] = [0, 0], bold = false }) {
  // Draw text
}

// Call drawText with an object literal
var item = { text: 'someText', location: [1, 2, 3], style: 'italics' }
drawText(item)
```

#### 解构赋值

解构模式也可以用于普通的赋值表达式中。
例如，交换两个变量的值可以写成一个单独的解构赋值表达式：

```ts
var x = 1
var y = 2
;[x, y] = [y, x]
```

## `namespace` 关键字

此前 TypeScript 使用 `module` 关键字同时定义“内部模块（internal modules）”和“外部模块（external modules）”；
这往往会让刚接触 TypeScript 的开发者感到困惑。
“内部模块”更接近大多数人所说的命名空间（namespace）；同样，在现代 JS 语境中，“外部模块”现在就单纯是指模块。

> 注意：以前定义内部模块的语法依然受支持。

**修改前**：

```ts
module Math {
    export function add(x, y) { ... }
}
```

**修改后**：

```ts
namespace Math {
    export function add(x, y) { ... }
}
```

## `let` 与 `const` 支持

在面向 ES3 和 ES5 时，现已支持 ES6 的 `let` 与 `const` 声明。

#### 常量

```ts
const MAX = 100

++MAX // Error: The operand of an increment or decrement
//        operator cannot be a constant.
```

#### 块级作用域

```ts
if (true) {
  let a = 4
  // use a
} else {
  let a = 'string'
  // use a
}

alert(a) // Error: a is not defined in this scope.
```

## `for..of` 循环支持

TypeScript 1.5 增加了在面向 ES3/ES5 时对数组的 ES6 `for..of` 循环支持，并在面向 ES6 时提供对 Iterator 接口的完整支持。

##### 示例

当面向 ES3/ES5 编译时，TypeScript 编译器会将数组的 `for..of` 转译为符合惯用写法的 ES3/ES5 JavaScript：

```ts
for (var v of expr) {
}
```

将输出为：

```js
for (var _i = 0, _a = expr; _i < _a.length; _i++) {
  var v = _a[_i]
}
```

## 装饰器

> TypeScript 装饰器基于 [ES7 装饰器提案](https://github.com/wycats/javascript-decorators)。

装饰器是一个：

- 表达式
- 其求值结果为一个函数
- 该函数接收 target、name 以及 property descriptor 作为参数
- 并可以选择返回一个属性描述符以安装在目标对象上

> 有关更多信息，请参阅[装饰器](https://github.com/Microsoft/TypeScript/issues/2249)提案。

##### 示例

装饰器 `readonly` 和 `enumerable(false)` 将在属性 `method` 安装到类 `C` 之前应用于该属性。
这使得装饰器能够修改其实现，而在本例中，将描述符扩充为 writable: false 和 enumerable: false。

```ts
class C {
  @readonly
  @enumerable(false)
  method() { ... }
}

function readonly(target, key, descriptor) {
    descriptor.writable = false;
}

function enumerable(value) {
    return function (target, key, descriptor) {
        descriptor.enumerable = value;
    };
}
```

## 计算属性

使用动态属性初始化对象以往略显繁琐。以下面的代码为例：

```ts
type NeighborMap = { [name: string]: Node }
type Node = { name: string; neighbors: NeighborMap }

function makeNode(name: string, initialNeighbor: Node): Node {
  var neighbors: NeighborMap = {}
  neighbors[initialNeighbor.name] = initialNeighbor
  return { name: name, neighbors: neighbors }
}
```

这里我们需要创建一个变量来暂存邻居映射表，以便对其进行初始化。
而在 TypeScript 1.5 中，我们可以让编译器来完成繁琐的工作：

```ts
function makeNode(name: string, initialNeighbor: Node): Node {
  return {
    name: name,
    neighbors: {
      [initialNeighbor.name]: initialNeighbor,
    },
  }
}
```

## 支持 `UMD` 与 `System` 模块输出

除了 `AMD` 和 `CommonJS` 模块加载器之外，TypeScript 现已支持将模块输出为 `UMD`（[Universal Module Definition](https://github.com/umdjs/umd)）和 [`System`](https://github.com/systemjs/systemjs) 模块格式。

**用法**：

> tsc --module umd

以及

> tsc --module system

## 字符串中的 Unicode 码点转义

ES6 引入了新的转义形式，允许用户仅使用单个转义序列来表示 Unicode 码点。

例如，假设需要对包含字符 '𠮷' 的字符串进行转义。
在 UTF-16/UCS2 中，'𠮷' 被表示为一个代理对（surrogate pair），即由一对 16 位代码单元的值进行编码，具体为 `0xD842` 和 `0xDFB7`。
此前这意味着你必须将该码点转义为 `"\uD842\uDFB7"`。
这种方式的主要缺点是很难分辨这究竟是两个独立的字符还是一个代理对。

借助 ES6 的码点转义，你可以在普通字符串和模板字符串中使用单个转义序列简洁清晰地表示该字符：`"\u{20bb7}"`。
在面向 ES3/ES5 编译时，TypeScript 会将其输出为 `"\uD842\uDFB7"`。

## ES3/ES5 中的带标签模板字符串

在 TypeScript 1.4 中，我们为所有目标版本添加了模板字符串支持，但带标签的模板字符串仅支持 ES6。
感谢 [@ivogabe](https://github.com/ivogabe) 付出的巨大努力，我们在 ES3 和 ES5 中也填补了带标签模板字符串的空白。

当面向 ES3/ES5 时，以下代码：

```ts
function oddRawStrings(strs: TemplateStringsArray, n1, n2) {
  return strs.raw.filter((raw, index) => index % 2 === 1)
}

oddRawStrings`Hello \n${123} \t ${456}\n world`
```

将输出为：

```js
function oddRawStrings(strs, n1, n2) {
  return strs.raw.filter(function (raw, index) {
    return index % 2 === 1
  })
}
;((_a = ['Hello \n', ' \t ', '\n world']),
  (_a.raw = ['Hello \\n', ' \\t ', '\\n world']),
  oddRawStrings(_a, 123, 456))
var _a
```

## AMD 依赖的可选名称

`/// <amd-dependency path="x" />` 用于通知编译器：生成的目标模块在 require 调用中需要注入一个非 TS 模块依赖；
然而，此前在 TS 代码中无法直接使用该模块。

新增的 `amd-dependency name` 属性允许为 AMD 依赖传递一个可选名称：

```ts
/// <amd-dependency path="legacy/moduleA" name="moduleA"/>
declare var moduleA: MyType
moduleA.callStuff()
```

生成的 JS 代码：

```js
define(['require', 'exports', 'legacy/moduleA'], function (
  require,
  exports,
  moduleA,
) {
  moduleA.callStuff()
})
```

## 通过 `tsconfig.json` 支持项目配置

在某个目录下添加 `tsconfig.json` 文件表明该目录是 TypeScript 项目的根目录。
`tsconfig.json` 文件指定了编译该项目所需的根文件和编译器选项。项目可以通过以下方式之一进行编译：

- 在不传入输入文件的情况下直接调用 tsc，此时编译器将从当前目录开始并沿着父目录链向上搜索 tsconfig.json 文件。
- 在不传入输入文件的情况下调用 tsc 并指定 -project（或简写 -p）命令行选项，该选项指定包含 tsconfig.json 文件的目录路径。

##### 示例

```json tsconfig
{
  "compilerOptions": {
    "module": "commonjs",
    "noImplicitAny": true,
    "sourceMap": true
  }
}
```

更多详情请参阅 [tsconfig.json wiki 页面](https://github.com/Microsoft/TypeScript/wiki/tsconfig.json)。

## `--rootDir` 命令行选项

选项 [`outDir`](/tsconfig#outDir) 会在输出目录中保留输入的目录层级结构。
编译器通过计算所有输入文件的最长公共路径来确定输入文件的根目录；
然后根据该根目录在输出中复现其所有子结构。

有时这种行为并不符合预期，例如对于输入文件 `FolderA\FolderB\1.ts` 和 `FolderA\FolderB\2.ts`，输出结构会镜像反映 `FolderA\FolderB\`。
此时若在输入中添加一个新文件 `FolderA\3.ts`，输出结构就会跳出一层变成镜像反映 `FolderA\`。

[`rootDir`](/tsconfig#rootDir) 用于显式指定需要在输出中镜像保留的输入目录，而不是自动计算它。

## `--noEmitHelpers` 命令行选项

TypeScript 编译器会在需要时生成一些辅助函数，例如 `__extends`。
这些辅助函数会被生成到引用了它们的每个文件中。
如果你希望将所有辅助函数整合到一个地方，或者覆盖默认行为，可以使用 [`noEmitHelpers`](/tsconfig#noEmitHelpers) 来指示编译器不要生成这些辅助函数。

## `--newLine` 命令行选项

默认情况下，在基于 Windows 的系统上生成的换行符是 `\r\n`，而在基于 \*nix 的系统上是 `\n`。
[`newLine`](/tsconfig#newLine) 命令行标志允许覆盖此行为，并指定在生成的输出文件中使用的换行符。

## `--inlineSourceMap` 与 `inlineSources` 命令行选项

[`inlineSourceMap`](/tsconfig#inlineSourceMap) 会让 source map 文件直接内联写入生成的 `.js` 文件中，而不是写入独立的 `.js.map` 文件。
[`inlineSources`](/tsconfig#inlineSources) 则允许进一步将 `.ts` 源文件内容直接内嵌到 `.js` 文件中。
