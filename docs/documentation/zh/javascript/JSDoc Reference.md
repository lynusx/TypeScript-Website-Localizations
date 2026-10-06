---
title: JSDoc 参考
layout: docs
permalink: /zh/docs/handbook/jsdoc-supported-types.html
oneline: 基于 TypeScript 的 JavaScript 支持哪些 JSDoc 特性？
translatable: true
---

下文列出了在 JavaScript 文件中使用 JSDoc 注解提供类型信息时，当前支持的所有语法结构。

注意：

- 下方未明确列出的任何标签（例如 `@async`）目前均暂不支持。
- TypeScript 文件中仅支持文档标签（Documentation tags）。其余标签仅在 JavaScript 文件中受支持。

#### Types

- [`@type`](#type)
- [`@import`](#import)
- [`@param`](#param-and-returns)（或 [`@arg`](#param-and-returns) 或 [`@argument`](#param-and-returns)）
- [`@returns`](#param-and-returns)（或 [`@return`](#param-and-returns)）
- [`@typedef`](#typedef-callback-and-param)
- [`@callback`](#typedef-callback-and-param)
- [`@template`](#template)
- [`@satisfies`](#satisfies)

#### Classes

- [Property Modifiers](#property-modifiers) `@public`, `@private`, `@protected`, `@readonly`
- [`@override`](#override)
- [`@extends`](#extends)（或 [`@augments`](#extends)）
- [`@implements`](#implements)
- [`@class`](#constructor)（或 [`@constructor`](#constructor)）
- [`@this`](#this)

#### Documentation

文档标签在 TypeScript 和 JavaScript 中均可使用。

- [`@deprecated`](#deprecated)
- [`@see`](#see)
- [`@link`](#link)

#### Other

- [`@enum`](#enum)
- [`@author`](#author)
- [Other supported patterns](#other-supported-patterns)
- [Unsupported patterns](#unsupported-patterns)
- [Unsupported tags](#unsupported-tags)

这些标签的含义通常与 [jsdoc.app](https://jsdoc.app) 中给出的含义相同，或是其超集。
下文的代码说明了具体差异，并给出了各标签的使用示例。

**注意：** 你可以使用 [Playground 体验 JSDoc 支持](/play?useJavaScript=truee=4#example/jsdoc-support)。

## Types

### `@type`

你可以使用 "@type" 标签来引用类型。该类型可以是：

1. 原始类型，例如 `string` 或 `number`。
2. 在 TypeScript 声明中声明的类型（无论是全局的还是导入的）。
3. 在 JSDoc [`@typedef`](#typedef-callback-and-param) 标签中声明的类型。

你可以使用大多数 JSDoc 类型语法以及任意 TypeScript 语法，从[最基础的 `string`](/docs/handbook/2/basic-types.html) 到[最高级的条件类型（Conditional Types）](/docs/handbook/2/conditional-types.html)。

```js twoslash
/**
 * @type {string}
 */
var s

/** @type {Window} */
var win

/** @type {PromiseLike<string>} */
var promisedString

// 你可以指定带有 DOM 属性的 HTML 元素
/** @type {HTMLElement} */
var myElement = document.querySelector(selector)
element.dataset.myData = ''
```

`@type` 可以指定联合类型 &mdash; 例如，某个变量既可以是字符串也可以是布尔值。

```js twoslash
/**
 * @type {string | boolean}
 */
var sb
```

你可以使用多种语法来指定数组类型：

```js twoslash
/** @type {number[]} */
var ns
/** @type {Array.<number>} */
var jsdoc
/** @type {Array<number>} */
var nas
```

你还可以指定对象字面量类型。
例如，具有属性 'a'（string）和 'b'（number）的对象可以使用以下语法：

```js twoslash
/** @type {{ a: string, b: number }} */
var var9
```

你可以使用字符串和数字索引签名来指定类似 Map 的对象和类数组对象，这既可以使用标准 JSDoc 语法，也可以使用 TypeScript 语法。

```js twoslash
/**
 * 类似 Map 的对象，将任意 `string` 属性映射为 `number`。
 *
 * @type {Object.<string, number>}
 */
var stringToNumber

/** @type {Object.<number, object>} */
var arrayLike
```

上述两个类型分别等价于 TypeScript 类型 `{ [x: string]: number }` 和 `{ [x: number]: any }`。编译器可以理解这两种语法。

你可以使用 TypeScript 语法或 Google Closure 语法来指定函数类型：

```js twoslash
/** @type {function(string, boolean): number} Closure 语法 */
var sbn
/** @type {(s: string, b: boolean) => number} TypeScript 语法 */
var sbn2
```

或者你也可以直接使用未指定参数和返回值的 `Function` 类型：

```js twoslash
/** @type {Function} */
var fn7
/** @type {function} */
var fn6
```

来自 Closure 的其他类型同样有效：

```js twoslash
/**
 * @type {*} - 可以是 'any' 类型
 */
var star
/**
 * @type {?} - 未知类型（等同于 'any'）
 */
var question
```

#### Casts

TypeScript 借鉴了 Google Closure 的类型转换语法。
通过在任意带括号的表达式前面添加 `@type` 标签，你可以将类型转换为其他类型。

```js twoslash
/**
 * @type {number | string}
 */
var numberOrString = Math.random() < 0.5 ? 'hello' : 100
var typeAssertedNumber = /** @type {number} */ (numberOrString)
```

你甚至可以像在 TypeScript 中一样转换为 `const`：

```js twoslash
let one = /** @type {const} */ (1)
```

#### Import types

你可以使用导入类型（Import types）从其他文件中导入声明。
该语法是 TypeScript 特有的，与 JSDoc 标准有所不同：

```js twoslash
// @filename: types.d.ts
export type Pet = {
  name: string,
};

// @filename: main.js
/**
 * @param {import("./types").Pet} p
 */
function walk(p) {
  console.log(`Walking ${p.name}...`);
}
```

如果你不知道某个值的类型，或者该值的类型过于庞大繁琐而不便手写，你可以使用导入类型从模块中获取该值的类型：

```js twoslash
// @types: node
// @filename: accounts.d.ts
export const userAccount = {
  name: 'Name',
  address: 'An address',
  postalCode: '',
  country: '',
  planet: '',
  system: '',
  galaxy: '',
  universe: '',
}
// @filename: main.js
// ---cut---
/**
 * @type {typeof import("./accounts").userAccount}
 */
var x = require('./accounts').userAccount
```

### `@import`

`@import` 标签允许我们引用其他文件的导出项。

```js twoslash
// @filename: types.d.ts
export type Pet = {
  name: string,
};
// @filename: main.js
// ---cut---
/**
 * @import {Pet} from "./types"
 */

/**
 * @type {Pet}
 */
var myPet;
myPet.name;
```

这些标签在运行时并不会真正导入文件，它们引入作用域的符号只能在 JSDoc 注释中用于类型检查。

```js twoslash
// @filename: dog.js
export class Dog {
  woof() {
    console.log('Woof!')
  }
}

// @filename: main.js
/** @import { Dog } from "./dog.js" */

const d = new Dog() // 报错！
```

### `@param` and `@returns`

`@param` 使用与 `@type` 相同的类型语法，但额外增加了一个参数名称。
还可以通过用方括号将参数名括起来，将参数声明为可选参数：

```js twoslash
// 可以使用多种语法形式来声明参数
/**
 * @param {string}  p1 - string 类型参数。
 * @param {string=} p2 - 可选参数（Google Closure 语法）
 * @param {string} [p3] - 另一个可选参数（JSDoc 语法）。
 * @param {string} [p4="test"] - 带有默认值的可选参数
 * @returns {string} 这是返回值
 */
function stringsStringStrings(p1, p2, p3, p4) {
  // TODO
}
```

同样地，对于函数的返回值类型：

```js twoslash
/**
 * @return {PromiseLike<string>}
 */
function ps() {}

/**
 * @returns {{ a: string, b: number }} - 可以使用 '@returns'，也可以使用 '@return'
 */
function ab() {}
```

### `@typedef`, `@callback`, and `@param`

你可以使用 `@typedef` 定义复杂类型。
类似的语法也适用于 `@param`。

```js twoslash
/**
 * @typedef {Object} SpecialType - 创建名为 'SpecialType' 的新类型
 * @property {string} prop1 - SpecialType 的 string 属性
 * @property {number} prop2 - SpecialType 的 number 属性
 * @property {number=} prop3 - SpecialType 的可选 number 属性
 * @prop {number} [prop4] - SpecialType 的可选 number 属性
 * @prop {number} [prop5=42] - SpecialType 带有默认值的可选 number 属性
 */

/** @type {SpecialType} */
var specialTypeObject
specialTypeObject.prop3
```

在第一行中，你可以使用 `object` 或 `Object`。

```js twoslash
/**
 * @typedef {object} SpecialType1 - 创建名为 'SpecialType1' 的新类型
 * @property {string} prop1 - SpecialType1 的 string 属性
 * @property {number} prop2 - SpecialType1 的 number 属性
 * @property {number=} prop3 - SpecialType1 的可选 number 属性
 */

/** @type {SpecialType1} */
var specialTypeObject1
```

`@param` 支持类似的语法用于一次性类型声明。
请注意，嵌套属性名称必须带有参数名称作为前缀：

```js twoslash
/**
 * @param {Object} options - 结构与上面的 SpecialType 相同
 * @param {string} options.prop1
 * @param {number} options.prop2
 * @param {number=} options.prop3
 * @param {number} [options.prop4]
 * @param {number} [options.prop5=42]
 */
function special(options) {
  return (options.prop4 || 1001) + options.prop5
}
```

`@callback` 与 `@typedef` 类似，但它指定的是函数类型而非对象类型：

```js twoslash
/**
 * @callback Predicate
 * @param {string} data
 * @param {number} [index]
 * @returns {boolean}
 */

/** @type {Predicate} */
const ok = (s) => !(s.length % 2)
```

当然，所有这些类型都可以在单行 `@typedef` 中使用 TypeScript 语法进行声明：

```js
/** @typedef {{ prop1: string, prop2: string, prop3?: number }} SpecialType */
/** @typedef {(data: string, index?: number) => boolean} Predicate */
```

### `@template`

你可以使用 `@template` 标签声明类型参数。
这让你可以创建泛型函数、泛型类或泛型类型：

```js twoslash
/**
 * @template T
 * @param {T} x - 传递至返回类型的泛型参数
 * @returns {T}
 */
function id(x) {
  return x
}

const a = id('string')
const b = id(123)
const c = id({})
```

使用逗号或多个标签来声明多个类型参数：

```js
/**
 * @template T,U,V
 * @template W,X
 */
```

你还可以在类型参数名称之前指定类型约束。
列表中只有第一个类型参数会被约束：

```js twoslash
/**
 * @template {string} K - K 必须是 string 或字符串字面量
 * @template {{ serious(): string }} Seriousalizable - 必须包含 serious 方法
 * @param {K} key
 * @param {Seriousalizable} object
 */
function seriousalize(key, object) {
  // ????
}
```

最后，你可以为类型参数指定默认值：

```js twoslash
/** @template [T=object] */
class Cache {
  /** @param {T} initial */
  constructor(initial) {}
}
let c = new Cache()
```

### `@satisfies`

`@satisfies` 允许访问 TypeScript 中的后缀[运算符 `satisfies`](/docs/handbook/release-notes/typescript-4-9.html)。Satisfies 用于声明某个值满足某种类型，但不会改变该值本身的推导类型。

```js twoslash
// @errors: 1360
// @ts-check
/**
 * @typedef {"hello world" | "Hello, world"} WelcomeMessage
 */

/** @satisfies {WelcomeMessage} */
const message = 'hello world'
//     ^?

/** @satisfies {WelcomeMessage} */
const failingMessage = 'Hello world!'

/** @type {WelcomeMessage} */
const messageUsingType = 'hello world'
//     ^?
```

## Classes

类可以声明为 ES6 类。

```js twoslash
class C {
  /**
   * @param {number} data
   */
  constructor(data) {
    // 属性类型可以被推断
    this.name = 'foo'

    // 或显式设置
    /** @type {string | null} */
    this.title = null

    // 或者如果它们在其他地方设置，只需添加类型注解
    /** @type {number} */
    this.size

    this.initialize(data) // 应当报错，initializer 期望接收一个 string
  }
  /**
   * @param {string} s
   */
  initialize = function (s) {
    this.size = s.length
  }
}

var c = new C(0)

// C 应当仅通过 new 调用，但
// 因为这是 JavaScript，所以这是允许的且
// 被视为 'any'。
var result = C(1)
```

它们也可以声明为构造函数；此时请结合使用 [`@constructor`](#constructor) 与 [`@this`](#this)。

### Property Modifiers

<div id="jsdoc-property-modifiers"></div>

`@public`、`@private` 和 `@protected` 的工作方式与 TypeScript 中的 `public`、`private` 和 `protected` 完全一致：

```js twoslash
// @errors: 2341
// @ts-check

class Car {
  constructor() {
    /** @private */
    this.identifier = 100
  }

  printIdentifier() {
    console.log(this.identifier)
  }
}

const c = new Car()
console.log(c.identifier)
```

- `@public` 始终是默认隐含的，可以省略，它表示属性可以从任何地方访问。
- `@private` 表示属性只能在包含它的类内部使用。
- `@protected` 表示属性只能在包含它的类及其所有派生子类内部使用，但不能在包含类的不同实例上访问。

`@public`、`@private` 和 `@protected` 在构造函数中不起作用。

### `@readonly`

`@readonly` 修饰符可确保属性仅在初始化期间被写入。

```js twoslash
// @errors: 2540
// @ts-check

class Car {
  constructor() {
    /** @readonly */
    this.identifier = 100
  }

  printIdentifier() {
    console.log(this.identifier)
  }
}

const c = new Car()
console.log(c.identifier)
```

### `@override`

`@override` 的工作方式与 TypeScript 中相同；将其用于重写基类方法的方法上：

```js twoslash
export class C {
  m() {}
}
class D extends C {
  /** @override */
  m() {}
}
```

在 tsconfig 中设置 `noImplicitOverride: true` 可以检查方法重写。

### `@extends`

当 JavaScript 类继承泛型基类时，JavaScript 并没有传递类型实参的语法。`@extends` 标签支持了这一需求：

```js twoslash
/**
 * @template T
 * @extends {Set<T>}
 */
class SortableSet extends Set {
  // ...
}
```

注意，`@extends` 仅适用于类。目前，构造函数无法继承类。

### `@implements`

同理，JavaScript 也没有实现 TypeScript 接口的语法。`@implements` 标签的作用与 TypeScript 中的实现机制完全一致：

```js twoslash
/** @implements {Print} */
class TextBook {
  print() {
    // TODO
  }
}
```

### `@constructor`

编译器会根据 this 属性赋值来推断构造函数，但如果添加 `@constructor` 标签，可以使检查更严格、代码提示更完善：

```js twoslash
// @checkJs
// @errors: 2345 2348
/**
 * @constructor
 * @param {number} data
 */
function C(data) {
  // 属性类型可以被推断
  this.name = 'foo'

  // 或显式设置
  /** @type {string | null} */
  this.title = null

  // 或者如果它们在其他地方设置，只需添加类型注解
  /** @type {number} */
  this.size

  this.initialize(data)
}
/**
 * @param {string} s
 */
C.prototype.initialize = function (s) {
  this.size = s.length
}

var c = new C(0)
c.size

var result = C(1)
```

> 注意：错误信息仅在配置了 [JSConfig](/docs/handbook/tsconfig-json.html) 并启用了 [`checkJs`](/tsconfig#checkJs) 的 JS 代码库中才会显示。

使用 `@constructor` 后，构造函数 `C` 内部会对 `this` 进行检查，因此你将获得 `initialize` 方法的代码提示，并在传入数字时收到错误。如果你直接调用 `C` 而非构造调用它，编辑器可能还会显示警告。

遗憾的是，这意味着同时支持直接调用的构造函数无法使用 `@constructor`。

### `@this`

当存在可用上下文时，编译器通常可以推断出 `this` 的类型。而在无法推断时，你可以使用 `@this` 明确指定 `this` 的类型：

```js twoslash
/**
 * @this {HTMLElement}
 * @param {*} e
 */
function callbackForLater(e) {
  this.clientHeight = parseInt(e) // 应该没问题！
}
```

## Documentation

### `@deprecated`

<div id="deprecated-comments"></div>

当某个函数、方法或属性被废弃时，你可以通过使用 `/** @deprecated */` JSDoc 注释标记来告知用户。该信息会显示在补全列表中，并作为编辑器的诊断建议进行特殊展示。在诸如 VS Code 等编辑器中，废弃的值通常会以带有删除线的样式显示，~~就像这样~~。

```js twoslash
// @noErrors
/** @deprecated */
const apiV1 = {}
const apiV2 = {}

apiV
// ^|
```

### `@see`

`@see` 允许你链接到程序中的其他标识符名称：

```ts twoslash
type Box<T> = { t: T }
/** @see Box 获取实现细节 */
type Boxify<T> = { [K in keyof T]: Box<T> }
```

部分编辑器会将 `Box` 显示为可点击的链接，以便快速跳转和返回。

### `@link`

`@link` 与 `@see` 类似，不同之处在于它可以内嵌在其他标签内部使用：

```ts twoslash
type Box<T> = { t: T }
/** @returns 包含该参数的 {@link Box}。 */
function box<U>(u: U): Box<U> {
  return { t: u }
}
```

你也可以链接到一个属性：

```ts twoslash
type Pet = {
  name: string
  hello: () => string
}

/**
 * 注意：你应该实现 Pet 的 {@link Pet.hello} 方法。
 */
function hello(p: Pet) {
  p.hello()
}
```

或者指定可选的显示名称：

```ts twoslash
type Pet = {
  name: string
  hello: () => string
}

/**
 * 注意：你应该实现 Pet 的 {@link Pet.hello | hello} 方法。
 */
function hello(p: Pet) {
  p.hello()
}
```

## Other

### `@enum`

`@enum` 标签允许你创建一个对象字面量，其所有成员均为指定的类型。与 JavaScript 中的大多数对象字面量不同，它不允许包含其他成员。
`@enum` 的设计初衷是为了兼容 Google Closure 的 `@enum` 标签。

```js twoslash
/** @enum {number} */
const JSDocState = {
  BeginningOfLine: 0,
  SawAsterisk: 1,
  SavingComments: 2,
}

JSDocState.SawAsterisk
```

注意，`@enum` 与 TypeScript 的 `enum` 截然不同，且要简单得多。不过与 TypeScript 枚举不同的是，`@enum` 可以是任意类型：

```js twoslash
/** @enum {function(number): number} */
const MathFuncs = {
  add1: (n) => n + 1,
  id: (n) => -n,
  sub1: (n) => n - 1,
}

MathFuncs.add1
```

### `@author`

你可以使用 `@author` 指定项目的作者：

```ts twoslash
/**
 * 欢迎使用 awesome.ts
 * @author Ian Awesome <i.am.awesome@example.com>
 */
```

请记得用尖括号括起电子邮箱地址。
否则，`@example` 会被解析为一个新的标签。

### Other supported patterns

```js twoslash
// @types: react
class Foo {}
// ---cut---
var someObj = {
  /**
   * @param {string} param1 - 属性赋值上的 JSDoc 同样生效
   */
  x: function (param1) {},
}

/**
 * 变量赋值上的 JSDoc 也同样有效
 * @return {Window}
 */
let someFunc = function () {}

/**
 * 类方法也同样如此
 * @param {string} greeting 要使用的问候语
 */
Foo.prototype.sayHi = (greeting) => console.log('Hi!')

/**
 * 箭头函数表达式也同样如此
 * @param {number} x - 乘数
 */
let myArrow = (x) => x * x

/**
 * 这意味着它同样适用于 JSX 中的函数组件
 * @param {{a: string, b: number}} props - 某些参数
 */
var fc = (props) => <div>{props.a.charAt(0)}</div>

/**
 * 参数可以是类构造函数，使用 Google Closure 语法。
 *
 * @param {{new(...args: any[]): object}} C - 要注册的类
 */
function registerClass(C) {}

/**
 * @param {...string} p1 - 字符串类型的 'rest' 参数（数组）。（被视为 'any'）
 */
function fn10(p1) {}

/**
 * @param {...string} p1 - 字符串类型的 'rest' 参数（数组）。（被视为 'any'）
 */
function fn9(p1) {
  return p1.join()
}
```

### Unsupported patterns

在对象字面量类型中，属性类型后置等号并不能将其指定为可选属性：

```js twoslash
/**
 * @type {{ a: string, b: number= }}
 */
var wrong
/**
 * 改为在属性名后使用问号：
 * @type {{ a: string, b?: number }}
 */
var right
```

可空类型（Nullable types）仅在开启 [`strictNullChecks`](/tsconfig#strictNullChecks) 时才有意义：

```js twoslash
/**
 * @type {?number}
 * 当 strictNullChecks 为 true  -- number | null
 * 当 strictNullChecks 为 false -- number
 */
var nullable
```

TypeScript 原生语法使用联合类型：

```js twoslash
/**
 * @type {number | null}
 * 当 strictNullChecks 为 true  -- number | null
 * 当 strictNullChecks 为 false -- number
 */
var unionNullable
```

不可空类型（Non-nullable types）没有特殊含义，仅会被当作其原始类型处理：

```js twoslash
/**
 * @type {!number}
 * 类型仅为 number
 */
var normal
```

与 JSDoc 的类型系统不同，TypeScript 仅允许你将类型标记为包含 null 或不包含 null。
并不存在显式的不可空性（non-nullability）&mdash;&mdash; 如果开启了 strictNullChecks，那么 `number` 本身就不可为空；如果关闭了该选项，那么 `number` 就是可空的。

### Unsupported tags

TypeScript 会忽略所有不受支持的 JSDoc 标签。

以下标签已有未关闭的 Issue 提议支持：

- `@memberof`（[Issue #7237](https://github.com/Microsoft/TypeScript/issues/7237)）
- `@yields`（[Issue #23857](https://github.com/Microsoft/TypeScript/issues/23857)）
- `@member`（[Issue #56674](https://github.com/microsoft/TypeScript/issues/56674)）

### Legacy type synonyms

为了兼容旧版 JavaScript 代码，为许多常见类型提供了别名。
部分别名与现存类型同名，尽管其中大多数很少使用。
例如，`String` 被视作 `string` 的别名。
尽管 `String` 在 TypeScript 中也是一种类型，但旧版 JSDoc 经常用它来表示 `string`。
此外，在 TypeScript 中，原始类型的大写形式是包装对象类型（Wrapper types）&mdash;&mdash; 使用它们几乎总是错误的。
因此编译器根据旧版 JSDoc 的用法将这些类型视为同义词：

- `String -> string`
- `Number -> number`
- `Boolean -> boolean`
- `Void -> void`
- `Undefined -> undefined`
- `Null -> null`
- `function -> Function`
- `array -> Array<any>`
- `promise -> Promise<any>`
- `Object -> any`
- `object -> any`

当启用 `noImplicitAny: true` 时，最后四个别名将被禁用：

- `object` 和 `Object` 是内置类型，尽管 `Object` 很少被使用。
- `array` 和 `promise` 不是内置类型，但可能会在你程序中的某个地方被声明。
