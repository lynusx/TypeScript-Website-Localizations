---
title: TypeScript 2.7
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-2-7.html
oneline: TypeScript 2.7 发布说明
---

## 常量命名属性

TypeScript 2.7 新增了对在类型上声明常量命名属性的支持，包括 ECMAScript symbol。

##### 示例

```ts
// Lib
export const SERIALIZE = Symbol('serialize-method-key')

export interface Serializable {
  [SERIALIZE](obj: {}): string
}
```

```ts
// consumer

import { SERIALIZE, Serializable } from 'lib'

class JSONSerializableItem implements Serializable {
  [SERIALIZE](obj: {}) {
    return JSON.stringify(obj)
  }
}
```

该特性同样适用于数字字面量和字符串字面量。

##### 示例

```ts
const Foo = 'Foo'
const Bar = 'Bar'

let x = {
  [Foo]: 100,
  [Bar]: 'hello',
}

let a = x[Foo] // has type 'number'
let b = x[Bar] // has type 'string'
```

## `unique symbol`

为了将 symbol 视为独特的字面量类型，TypeScript 引入了新的 `unique symbol` 类型。
`unique symbol` 是 `symbol` 的子类型，只能通过调用 `Symbol()` 或 `Symbol.for()`，或通过显式类型注解来产生。
该新类型仅允许用于 `const` 声明和 `readonly static` 属性上。若要引用某个特定的 unique symbol，必须使用 `typeof` 运算符。
每个 `unique symbol` 的引用都代表与特定声明绑定的完全唯一的标识。

##### 示例

```ts
// Works
declare const Foo: unique symbol

// Error! 'Bar' isn't a constant.
let Bar: unique symbol = Symbol()

// Works - refers to a unique symbol, but its identity is tied to 'Foo'.
let Baz: typeof Foo = Foo

// Also works.
class C {
  static readonly StaticSymbol: unique symbol = Symbol()
}
```

由于每个 `unique symbol` 都有完全独立的标识，两个 `unique symbol` 类型之间不可相互赋值或比较。

##### 示例

```ts
const Foo = Symbol()
const Bar = Symbol()

// Error: can't compare two unique symbols.
if (Foo === Bar) {
  // ...
}
```

## 严格的类初始化检查

