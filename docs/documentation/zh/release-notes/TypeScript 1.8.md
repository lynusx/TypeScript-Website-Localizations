---
title: TypeScript 1.8
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-1-8.html
oneline: TypeScript 1.8 发布说明
---

## 类型参数作为约束

在 TypeScript 1.8 中，类型参数约束可以引用同一个类型参数列表中的类型参数。
此前这会被报告为错误。
该能力通常被称为 [F-Bounded 多态（F-Bounded Polymorphism）](https://wikipedia.org/wiki/Bounded_quantification#F-bounded_quantification)。

##### 示例

```ts
function assign<T extends U, U>(target: T, source: U): T {
  for (let id in source) {
    target[id] = source[id]
  }
  return target
}

let x = { a: 1, b: 2, c: 3, d: 4 }
assign(x, { b: 10, d: 20 })
assign(x, { e: 0 }) // 错误
```

## 控制流分析错误

TypeScript 1.8 引入了控制流分析，帮助捕获开发者常犯的错误。
请继续阅读以了解更多细节，并查看这些错误的实际演示：

![cfa](https://cloud.githubusercontent.com/assets/8052307/5210657/c5ae0f28-7585-11e4-97d8-86169ef2a160.gif)

### 无法执行的代码（Unreachable code）

在运行时确定永远不会执行的语句现在会被正确标记为不可达代码（unreachable code）错误。
例如，跟在无条件的 `return`、`throw`、`break` 或 `continue` 语句之后的语句均被视为不可达。
可以使用 [`allowUnreachableCode`](/tsconfig#allowUnreachableCode) 来禁用不可达代码的检测与报错。

##### 示例

下面是一个不可达代码错误的简单示例：

```ts
function f(x) {
  if (x) {
    return true
  } else {
    return false
  }

  x = 0 // 错误：检测到无法访问的代码。
}
```

该特性捕获的更常见错误是在 `return` 语句之后添加换行：

```ts
function f() {
  return // 在换行处触发了自动分号插入（ASI）
  {
    x: 'string' // 错误：检测到无法访问的代码。
  }
}
```

由于 JavaScript 会在行尾自动插入分号（ASI）来终结 `return` 语句，因此后面的对象字面量变成了一个独立的代码块。

### 未使用的标签（Unused labels）

未使用的标签也会被标记报错。
与不可达代码检查一样，这些检查默认开启；
可以使用 [`allowUnusedLabels`](/tsconfig#allowUnusedLabels) 来停止报告此类错误。

##### 示例

```ts
loop: while (x > 0) {
  // 错误：未使用的标签。
  x++
}
```

### 隐式返回（Implicit returns）

在 JavaScript 中，若函数存在未返回值的代码路径，则会隐式返回 `undefined`。
现在编译器可以将这些情况标记为隐式返回错误。
该检查默认_关闭_；可以使用 [`noImplicitReturns`](/tsconfig#noImplicitReturns) 将其开启。

##### 示例

```ts
function f(x) {
  // 错误：并非所有代码路径都有返回值。
  if (x) {
    return false
  }

  // 隐式返回 `undefined`
}
```

### case 子句落空贯穿（Case clause fall-throughs）

当 switch 语句中非空的 case 子句发生落空贯穿（fall-through）时，TypeScript 可以对其报告错误。
该检查默认_关闭_，可以通过 [`noFallthroughCasesInSwitch`](/tsconfig#noFallthroughCasesInSwitch) 启用。

##### 示例

在开启 [`noFallthroughCasesInSwitch`](/tsconfig#noFallthroughCasesInSwitch) 时，以下示例将触发错误：

```ts
switch (x % 2) {
  case 0: // 错误：switch 中存在贯穿（Fallthrough）case。
    console.log('even')

  case 1:
    console.log('odd')
    break
}
```

但在以下示例中不会报告错误，因为贯穿的 case 子句为空：

```ts
switch (x % 3) {
  case 0:
  case 1:
    console.log('Acceptable')
    break

  case 2:
    console.log('This is *two much*!')
    break
}
```

## React 中的函数组件（Function Components）

TypeScript 现在支持[函数组件（Function components）](https://reactjs.org/docs/components-and-props.html#functional-and-class-components)。
这是一种能够轻松组合其他组件的轻量级组件：

```ts
// 使用参数解构和默认值轻松定义 'props' 类型
const Greeter = ({ name = "world" }) => <div>Hello, {name}!</div>;

// 属性会进行验证
let example = <Greeter name="TypeScript 1.8" />;
```

为了使用此功能以及简化的 props，请务必使用[最新版本的 react.d.ts](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/index.d.ts)。

## 简化 React 中的 `props` 类型管理

在 TypeScript 1.8 中，配合最新版本的 react.d.ts（参见上文），我们还大幅简化了 `props` 类型的声明。

具体而言：

- 不再需要显式声明 `ref` 和 `key`，也不需要 `extend React.Props`
- `ref` 和 `key` 属性将在所有组件上以正确的类型出现
- 无状态函数式组件（Stateless Function components）的实例上会正确禁用 `ref` 属性

## 从模块中扩充全局/模块作用域

现在，用户可以对现有模块声明自己需要的扩充增强，或者声明其他使用者已经做出的扩充。
模块补充增强（Module augmentation）的形式与传统的外部环境模块声明相同（即 `declare module "foo" { }` 语法），可以直接嵌套在你的模块内部，或者嵌套在另一个顶层外部环境模块中。

此外，TypeScript 还引入了形如 `declare global { }` 的_全局_补充增强机制。
这允许模块在必要时扩充像 `Array` 这样的全局类型。

模块补充增强的名称解析遵循与 `import` 和 `export` 声明中的模块标识符完全相同的规则。
模块补充增强中的声明会与现有的声明合并，就像它们在同一个文件中声明一样。

无论是模块补充增强还是全局补充增强，都不能向顶层作用域添加全新的实体——它们只能对现有的声明进行“打补丁”（patch）。

##### 示例

在下面的代码中，`map.ts` 可以声明它在内部为来自 `observable.ts` 的 `Observable` 类型打补丁，并为其添加 `map` 方法：

```ts
// observable.ts
export class Observable<T> {
  // ...
}
```

```ts
// map.ts
import { Observable } from "./observable";

// 为 "./observable" 创建扩充
declare module "./observable" {

    // 通过接口合并扩充 'Observable' 类定义
    interface Observable<T> {
        map<U>(proj: (el: T) => U): Observable<U>;
    }

}

Observable.prototype.map = /*...*/;
```

```ts
// consumer.ts
import { Observable } from './observable'
import './map'

let o: Observable<number>
o.map((x) => x.toFixed())
```

类似地，也可以在模块中使用 `declare global` 声明来扩充全局作用域：

##### 示例

```ts
// 确保此文件被视为模块。
export {}

declare global {
  interface Array<T> {
    mapToNumbers(): number[]
  }
}

Array.prototype.mapToNumbers = function () {
  /* ... */
}
```

## 字符串字面量类型（String literal types）

API 期望某些特定字符串集合作为参数值的情况非常普遍。
例如，考虑一个 UI 库，它可以在屏幕上移动元素，同时控制动画的[“缓动”（easing）](https://wikipedia.org/wiki/Inbetweening)：

```ts
declare class UIElement {
  animate(options: AnimationOptions): void
}

interface AnimationOptions {
  deltaX: number
  deltaY: number
  easing: string // 可以是 "ease-in"、"ease-out"、"ease-in-out"
}
```

然而这种做法很容易出错——无法阻止用户不小心拼错合法的缓动值：

```ts
// 无错误
new UIElement().animate({ deltaX: 100, deltaY: 100, easing: 'ease-inout' })
```

在 TypeScript 1.8 中，我们引入了字符串字面量类型。
这些类型的书写方式与字符串字面量相同，但出现在类型位置。

开发者现在可以确保类型系统能够捕获此类错误。
以下是使用字符串字面量类型重写的 `AnimationOptions`：

```ts
interface AnimationOptions {
  deltaX: number
  deltaY: number
  easing: 'ease-in' | 'ease-out' | 'ease-in-out'
}

// 错误：类型 '"ease-inout"' 不能赋给类型 '"ease-in" | "ease-out" | "ease-in-out"'
new UIElement().animate({ deltaX: 100, deltaY: 100, easing: 'ease-inout' })
```

## 联合/交叉类型的类型推断增强

TypeScript 1.8 改进了在源类型和目标类型均为联合类型或交叉类型时的类型推断。
例如，当从 `string | string[]` 推断到 `string | T` 时，我们会将类型消减为 `string[]` 和 `T`，从而将 `T` 推断为 `string[]`。

##### 示例

```ts
type Maybe<T> = T | void

function isDefined<T>(x: Maybe<T>): x is T {
  return x !== undefined && x !== null
}

function isUndefined<T>(x: Maybe<T>): x is void {
  return x === undefined || x === null
}

function getOrElse<T>(x: Maybe<T>, defaultValue: T): T {
  return isDefined(x) ? x : defaultValue
}

function test1(x: Maybe<string>) {
  let x1 = getOrElse(x, 'Undefined') // string
  let x2 = isDefined(x) ? x : 'Undefined' // string
  let x3 = isUndefined(x) ? 'Undefined' : x // string
}

function test2(x: Maybe<number>) {
  let x1 = getOrElse(x, -1) // number
  let x2 = isDefined(x) ? x : -1 // number
  let x3 = isUndefined(x) ? -1 : x // number
}
```

## 使用 `--outFile` 拼合打包 `AMD` 和 `System` 模块

将 [`outFile`](/tsconfig#outFile) 与 `--module amd` 或 `--module system` 结合指定时，会将编译中的所有模块拼合打包到一个包含多个模块闭包的单一输出文件中。

每个模块的模块名称将根据其相对于 [`rootDir`](/tsconfig#rootDir) 的相对路径进行计算。

##### 示例

```ts
// 文件 src/a.ts
import * as B from './lib/b'
export function createA() {
  return B.createB()
}
```

```ts
// 文件 src/lib/b.ts
export function createB() {
  return {}
}
```

编译输出如下：

```js
define('lib/b', ['require', 'exports'], function (require, exports) {
  'use strict'
  function createB() {
    return {}
  }
  exports.createB = createB
})
define('a', ['require', 'exports', 'lib/b'], function (require, exports, B) {
  'use strict'
  function createA() {
    return B.createB()
  }
  exports.createA = createA
})
```

## 支持与 SystemJS 的 `default` 导入互操作

像 SystemJS 这样的模块加载器会包装 CommonJS 模块并将其作为 `default` ES6 导入公开。这导致无法在模块的 SystemJS 和 CommonJS 实现之间共享定义文件，因为根据加载器的不同，模块的形态看起来有所差异。

设置新的编译器标志 [`allowSyntheticDefaultImports`](/tsconfig#allowSyntheticDefaultImports) 表明模块加载器会执行某种合成默认导入成员的创建操作，而该操作在导入的 .ts 或 .d.ts 中未作说明。编译器将推断存在一个具有整个模块本身形态的 `default` 导出。

System 模块默认开启此标志。

## 允许在循环中捕获 `let`/`const`

此前这会被视为错误，而在 TypeScript 1.8 中已得到支持。
在循环内部声明并被函数捕获的 `let`/`const` 声明，现在在编译输出时能够正确符合 `let`/`const` 的新鲜度语义（freshness semantics）。

##### 示例

```ts
let list = []
for (let i = 0; i < 5; i++) {
  list.push(() => i)
}

list.forEach((f) => console.log(f()))
```

编译输出为：

```js
var list = []
var _loop_1 = function (i) {
  list.push(function () {
    return i
  })
}
for (var i = 0; i < 5; i++) {
  _loop_1(i)
}
list.forEach(function (f) {
  return console.log(f())
})
```

运行结果为：

```cmd
0
1
2
3
4
```

## 改进对 `for..in` 语句的检查

此前，`for..in` 变量的类型会被推断为 `any`；这导致编译器会忽略 `for..in` 循环体内的无效使用。

从 TypeScript 1.8 开始：

- 在 `for..in` 语句中声明的变量类型隐式为 `string`。
- 当一个带有 `T` 类型的数字索引签名的对象（例如数组），被所在 `for..in` 语句中带有数字索引签名且_不带_字符串索引签名的对象的 `for..in` 变量进行索引时（同样例如数组），所产生的值类型为 `T`。

##### 示例

```ts
var a: MyObject[]
for (var x in a) {
  // x 的类型隐式为 string
  var obj = a[x] // obj 的类型为 MyObject
}
```

## 生成的模块现在包含 `"use strict";` 序言

根据 ES6 规范，模块始终在严格模式下进行解析，但对于非 ES6 目标，生成代码中并未遵循这一点。从 TypeScript 1.8 开始，生成的模块始终处于严格模式。对于大多数代码而言这应该不会产生任何可见的变化，因为 TypeScript 在编译期就已经将大多数严格模式错误视为编译错误；但这意味着在以前的 TS 代码中某些在运行时静默失败的操作（例如对 `NaN` 进行赋值），现在将会直接抛出异常。有关严格模式与非严格模式差异的详细列表，请参阅关于严格模式的 [MDN 文章](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Strict_mode)。

## 允许通过 `--allowJs` 包含 JavaScript 文件

项目中经常会包含非 TypeScript 编写的外部源文件。
或者，你可能正处于将 JS 代码库转换为 TS 的过渡阶段，但仍希望将所有 JS 代码与新 TS 代码的输出打包合并为一个单一文件。

现在允许将 `.js` 文件作为 `tsc` 的输入。
TypeScript 编译器会检查输入的 `.js` 文件是否存在语法错误，并根据 [`target`](/tsconfig#target) 和 [`module`](/tsconfig#module) 标志生成合法的输出。
输出结果也可以与其他 `.ts` 文件结合在一起。
同 `.ts` 文件一样，编译器也会为 `.js` 文件生成 source map。

## 使用 `--reactNamespace` 自定义 JSX 工厂

将 `--reactNamespace <JSX 工厂名称>` 与 `--jsx react` 一起传递，允许使用不同于默认 `React` 的 JSX 工厂。

新的工厂名称将用于调用 `createElement` 和 `__spread` 函数。

##### 示例

```ts
import { jsxFactory } from "jsxFactory";

var div = <div>Hello JSX!</div>;
```

使用以下命令编译：

```shell
tsc --jsx react --reactNamespace jsxFactory --m commonJS
```

编译输出如下：

```js
'use strict'
var jsxFactory_1 = require('jsxFactory')
var div = jsxFactory_1.jsxFactory.createElement('div', null, 'Hello JSX!')
```

## 基于 `this` 的类型守卫

TypeScript 1.8 将[用户自定义类型守卫函数](./typescript-1.6.html#user-defined-type-guard-functions)扩展到了类和接口的方法中。

`this is T` 现在是类和接口中方法的有效返回类型注解。
当用于类型收窄位置（例如 `if` 语句）时，调用表达式目标对象的类型将被收窄为 `T`。

##### 示例

```ts
class FileSystemObject {
  isFile(): this is File {
    return this instanceof File
  }
  isDirectory(): this is Directory {
    return this instanceof Directory
  }
  isNetworked(): this is Networked & this {
    return this.networked
  }
  constructor(
    public path: string,
    private networked: boolean,
  ) {}
}

class File extends FileSystemObject {
  constructor(
    path: string,
    public content: string,
  ) {
    super(path, false)
  }
}
class Directory extends FileSystemObject {
  children: FileSystemObject[]
}
interface Networked {
  host: string
}

let fso: FileSystemObject = new File('foo/bar.txt', 'foo')
if (fso.isFile()) {
  fso.content // fso 为 File
} else if (fso.isDirectory()) {
  fso.children // fso 为 Directory
} else if (fso.isNetworked()) {
  fso.host // fso 具有网络属性
}
```

## 官方 TypeScript NuGet 软件包

从 TypeScript 1.8 开始，TypeScript 编译器（`tsc.exe`）以及 MSBuild 集成组件（`Microsoft.TypeScript.targets` 和 `Microsoft.TypeScript.Tasks.dll`）均已提供官方 NuGet 软件包。

稳定版本软件包可在此处获取：

- [Microsoft.TypeScript.Compiler](https://www.nuget.org/packages/Microsoft.TypeScript.Compiler/)
- [Microsoft.TypeScript.MSBuild](https://www.nuget.org/packages/Microsoft.TypeScript.MSBuild/)

此外，匹配 [npm 每夜构建版本（nightly npm package）](http://blogs.msdn.com/b/typescript/archive/2015/07/27/introducing-typescript-nightlies.aspx)的每夜构建 NuGet 软件包也可以在 [myget](https://myget.org) 上获取：

- [TypeScript-Preview](https://www.myget.org/gallery/typescript-preview)

## 更美观的 `tsc` 错误消息输出

我们深知大量单色文本输出会让眼睛感到疲劳。
色彩有助于识别消息的起始与结束位置，当错误输出非常庞大时，这些视觉线索尤为重要。

只需传递 [`pretty`](/tsconfig#pretty) 命令行选项，TypeScript 就会提供更加丰富多彩的输出，并附带出错位置的相关上下文信息。

![Showing off pretty error messages in ConEmu](https://raw.githubusercontent.com/wiki/Microsoft/TypeScript/images/new-in-typescript/pretty01.png)

## VS 2015 中 JSX 代码的高亮着色

在 TypeScript 1.8 中，Visual Studio 2015 现已支持对 JSX 标签进行分类与语法高亮着色。

![jsx](https://cloud.githubusercontent.com/assets/8052307/12271404/b875c502-b90f-11e5-93d8-c6740be354d1.png)

通过 `工具（Tools）` -> `选项（Options）` -> `环境（Environment）` -> `字体和颜色（Fonts and Colors）` 页面，修改 `VB XML` 的颜色与字体设置，可以进一步自定义此类分类的样式。

## `--project`（`-p`）参数现在可以接受任意文件路径

`--project` 命令行选项原本只能接受包含 `tsconfig.json` 的文件夹路径。
鉴于构建配置的不同使用场景，允许 `--project` 指向任何其他兼容的 JSON 文件是十分合理的。
例如，用户可能希望针对 Node 5 将编译目标设为 ES2015 并采用 CommonJS 模块，而针对浏览器将编译目标设为 ES5 并采用 AMD 模块。
借助这项新改进，用户仅需使用 `tsc` 即可轻松管理两个独立的构建目标，而无需采用将 `tsconfig.json` 文件放在不同目录等变通方法。

如果传入的是目录，旧行为依然保持不变——编译器会尝试在该目录中查找名为 `tsconfig.json` 的文件。

## 支持在 tsconfig.json 中编写注释

能够在配置文件中添加说明文档总是一件乐事！
`tsconfig.json` 现在支持单行和多行注释。

```json tsconfig
{
  "compilerOptions": {
    "target": "ES2015", // 运行在 node v5 上，好耶！
    "sourceMap": true // 让调试更轻松
  },
  /*
   * 排除的文件
   */
  "exclude": ["file.d.ts"]
}
```

## 支持输出到 IPC 驱动的文件

TypeScript 1.8 允许用户将 [`outFile`](/tsconfig#outFile) 参数与特殊文件系统实体（如命名管道、设备文件等）配合使用。

例如，在许多类 Unix 系统上，可以通过文件 `/dev/stdout` 访问标准输出流。

```shell
tsc foo.ts --outFile /dev/stdout
```

这也可以用于在命令之间通过管道传递输出。

例如，我们可以将生成的 JavaScript 管道传输到像 [pretty-js](https://www.npmjs.com/package/pretty-js) 这样的美化输出工具中：

```shell
tsc foo.ts --outFile /dev/stdout | pretty-js
```

## 改进 Visual Studio 2015 对 `tsconfig.json` 的支持

TypeScript 1.8 允许在所有项目类型中使用 `tsconfig.json` 文件。
这包括 ASP.NET v4 项目、_控制台应用程序（Console Application）_ 以及 _带有 TypeScript 的 Html 应用程序（Html Application with TypeScript）_ 项目类型。
此外，你不再局限于单个 `tsconfig.json` 文件，而是可以添加多个，每个文件都会作为项目的一部分进行构建。
这使你可以为应用程序的不同部分拆分配置，而无需创建多个不同的项目。

![Showing off tsconfig.json in Visual Studio](https://raw.githubusercontent.com/wiki/Microsoft/TypeScript/images/new-in-typescript/tsconfig-in-vs.png)

当你添加 `tsconfig.json` 文件后，我们还会禁用项目属性页面。
这意味着所有配置更改都必须直接在 `tsconfig.json` 文件本身中进行。

### 一些限制说明

- 如果添加了 `tsconfig.json` 文件，则不属于该上下文的 TypeScript 文件将不会被编译。
- Apache Cordova 应用仍然存在现有的一项限制：只能拥有单个 `tsconfig.json` 文件，且该文件必须位于根目录或 `scripts` 文件夹中。
- 大多数项目类型中并未提供 `tsconfig.json` 的项目模板。
