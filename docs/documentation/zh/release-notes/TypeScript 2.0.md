---
title: TypeScript 2.0
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-2-0.html
oneline: TypeScript 2.0 发布说明
---

## null 与 undefined 感知类型

TypeScript 有两种特殊类型：Null 和 Undefined，分别对应值 `null` 和 `undefined`。以前无法显式地命名这两种类型，但现在不论处于何种类型检查模式，`null` 和 `undefined` 均可作为类型名称使用。

类型检查器以前将 `null` 和 `undefined` 视为可赋值给任意类型的值。实际上，`null` 和 `undefined` 是_每一种_类型的合法值，无法将其从类型中排除（因此也无法检测对它们的错误使用）。

## `--strictNullChecks`

[`strictNullChecks`](/tsconfig#strictNullChecks) 会开启一种新的严格空值检查模式。

在严格空值检查模式下，`null` 和 `undefined` 不再属于每种类型的值域，只能赋值给自身以及 `any`（唯一例外是 `undefined` 也可赋值给 `void`）。因此，在普通类型检查模式下 `T` 与 `T | undefined` 被视为同义词（因为 `undefined` 被认为是任意 `T` 的子类型），而在严格类型检查模式下二者是不同的类型，只有 `T | undefined` 才允许 `undefined` 值。`T` 与 `T | null` 的关系亦同理。

##### 示例

```ts
// 在启用 --strictNullChecks 下编译
let x: number
let y: number | undefined
let z: number | null | undefined
x = 1 // 正常
y = 1 // 正常
z = 1 // 正常
x = undefined // 错误
y = undefined // 正常
z = undefined // 正常
x = null // 错误
y = null // 错误
z = null // 正常
x = y // 错误
x = z // 错误
y = x // 正常
y = z // 错误
z = x // 正常
z = y // 正常
```

## 赋值前使用检查

在严格空值检查模式下，编译器要求对于类型中不包含 `undefined` 的局部变量，每次引用该变量之前，在所有可能的代码路径上都必须有对其的赋值。

##### 示例

```ts
// 在启用 --strictNullChecks 下编译
let x: number
let y: number | null
let z: number | undefined
x // 错误，引用前未进行赋值
y // 错误，引用前未进行赋值
z // 正常
x = 1
y = null
x // 正常
y // 正常
```

编译器通过执行_基于控制流的类型分析_来检查变量是否已确定被赋值。详情见后文。

## 可选参数和可选属性

可选参数和可选属性会自动在其类型中加入 `undefined`，即使其类型注解没有显式包含 `undefined`。例如，以下两种类型是等价的：

```ts
// 在启用 --strictNullChecks 下编译
type T1 = (x?: number) => string // x 的类型为 number | undefined
type T2 = (x?: number | undefined) => string // x 的类型为 number | undefined
```

## 非 null 与非 undefined 类型保护

如果对象或函数的类型包含 `null` 或 `undefined`，则对其进行属性访问或函数调用会产生编译时错误。不过，类型保护已扩展为支持非 null 和非 undefined 检查。

##### 示例

```ts
// 在启用 --strictNullChecks 下编译
declare function f(x: number): string
let x: number | null | undefined
if (x) {
  f(x) // 正常，此处 x 的类型为 number
} else {
  f(x) // 错误，此处 x 的类型为 number?
}
let a = x != null ? f(x) : '' // a 的类型为 string
let b = x && f(x) // b 的类型为 string | 0 | null | undefined
```

非 null 与非 undefined 类型保护可以使用 `==`、`!=`、`===` 或 `!==` 运算符与 `null` 或 `undefined` 进行比较，例如 `x != null` 或 `x === undefined`。对目标变量类型的影响准确反映了 JavaScript 语义（例如，双等号运算符无论指定的是哪个值，都会同时检查两者；而三等号运算符只检查指定的那个值）。

## 类型保护中的点路径名

类型保护以前只支持检查局部变量和参数，现在扩展为支持检查由变量名或参数名后跟一个或多个属性访问所构成的"点路径名"。

##### 示例

```ts
interface Options {
  location?: {
    x?: number
    y?: number
  }
}

function foo(options?: Options) {
  if (options && options.location && options.location.x) {
    const x = options.location.x // x 的类型为 number
  }
}
```

点路径名的类型保护同样适用于用户自定义类型保护函数、`typeof` 和 `instanceof` 运算符，且不依赖于 [`strictNullChecks`](/tsconfig#strictNullChecks) 编译器选项。

对点路径名中任意部分进行赋值后，该路径名的类型保护将失效。例如，对 `x`、`x.y` 或 `x.y.z` 进行赋值后，`x.y.z` 的类型保护便不再生效。

## 表达式运算符

表达式运算符允许操作数类型包含 `null` 和/或 `undefined`，但始终产生非 null、非 undefined 类型的值。

```ts
// 在启用 --strictNullChecks 下编译
function sum(a: number | null, b: number | null) {
  return a + b // 产生类型为 number 的值
}
```

`&&` 运算符会根据左操作数类型中存在的情况，将 `null` 和/或 `undefined` 添加到右操作数的类型中；`||` 运算符则会从结果联合类型中同时移除左操作数类型中的 `null` 和 `undefined`。

```ts
// 在启用 --strictNullChecks 下编译
interface Entity {
  name: string
}
let x: Entity | null
let s = x && x.name // s 的类型为 string | null
let y = x || { name: 'test' } // y 的类型为 Entity
```

## 类型拓宽

在严格空值检查模式下，`null` 和 `undefined` 类型**不会**被拓宽为 `any`。

```ts
let z = null // z 的类型为 null
```

在普通类型检查模式下，`z` 的推断类型因拓宽而为 `any`；但在严格空值检查模式下，`z` 的推断类型为 `null`（因此，在没有类型注解的情况下，`null` 是 `z` 唯一可能的值）。

## 非空断言运算符

新增的后缀表达式运算符 `!` 可用于断言其操作数在类型检查器无法判断该事实的上下文中为非 null 且非 undefined。具体而言，操作 `x!` 会产生一个排除了 `null` 和 `undefined` 的 `x` 类型值。与 `<T>x` 和 `x as T` 形式的类型断言类似，非空断言运算符 `!` 在生成的 JavaScript 代码中会被直接移除。

```ts
// 在启用 --strictNullChecks 下编译
function validateEntity(e?: Entity) {
  // 如果 e 为 null 或无效实体则抛出异常
}

function processEntity(e?: Entity) {
  validateEntity(e)
  let s = e!.name // 断言 e 非空并访问 name
}
```

## 兼容性

这些新特性经过设计，既可在严格空值检查模式下使用，也可在普通类型检查模式下使用。特别是，在普通类型检查模式下，`null` 和 `undefined` 类型会自动从联合类型中消除（因为它们是所有其他类型的子类型），且允许使用 `!` 非空断言表达式运算符，但它在该模式下不产生任何效果。因此，更新后使用了 null/undefined 感知类型的声明文件仍可在普通类型检查模式下使用，以保持向后兼容性。

实际操作中，严格空值检查模式要求编译中的所有文件都具有 null/undefined 感知。

## 基于控制流的类型分析

TypeScript 2.0 为局部变量和参数实现了基于控制流的类型分析。以前，针对类型保护的类型分析仅限于 `if` 语句和 `?:` 条件表达式，不涵盖赋值和 `return`、`break` 等控制流结构的影响。TypeScript 2.0 起，类型检查器会分析语句和表达式中所有可能的控制流路径，从而为声明为联合类型的局部变量或参数在任意位置推导出最具体的类型（即**收窄类型**）。

##### 示例

```ts
function foo(x: string | number | boolean) {
  if (typeof x === 'string') {
    x // 此处 x 的类型为 string
    x = 1
    x // 此处 x 的类型为 number
  }
  x // 此处 x 的类型为 number | boolean
}

function bar(x: string | number) {
  if (typeof x === 'number') {
    return
  }
  x // 此处 x 的类型为 string
}
```

基于控制流的类型分析在 [`strictNullChecks`](/tsconfig#strictNullChecks) 模式下尤为重要，因为可空类型通过联合类型来表示：

```ts
function test(x: string | null) {
  if (x === null) {
    return
  }
  x // 函数剩余部分中 x 的类型为 string
}
```

此外，在 [`strictNullChecks`](/tsconfig#strictNullChecks) 模式下，基于控制流的类型分析还包括对不允许 `undefined` 值的局部变量进行_确定赋值分析_。

```ts
function mumble(check: boolean) {
  let x: number // 类型不允许为 undefined
  x // 错误，x 为 undefined
  if (check) {
    x = 1
    x // 正常
  }
  x // 错误，x 可能为 undefined
  x = 2
  x // 正常
}
```

## 标记联合类型

TypeScript 2.0 实现了对标记联合类型（也称为可辨识联合类型）的支持。具体来说，TS 编译器现在支持基于可辨识属性测试来收窄联合类型的类型保护，并进一步将此能力扩展至 `switch` 语句。

##### 示例

```ts
interface Square {
  kind: 'square'
  size: number
}

interface Rectangle {
  kind: 'rectangle'
  width: number
  height: number
}

interface Circle {
  kind: 'circle'
  radius: number
}

type Shape = Square | Rectangle | Circle

function area(s: Shape) {
  // 在接下来的 switch 语句中，根据判别式属性的值，s 的类型在每个 case 子句中都被收窄，
  // 从而允许在无需类型断言的情况下访问该变体的其他属性。
  switch (s.kind) {
    case 'square':
      return s.size * s.size
    case 'rectangle':
      return s.width * s.height
    case 'circle':
      return Math.PI * s.radius * s.radius
  }
}

function test1(s: Shape) {
  if (s.kind === 'square') {
    s // Square
  } else {
    s // Rectangle | Circle
  }
}

function test2(s: Shape) {
  if (s.kind === 'square' || s.kind === 'rectangle') {
    return
  }
  s // Circle
}
```

**可辨识属性类型保护**是形如 `x.p == v`、`x.p === v`、`x.p != v` 或 `x.p !== v` 的表达式，其中 `p` 是属性名，`v` 是字符串字面量类型或字符串字面量联合类型的表达式。可辨识属性类型保护会将 `x` 的类型收窄为那些拥有可辨识属性 `p` 且该属性值在 `v` 的可能取值范围内的成员类型。

请注意，目前仅支持字符串字面量类型的可辨识属性，未来计划支持布尔和数字字面量类型。

## `never` 类型

TypeScript 2.0 引入了新的原始类型 `never`，用于表示永不出现的值的类型。具体而言，`never` 是永不返回的函数的返回类型，也是在类型保护条件永远为 false 时变量的类型。

`never` 类型具有以下特性：

- `never` 是所有类型的子类型，且可赋值给所有类型。
- 没有任何类型是 `never` 的子类型或可赋值给 `never`（`never` 自身除外）。
- 在没有返回类型注解的函数表达式或箭头函数中，若函数没有 `return` 语句，或仅有类型为 `never` 的 `return` 语句，且控制流分析确定函数末端不可达，则推断返回类型为 `never`。
- 在显式声明返回类型为 `never` 的函数中，所有 `return` 语句（如有）的表达式类型必须为 `never`，且函数末端必须不可达。

由于 `never` 是所有类型的子类型，它在联合类型中始终被省略；只要函数还有其他返回类型，`never` 在函数返回类型推断中也会被忽略。

以下是返回 `never` 的函数示例：

```ts
// 返回 never 的函数必须具有无法到达的终点
function error(message: string): never {
  throw new Error(message)
}

// 推断的返回类型为 never
function fail() {
  return error('Something failed')
}

// 返回 never 的函数必须具有无法到达的终点
function infiniteLoop(): never {
  while (true) {}
}
```

以下是使用返回 `never` 的函数的示例：

```ts
// 推断的返回类型为 number
function move1(direction: 'up' | 'down') {
  switch (direction) {
    case 'up':
      return 1
    case 'down':
      return -1
  }
  return error('Should never get here')
}

// 推断的返回类型为 number
function move2(direction: 'up' | 'down') {
  return direction === 'up'
    ? 1
    : direction === 'down'
      ? -1
      : error('Should never get here')
}

// 推断的返回类型为 T
function check<T>(x: T | undefined) {
  return x || error('Undefined value')
}
```

由于 `never` 可赋值给所有类型，当需要一个返回更具体类型的回调函数时，也可以使用返回 `never` 的函数：

```ts
function test(cb: () => string) {
  let s = cb()
  return s
}

test(() => 'hello')
test(() => fail())
test(() => {
  throw new Error()
})
```

## 只读属性与索引签名

属性或索引签名现在可以用 `readonly` 修饰符声明。

只读属性可以有初始化器，也可以在同一类声明的构造函数中赋值，但其他情况下对只读属性的赋值均不允许。

此外，在以下几种情况下，实体会被**隐式**视为只读：

- 声明了 `get` 访问器但没有 `set` 访问器的属性被视为只读。
- 在枚举对象的类型中，枚举成员被视为只读属性。
- 在模块对象的类型中，导出的 `const` 变量被视为只读属性。
- `import` 语句中声明的实体被视为只读。
- 通过 ES2015 命名空间导入访问的实体被视为只读（例如，当 `foo` 声明为 `import * as foo from "foo"` 时，`foo.x` 是只读的）。

##### 示例

```ts
interface Point {
  readonly x: number
  readonly y: number
}

var p1: Point = { x: 10, y: 20 }
p1.x = 5 // 错误，p1.x 是只读的

var p2 = { x: 1, y: 1 }
var p3: Point = p2 // 正常，p2 的只读别名
p3.x = 5 // 错误，p3.x 是只读的
p2.x = 5 // 正常，但由于别名引用也会修改 p3.x
```

```ts
class Foo {
  readonly a = 1
  readonly b: string
  constructor() {
    this.b = 'hello' // 允许在构造函数中赋值
  }
}
```

```ts
let a: Array<number> = [0, 1, 2, 3, 4]
let b: ReadonlyArray<number> = a
b[5] = 5 // 错误，元素是只读的
b.push(5) // 错误，没有 push 方法（因为会修改数组）
b.length = 3 // 错误，length 是只读的
a = b // 错误，缺少变更方法
```

## 为函数指定 `this` 的类型

在类或接口中指定 `this` 类型的基础上，函数和方法现在也可以声明其期望的 `this` 类型。

函数内部 `this` 的默认类型为 `any`。从 TypeScript 2.0 起，可以提供显式的 `this` 参数。`this` 参数是出现在函数参数列表第一位的虚拟参数：

```ts
function f(this: void) {
  // 确保 `this` 在此独立函数中不可用
}
```

## 回调函数中的 `this` 参数

库也可以使用 `this` 参数来声明回调函数的调用方式。

##### 示例

```ts
interface UIElement {
  addClickListener(onclick: (this: void, e: Event) => void): void
}
```

`this: void` 表示 `addClickListener` 期望 `onclick` 是一个不需要 `this` 类型的函数。

现在，如果你在调用代码中使用 `this` 注解：

```ts
class Handler {
  info: string
  onClickBad(this: Handler, e: Event) {
    // 糟糕，这里使用了 this。使用此回调在运行时会崩溃
    this.info = e.message
  }
}
let h = new Handler()
uiElement.addClickListener(h.onClickBad) // 错误！
```

## `--noImplicitThis`

TypeScript 2.0 还新增了一个标志，用于标记函数中所有没有显式类型注解的 `this` 使用。

## `tsconfig.json` 中的 Glob 支持

Glob 支持终于来了！它是[最受欢迎的功能之一](https://github.com/Microsoft/TypeScript/issues/1927)。

两个属性 [`include`](/tsconfig#include) 和 [`exclude`](/tsconfig#exclude) 支持类 Glob 的文件模式。

##### 示例

```json tsconfig
{
  "compilerOptions": {
    "module": "commonjs",
    "noImplicitAny": true,
    "removeComments": true,
    "preserveConstEnums": true,
    "outFile": "../../built/local/tsc.js",
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "**/*.spec.ts"]
}
```

支持的 Glob 通配符有：

- `*` 匹配零个或多个字符（不含目录分隔符）
- `?` 匹配任意单个字符（不含目录分隔符）
- `**/` 递归匹配任意子目录

若 Glob 模式的某个片段仅包含 `*` 或 `.*`，则只有具有受支持扩展名的文件才会被包含（默认为 `.ts`、`.tsx` 和 `.d.ts`；若 [`allowJs`](/tsconfig#allowJs) 设为 true，还包含 `.js` 和 `.jsx`）。

若 [`files`](/tsconfig#files) 和 [`include`](/tsconfig#include) 均未指定，编译器默认包含所在目录及子目录中所有 TypeScript 文件（`.ts`、`.d.ts` 和 `.tsx`），但通过 [`exclude`](/tsconfig#exclude) 排除的除外。若 [`allowJs`](/tsconfig#allowJs) 设为 true，JS 文件（`.js` 和 `.jsx`）也会被包含。

若指定了 [`files`](/tsconfig#files) 或 [`include`](/tsconfig#include)，编译器则会包含这两个属性所指定文件的并集。[`outDir`](/tsconfig#outDir) 指定目录下的文件始终被排除，除非通过 [`files`](/tsconfig#files) 属性显式包含（即使指定了 [`exclude`](/tsconfig#exclude) 也如此）。

通过 [`include`](/tsconfig#include) 包含的文件可使用 [`exclude`](/tsconfig#exclude) 进行过滤，但通过 [`files`](/tsconfig#files) 显式包含的文件则始终会被包含，不受 [`exclude`](/tsconfig#exclude) 影响。未指定时，[`exclude`](/tsconfig#exclude) 属性默认排除 `node_modules`、`bower_components` 和 `jspm_packages` 目录。

## 模块解析增强：baseUrl、路径映射、rootDirs 与追踪

TypeScript 2.0 提供了一组额外的模块解析选项，用于_告知_编译器在何处查找给定模块的声明。

详情请参阅[模块解析](http://www.typescriptlang.org/docs/handbook/module-resolution.html)文档。

## Base URL

在使用 AMD 模块加载器的应用中，使用 [`baseUrl`](/tsconfig#baseUrl) 是一种常见实践——模块在运行时会被"部署"到单个文件夹中。所有使用裸说明符名称的模块导入均被视为相对于 [`baseUrl`](/tsconfig#baseUrl)。

##### 示例

```json tsconfig
{
  "compilerOptions": {
    "baseUrl": "./modules"
  }
}
```

此后，对 `"moduleA"` 的导入会在 `./modules/moduleA` 中查找：

```ts
import A from 'moduleA'
```

## 路径映射

有时模块并不直接位于 _baseUrl_ 下。加载器使用映射配置将模块名在运行时映射到文件，参见 [RequireJS 文档](http://requirejs.org/docs/api.html#config-paths)和 [SystemJS 文档](https://github.com/systemjs/systemjs/blob/main/docs/import-maps.md)。

TypeScript 编译器支持通过 `tsconfig.json` 中的 [`paths`](/tsconfig#paths) 属性声明此类映射。

##### 示例

例如，对模块 `"jquery"` 的导入在运行时会被转换为 `"node_modules/jquery/dist/jquery.slim.min.js"`：

```json tsconfig
{
  "compilerOptions": {
    "baseUrl": "./node_modules",
    "paths": {
      "jquery": ["jquery/dist/jquery.slim.min"]
    }
}
```

使用 [`paths`](/tsconfig#paths) 还可以实现更复杂的映射，包括多个回退路径。例如，某些模块只在一处可用，其余模块在另一处，就可以针对这种项目配置进行处理。

## 使用 `rootDirs` 的虚拟目录

使用 `rootDirs`，你可以告知编译器构成这个"虚拟"目录的各个_根目录_；编译器便可以将这些"虚拟"目录中的相对模块导入解析为_仿佛_所有根目录已合并到一个目录中的效果。

##### 示例

给定以下项目结构：

```tree
 src
 └── views
     └── view1.ts (imports './template1')
     └── view2.ts

 generated
 └── templates
         └── views
             └── template1.ts (imports './view2')
```

构建步骤会将 `/src/views` 和 `/generated/templates/views` 中的文件复制到输出的同一目录下。在运行时，视图可以预期其模板文件与自身同目录，因此应使用相对路径 `"./template"` 导入模板。

[`rootDirs`](/tsconfig#rootDirs) 指定了一组_根目录_，其内容在运行时预期会合并。因此，依照上例，`tsconfig.json` 文件应如下所示：

```json tsconfig
{
  "compilerOptions": {
    "rootDirs": ["src/views", "generated/templates/views"]
  }
}
```

## 追踪模块解析

[`traceResolution`](/tsconfig#traceResolution) 提供了一种便捷方式，可了解编译器如何解析模块。

```shell
tsc --traceResolution
```

## 简写外围模块声明

如果你不想在使用新模块之前花时间编写声明，现在可以直接使用简写声明快速上手。

##### declarations.d.ts

```ts
declare module 'hot-new-module'
```

来自简写模块的所有导入类型均为 `any`。

```ts
import x, { y } from 'hot-new-module'
x(y)
```

## 模块名中的通配符

以前，使用模块加载器扩展（如 [AMD](https://github.com/amdjs/amdjs-api/blob/master/LoaderPlugins.md) 或 [SystemJS](https://github.com/systemjs/systemjs/blob/main/docs/module-types.md)）导入非代码资源并不方便，以往需要为每个资源单独定义外围模块声明。

TypeScript 2.0 支持使用通配符（`*`）声明一"族"模块名，这样每个扩展只需声明一次，而无需为每个资源都声明。

##### 示例

```ts
declare module '*!text' {
  const content: string
  export default content
}
// 有些则是相反的方式。
declare module 'json!*' {
  const value: any
  export default value
}
```

现在你可以导入匹配 `"*!text"` 或 `"json!*"` 的内容：

```ts
import fileContent from './xyz.txt!text'
import data from 'json!http://example.com/data.json'
console.log(data, fileContent)
```

从无类型代码库迁移时，通配符模块名尤为有用。结合简写外围模块声明，一组模块可以轻松声明为 `any`。

##### 示例

```ts
declare module 'myLibrary/*'
```

编译器将把对 `myLibrary` 下所有模块的导入视为 `any` 类型，从而关闭对这些模块形状和类型的所有检查。

```ts
import { readFile } from "myLibrary/fileSystem/readFile`;

readFile(); // readFile 的类型为 'any'
```

## 支持 UMD 模块定义

某些库设计为可在多种模块加载器中使用，或无需模块加载器（作为全局变量）使用。这类库被称为 [UMD](https://github.com/umdjs/umd) 或[同构](http://isomorphic.net)模块，可通过导入或全局变量两种方式访问。

例如：

##### math-lib.d.ts

```ts
export const isPrime(x: number): boolean;
export as namespace mathLib;
```

该库可作为导入在模块内部使用：

```ts
import { isPrime } from 'math-lib'
isPrime(2)
mathLib.isPrime(2) // 错误：无法在模块内部使用全局定义
```

也可以作为全局变量使用，但只能在脚本中使用（脚本是没有导入或导出的文件）。

```ts
mathLib.isPrime(2)
```

## 可选类属性

类中现在可以声明可选属性和方法，与接口中已允许的写法类似。

##### 示例

```ts
class Bar {
  a: number
  b?: number
  f() {
    return 1
  }
  g?(): number // 可选方法的方法体可以省略
  h?() {
    return 2
  }
}
```

在 [`strictNullChecks`](/tsconfig#strictNullChecks) 模式下编译时，可选属性和方法的类型中会自动包含 `undefined`。因此，上例中 `b` 属性的类型为 `number | undefined`，`g` 方法的类型为 `(() => number) | undefined`。可以使用类型保护来排除类型中的 `undefined` 部分：

```ts
function test(x: Bar) {
  x.a // number
  x.b // number | undefined
  x.f // () => number
  x.g // (() => number) | undefined
  let f1 = x.f() // number
  let g1 = x.g && x.g() // number | undefined
  let g2 = x.g ? x.g() : 0 // number
}
```

## 私有和受保护的构造函数

类的构造函数可以标记为 `private` 或 `protected`。带有私有构造函数的类不能在类体外部实例化，也不能被继承。带有受保护构造函数的类不能在类体外部实例化，但可以被继承。

##### 示例

```ts
class Singleton {
  private static instance: Singleton

  private constructor() {}

  static getInstance() {
    if (!Singleton.instance) {
      Singleton.instance = new Singleton()
    }
    return Singleton.instance
  }
}

let e = new Singleton() // 错误：'Singleton' 的构造函数是私有的。
let v = Singleton.getInstance()
```

## 抽象属性与访问器

抽象类可以声明抽象属性和/或访问器。所有子类都需要声明这些抽象属性，否则必须标记为抽象类。抽象属性不能有初始化器，抽象访问器不能有方法体。

##### 示例

```ts
abstract class Base {
  abstract name: string
  abstract get value()
  abstract set value(v: number)
}

class Derived extends Base {
  name = 'derived'

  value = 1
}
```

## 隐式索引签名

如果对象字面量类型中所有已知属性都可赋值给某个索引签名，则该对象字面量类型现在可赋值给具有该索引签名的类型。这使得将用对象字面量初始化的变量传递给期望 map 或字典的函数成为可能：

```ts
function httpService(path: string, headers: { [x: string]: string }) {}

const headers = {
  'Content-Type': 'application/x-www-form-urlencoded',
}

httpService('', { 'Content-Type': 'application/x-www-form-urlencoded' }) // 正常
httpService('', headers) // 现在正常，以前不支持
```

## 使用 `--lib` 引入内置类型声明

以前，ES6/ES2015 内置 API 声明只能在 `target: ES6` 时使用。引入 [`lib`](/tsconfig#lib) 后，你可以指定一组内置 API 声明分组，并选择将其包含在项目中。例如，如果你的运行时支持 `Map`、`Set` 和 `Promise`（如当今大多数常青浏览器），只需包含 `--lib es2015.collection,es2015.promise` 即可。同样，你也可以排除不需要的声明，例如，在 Node.js 项目中使用 `--lib es5,es6` 来排除 DOM 声明。

以下是可用的 API 分组列表：

- dom
- webworker
- es5
- es6 / es2015
- es2015.core
- es2015.collection
- es2015.iterable
- es2015.promise
- es2015.proxy
- es2015.reflect
- es2015.generator
- es2015.symbol
- es2015.symbol.wellknown
- es2016
- es2016.array.include
- es2017
- es2017.object
- es2017.sharedmemory
- scripthost

##### 示例

```bash
tsc --target es5 --lib es5,es2015.promise
```

```json tsconfig
"compilerOptions": {
    "lib": ["es5", "es2015.promise"]
}
```

## 使用 `--noUnusedParameters` 和 `--noUnusedLocals` 标记未使用的声明

TypeScript 2.0 新增了两个标志，帮助你保持代码库整洁。[`noUnusedParameters`](/tsconfig#noUnusedParameters) 会将所有未使用的函数或方法参数标记为错误；[`noUnusedLocals`](/tsconfig#noUnusedLocals) 会将所有未使用的局部（未导出）声明（如变量、函数、类、导入等）标记为错误。此外，在 [`noUnusedLocals`](/tsconfig#noUnusedLocals) 下，类的未使用私有成员也会被标记为错误。

##### 示例

```ts
import B, { readFile } from './b'
//     ^ 错误：声明了 `B` 但从未使用
readFile()

export function write(message: string, args: string[]) {
  //                                 ^^^^  错误：声明了 'arg' 但从未使用。
  console.log(message)
}
```

名称以 `_` 开头的参数声明不受未使用参数检查的影响，例如：

```ts
function returnNull(_a) {
  // 正常
  return null
}
```

## 模块标识符允许使用 `.js` 扩展名

在 TypeScript 2.0 之前，模块标识符始终被假定为不带扩展名；例如，对于 `import d from "./moduleA.js"` 这样的导入，编译器会在 `./moduleA.js.ts` 或 `./moduleA.js.d.ts` 中查找 `"moduleA.js"` 的定义，这给使用 [SystemJS](https://github.com/systemjs/systemjs) 等在模块标识符中使用 URI 的打包/加载工具带来了困难。

TypeScript 2.0 起，编译器会在 `./moduleA.ts` 或 `./moduleA.d.t` 中查找 `"moduleA.js"` 的定义。

## 支持 `target: es5` 与 `module: es6` 组合

以前此组合被视为无效标志，现在 `target: es5` 与 `module: es6` 已受到支持。这有助于使用基于 ES2015 的 tree shaker，例如 [rollup](https://github.com/rollup/rollup)。

## 函数参数和实参列表中的尾随逗号

函数参数和实参列表中现在允许使用尾随逗号。这是对 [Stage-3 ECMAScript 提案](https://jeffmo.github.io/es-trailing-function-commas/)的实现，可向下编译为合法的 ES3/ES5/ES6。

##### 示例

```ts
function foo(
  bar: Bar,
  baz: Baz, // 参数列表中允许尾随逗号
) {
  // 实现...
}

foo(
  bar,
  baz, // 实参列表中同样允许
)
```

## 新增 `--skipLibCheck`

TypeScript 2.0 新增了 [`skipLibCheck`](/tsconfig#skipLibCheck) 编译器选项，可跳过对声明文件（扩展名为 `.d.ts` 的文件）的类型检查。当程序包含大型声明文件时，编译器会花费大量时间对已知不含错误的声明进行类型检查，跳过声明文件类型检查可以显著缩短编译时间。

由于一个文件中的声明可能影响其他文件中的类型检查，指定 [`skipLibCheck`](/tsconfig#skipLibCheck) 时可能无法检测到某些错误。例如，若非声明文件对声明文件中声明的类型进行了扩展，则仅在检查声明文件时才会报告相关错误；但在实践中，这种情况极为罕见。

## 允许跨声明的重复标识符

这一直是重复定义错误的常见来源——多个声明文件对同一接口成员进行了定义。

TypeScript 2.0 放宽了这一限制，允许跨代码块的重复标识符，前提是它们具有_完全相同_的类型。同一代码块内的重复定义仍不允许。

##### 示例

```ts
interface Error {
  stack?: string
}

interface Error {
  code?: string
  path?: string
  stack?: string // 正常
}
```

## 新增 `--declarationDir`

[`declarationDir`](/tsconfig#declarationDir) 允许将声明文件生成到与 JavaScript 文件不同的位置。