TypeScript 2.7 引入了一个名为 [`strictPropertyInitialization`](/tsconfig#strictPropertyInitialization) 的新标志。
该标志会检查类的每个实例属性是否在构造函数体中或通过属性初始化器完成了初始化。
例如：

```ts
class C {
  foo: number
  bar = 'hello'
  baz: boolean
  //  ~~~
  //  Error! Property 'baz' has no initializer and is not definitely assigned in the
  //         constructor.

  constructor() {
    this.foo = 42
  }
}
```

在上面的例子中，如果我们确实希望 `baz` 可以是 `undefined`，应将其声明为 `boolean | undefined` 类型。

某些情况下，属性可能通过辅助方法或依赖注入库间接完成初始化，此时可以对属性使用新的**确定性赋值断言修饰符**（见下文）。

```ts
class C {
  foo!: number
  // ^
  // Notice this '!' modifier.
  // This is the "definite assignment assertion"

  constructor() {
    this.initialize()
  }

  initialize() {
    this.foo = 0
  }
}
```

请注意，[`strictPropertyInitialization`](/tsconfig#strictPropertyInitialization) 会随其他 [`strict`](/tsconfig#strict) 模式标志一同开启，可能会影响你的项目。
你可以在 `tsconfig.json` 的 `compilerOptions` 中将 [`strictPropertyInitialization`](/tsconfig#strictPropertyInitialization) 设置为 `false`，或在命令行使用 `--strictPropertyInitialization false` 来关闭此检查。

## 确定性赋值断言

确定性赋值断言是一项允许在实例属性和变量声明后放置 `!` 的特性，用于向 TypeScript 表明该变量在实际使用前已经被赋值，即便 TypeScript 的分析无法检测到这一点。

##### 示例

```ts
let x: number
initialize()
console.log(x + x)
//          ~   ~
// Error! Variable 'x' is used before being assigned.

function initialize() {
  x = 10
}
```

使用确定性赋值断言，可以通过在声明时追加 `!` 来断言 `x` 确实已被赋值：

```ts
// Notice the '!'
let x!: number
initialize()

// No error!
console.log(x + x)

function initialize() {
  x = 10
}
```

从某种意义上说，确定性赋值断言运算符与非空断言运算符（在**表达式**后面加 `!`）是对偶关系，后者同样可用于该示例中：

```ts
let x: number
initialize()

// No error!
console.log(x! + x!)

function initialize() {
  x = 10
}
```

在我们的示例中，由于已知所有使用 `x` 的地方都已初始化，使用确定性赋值断言比非空断言更合适。

## 固定长度元组

在 TypeScript 2.6 及更早版本中，`[number, string, string]` 被视为 `[number, string]` 的子类型。
这源于 TypeScript 的结构化类型特性：`[number, string, string]` 的第一个和第二个元素分别是 `[number, string]` 对应元素的子类型。
然而，通过观察元组在实际场景中的用法，我们发现这种允许行为在大多数情况下并不是期望的。

TypeScript 2.7 起，不同元数的元组之间不再可以相互赋值。
感谢 [Kiara Grouwstra](https://github.com/KiaraGrouwstra) 的 pull request，元组类型现在将元数编码进其 `length` 属性的类型中。
这通过利用数字字面量类型来实现，使得不同元数的元组能够相互区分。

从概念上看，你可以将类型 `[number, string]` 理解为等同于以下 `NumStrTuple` 的声明：

```ts
interface NumStrTuple extends Array<number | string> {
  0: number
  1: string
  length: 2 // using the numeric literal type '2'
}
```

注意，这是一项破坏性变更。
如果你需要回归原来的行为（元组只强制最小长度），可以使用类似的声明，但不显式定义 `length` 属性，使其回退为 `number`：

```ts
interface MinimumNumStrTuple extends Array<number | string> {
  0: number
  1: string
}
```

注意，这并不意味着元组代表不可变数组，只是一种约定俗成的规范。

## 改进对象字面量的类型推断

TypeScript 2.7 改进了在相同上下文中出现多个对象字面量时的类型推断。
当多个对象字面量类型共同构成一个联合类型时，我们现在会对这些对象字面量类型进行**规范化**，确保联合类型的每个成员都包含所有属性。

考虑如下代码：

```ts
const obj = test ? { text: 'hello' } : {} // { text: string } | { text?: undefined }
const s = obj.text // string | undefined
```

以前，`obj` 被推断为类型 `{}`，导致第二行报错——因为 `obj` 看起来没有任何属性。
这显然不是理想的行为。

##### 示例

```ts
// let obj: { a: number, b: number } |
//     { a: string, b?: undefined } |
//     { a?: undefined, b?: undefined }
let obj = [{ a: 1, b: 2 }, { a: 'abc' }, {}][0]
obj.a // string | number | undefined
obj.b // number | undefined
```

对同一类型参数的多个对象字面量类型推断同样会被合并为一个规范化的联合类型：

```ts
declare function f<T>(...items: T[]): T
// let obj: { a: number, b: number } |
//     { a: string, b?: undefined } |
//     { a?: undefined, b?: undefined }
let obj = f({ a: 1, b: 2 }, { a: 'abc' }, {})
obj.a // string | number | undefined
obj.b // number | undefined
```

## 改进对结构相同的类与 `instanceof` 表达式的处理

TypeScript 2.7 改进了联合类型和 `instanceof` 表达式中对结构相同的类的处理：

- 结构相同但不同的类类型现在会在联合类型中保留（而不是只保留其中一个）。
- 联合类型的子类型化归约只会在某个类类型是联合中另一个类类型的子类**且**派生自该类时才将其移除。
- `instanceof` 运算符的类型检查现在基于左操作数的类型是否**派生自**右操作数所表示的类型（而非结构子类型检查）。

这意味着联合类型和 `instanceof` 能够正确区分结构相同的类。

##### 示例

```ts
class A {}
class B extends A {}
class C extends A {}
class D extends A {
  c: string
}
class E extends D {}

let x1 = !true ? new A() : new B() // A
let x2 = !true ? new B() : new C() // B | C (previously B)
let x3 = !true ? new C() : new D() // C | D (previously C)

let a1 = [new A(), new B(), new C(), new D(), new E()] // A[]
let a2 = [new B(), new C(), new D(), new E()] // (B | C | D)[] (previously B[])

function f1(x: B | C | D) {
  if (x instanceof B) {
    x // B (previously B | D)
  } else if (x instanceof C) {
    x // C
  } else {
    x // D (previously never)
  }
}
```

## 从 `in` 运算符推断类型守卫

`in` 运算符现在可用作类型的收窄表达式。

对于 `n in x` 表达式，其中 `n` 是字符串字面量或字符串字面量类型，`x` 是联合类型，"true" 分支会收窄为拥有可选或必需属性 `n` 的类型，"false" 分支会收窄为拥有可选或缺少属性 `n` 的类型。

##### 示例

```ts
interface A {
  a: number
}
interface B {
  b: string
}

function foo(x: A | B) {
  if ('a' in x) {
    return x.a
  }
  return x.b
}
```

## 通过 `--esModuleInterop` 支持从 CommonJS 模块使用 `import d from "cjs"`

TypeScript 2.7 更新了 CommonJS/AMD/UMD 模块的代码生成，在 [`esModuleInterop`](/tsconfig#esModuleInterop) 下基于 `__esModule` 标志合成命名空间记录。
这使 TypeScript 生成的输出更接近 Babel 的生成结果。

此前，CommonJS/AMD/UMD 模块与 ES6 模块的处理方式相同，由此引发了若干问题，具体如下：

- TypeScript 将 CommonJS/AMD/UMD 模块的命名空间导入（即 `import * as foo from "foo"`）等同于 `const foo = require("foo")`。这在简单情况下可以工作，但当被导入的主体对象是原始值、类或函数时则会出现问题。ECMAScript 规范规定命名空间记录是一个普通对象，且命名空间导入（如上例中的 `foo`）不可调用——尽管 TypeScript 允许这样做。
- 类似地，CommonJS/AMD/UMD 模块的默认导入（即 `import d from "foo"`）被等同于 `const d = require("foo").default`。而当今绝大多数 CommonJS/AMD/UMD 模块并没有 `default` 导出，这使得该导入模式在导入非 ES 模块（即 CommonJS/AMD/UMD）时几乎无法使用。例如 `import fs from "fs"` 或 `import express from "express"` 均不被允许。

启用新的 [`esModuleInterop`](/tsconfig#esModuleInterop) 后，上述两个问题将得到解决：

- 命名空间导入（即 `import * as foo from "foo"`）现在会被正确标记为不可调用，调用它将产生错误。
- 对 CommonJS/AMD/UMD 的默认导入（如 `import fs from "fs"`）现在是允许的，并可按预期正常工作。

> 注意：此新行为放置在一个编译选项下，以避免对现有代码库造成意外的破坏。**我们强烈建议在新建项目和现有项目中都启用该选项。**
> 对于现有项目，命名空间导入（`import * as express from "express"; express();`）需要转换为默认导入（`import express from "express"; express();`）。

##### 示例

启用 [`esModuleInterop`](/tsconfig#esModuleInterop) 后，将为 `import *` 和 `import default` 分别生成两个新的辅助函数 `__importStar` 和 `__importDefault`。
例如，以下输入：

```ts
import * as foo from 'foo'
import b from 'bar'
```

将生成：

```js
'use strict'
var __importStar =
  (this && this.__importStar) ||
  function (mod) {
    if (mod && mod.__esModule) return mod
    var result = {}
    if (mod != null)
      for (var k in mod)
        if (Object.hasOwnProperty.call(mod, k)) result[k] = mod[k]
    result['default'] = mod
    return result
  }
var __importDefault =
  (this && this.__importDefault) ||
  function (mod) {
    return mod && mod.__esModule ? mod : { default: mod }
  }
exports.__esModule = true
var foo = __importStar(require('foo'))
var bar_1 = __importDefault(require('bar'))
```

## 数字分隔符

TypeScript 2.7 支持 [ES 数字分隔符提案](https://github.com/tc39/proposal-numeric-separator)。
数字字面量现在可以使用 `_` 分隔各段。

##### 示例

```ts
const million = 1_000_000
const phone = 555_734_2231
const bytes = 0xff_0c_00_ff
const word = 0b1100_0011_1101_0001
```

## `--watch` 模式下更整洁的输出

TypeScript 的 `--watch` 模式现在会在触发重新编译后清空屏幕。

## 更美观的 `--pretty` 输出

TypeScript 的 [`pretty`](/tsconfig#pretty) 标志可以让错误信息更易于阅读和管理。
[`pretty`](/tsconfig#pretty) 现在会为文件名、诊断代码和行号添加颜色。
文件名和位置信息也经过格式化，支持在常见终端（如 Visual Studio Code 终端）中直接跳转。
