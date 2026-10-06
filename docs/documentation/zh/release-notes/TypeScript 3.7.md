---
title: TypeScript 3.7
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-3-7.html
oneline: TypeScript 3.7 发布说明
---

## 可选链（Optional Chaining）

[演练场](/play/#example/optional-chaining)

可选链是 TypeScript 议题追踪器上的[第 16 号议题（issue #16）](https://github.com/microsoft/TypeScript/issues/16)。作为参考，自那时起，TypeScript 议题追踪器上已经产生过超过 23,000 个议题。

从核心机制来说，可选链允许我们编写这样一种代码：一旦遇到 `null` 或 `undefined`，TypeScript 就会立即停止执行后续的部分表达式。
可选链中最核心的亮点是用于_可选属性访问_的全新 `?.` 运算符。
当我们编写如下代码时：

```ts
let x = foo?.bar.baz()
```

这表达的是：当 `foo` 有定义时，将计算 `foo.bar.baz()`；但当 `foo` 为 `null` 或 `undefined` 时，立即中止后续操作并直接返回 `undefined`。

更通俗地说，该代码片段等同于编写以下代码：

```ts
let x = foo === null || foo === undefined ? undefined : foo.bar.baz()
```

请注意，如果 `bar` 为 `null` 或 `undefined`，代码在访问 `baz` 时仍然会报错。
同样地，如果 `baz` 为 `null` 或 `undefined`，在调用处也会报错。
`?.` 仅检查其_左侧_的值是否为 `null` 或 `undefined`——而不会检查后续的属性。

你可能会发现，可以使用 `?.` 来替换大量使用 `&&` 运算符进行重复空值检查的代码：

```ts
// 之前
if (foo && foo.bar && foo.bar.baz) {
  // ...
}

// 之后大致如此
if (foo?.bar?.baz) {
  // ...
}
```

请注意，`?.` 的行为与 `&&` 操作有所不同，因为 `&&` 会对所有“假值（falsy）”（例如空字符串、`0`、`NaN` 以及 `false`）生效，但这是可选链设计上的刻意为之。
它不会对像 `0` 或空字符串这样的有效数据发生短路。

可选链还包含另外两种操作。
首先是_可选元素访问_，其行为与可选属性访问类似，但允许我们访问非标识符属性（例如任意字符串、数字和 symbol）：

```ts
/**
 * 获取数组的第一个元素（如果传入了数组）。
 * 否则返回 undefined。
 */
function tryGetFirstElement<T>(arr?: T[]) {
  return arr?.[0]
  // 等价于
  //   return (arr === null || arr === undefined) ?
  //       undefined :
  //       arr[0];
}
```

其次是_可选调用_，它允许我们在表达式不为 `null` 或 `undefined` 时有条件地调用它们：

```ts
async function makeRequest(url: string, log?: (msg: string) => void) {
  log?.(`Request started at ${new Date().toISOString()}`)
  // 大致等价于
  //   if (log != null) {
  //       log(`Request started at ${new Date().toISOString()}`);
  //   }

  const result = (await fetch(url)).json()

  log?.(`Request finished at ${new Date().toISOString()}`)

  return result
}
```

可选链的“短路”行为仅限于属性访问、函数调用以及元素访问——不会进一步向外扩展到这些表达式之外。
换句话说，

```ts
let result = foo?.bar / someComputation()
```

并不会阻止除法运算或 `someComputation()` 的调用。
它等价于：

```ts
let temp = foo === null || foo === undefined ? undefined : foo.bar

let result = temp / someComputation()
```

这可能会导致与 `undefined` 进行除法运算，因此在开启 [`strictNullChecks`](/tsconfig#strictNullChecks) 时，以下代码会报错：

```ts
function barPercentage(foo?: { bar: number }) {
  return foo?.bar / 100
  //     ~~~~~~~~
  // 错误：对象可能为 undefined。
}
```

欲了解更多详情，可以[查阅该提案规范](https://github.com/tc39/proposal-optional-chaining/)并[查看原始 Pull Request](https://github.com/microsoft/TypeScript/pull/33294)。

## 空值合并运算符（Nullish Coalescing）

[演练场](/play/#example/nullish-coalescing)

_空值合并运算符（nullish coalescing operator）_是另一项即将加入 ECMAScript 的特性，它与可选链相辅相成，TypeScript 团队也深入参与了 TC39 中该特性的推动与支持。

你可以将这个特性——即 `??` 运算符——视为在遇到 `null` 或 `undefined` 时“回退”到默认值的一种方式。
当我们编写如下代码时：

```ts
let x = foo ?? bar()
```

这是一种新的表达方式，表示当 `foo` “存在”时直接使用它；
但当它为 `null` 或 `undefined` 时，则计算并使用 `bar()`。

同样地，上述代码等同于以下写法：

```ts
let x = foo !== null && foo !== undefined ? foo : bar()
```

在尝试使用默认值时，`??` 运算符可以替代使用 `||` 的场景。
例如，下面的代码片段尝试获取上次保存在 [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) 中的音量设置（如果保存过的话）；
然而，由于使用了 `||`，这里存在一个 bug：

```ts
function initializeAudio() {
  let volume = localStorage.volume || 0.5

  // ...
}
```

当 `localStorage.volume` 设置为 `0` 时，页面会将音量设置为 `0.5`，这并非预期行为。
`??` 避免了因将 `0`、`NaN` 和 `""` 视作假值而导致的一些非预期行为。

我们非常感谢社区成员 [Wenlu Wang](https://github.com/Kingwl) 与 [Titian Cernicova Dragomir](https://github.com/dragomirtitian) 实现了这一特性！
欲了解更多详情，请[查看他们的 Pull Request](https://github.com/microsoft/TypeScript/pull/32883) 以及[空值合并提案仓库](https://github.com/tc39/proposal-nullish-coalescing/)。

## 断言函数（Assertion Functions）

[演练场](/play/#example/assertion-functions)

有一类特殊的函数，当发生意外情况时会 `throw` 一个错误。
它们被称为“断言”函数。
例如，Node.js 就为此提供了一个专门的函数 `assert`：

```js
assert(someValue === 42)
```

在这个例子中，如果 `someValue` 不等于 `42`，`assert` 就会抛出 `AssertionError`。

JavaScript 中的断言经常用于防止传入不符合预期的类型。
例如：

```js
function multiply(x, y) {
  assert(typeof x === 'number')
  assert(typeof y === 'number')

  return x * y
}
```

遗憾的是，在以往的 TypeScript 中，这些检查无法被准确地类型化。
对于弱类型风格的代码，这意味着 TypeScript 进行的检查变少了；而对于较为严谨的代码，又往往迫使用户使用类型断言。

```ts
function yell(str) {
  assert(typeof str === 'string')

  return str.toUppercase()
  // 糟糕！我们把 'toUpperCase' 拼错了。
  // 如果 TypeScript 仍然能捕获到这一点就太棒了！
}
```

另一种替代方案是重写代码，使类型系统能够进行静态分析，但这并不方便：

```ts
function yell(str) {
  if (typeof str !== 'string') {
    throw new TypeError('str should have been a string.')
  }
  // 捕获到错误！
  return str.toUppercase()
}
```

归根结底，TypeScript 的目标是以侵入性最小的方式为现有的 JavaScript 结构赋予类型。
因此，TypeScript 3.7 引入了一个名为“断言签名（assertion signatures）”的新概念，用来对这类断言函数进行建模。

第一种断言签名对 Node 的 `assert` 函数工作方式进行建模。
它能够确保所检查的任何条件在当前包含作用域的后续代码中必须为真：

```ts
function assert(condition: any, msg?: string): asserts condition {
  if (!condition) {
    throw new AssertionError(msg)
  }
}
```

`asserts condition` 表示如果 `assert` 正常返回（否则会抛出错误），传入 `condition` 参数的任何表达式都必须为真。
这意味着在当前作用域的其余部分中，该条件必须为真值。
例如，使用这个断言函数意味着我们_确实_能够捕获最初 `yell` 示例中的错误：

```ts
function yell(str) {
  assert(typeof str === 'string')

  return str.toUppercase()
  //         ~~~~~~~~~~~
  // 错误：类型 'string' 上不存在属性 'toUppercase'。
  //        你指的是 'toUpperCase' 吗？
}

function assert(condition: any, msg?: string): asserts condition {
  if (!condition) {
    throw new AssertionError(msg)
  }
}
```

另一种断言签名不检查条件表达式，而是告诉 TypeScript 某个特定的变量或属性具有不同的类型：

```ts
function assertIsString(val: any): asserts val is string {
  if (typeof val !== 'string') {
    throw new AssertionError('Not a string!')
  }
}
```

这里的 `asserts val is string` 能够确保在调用 `assertIsString` 之后，传入的任何变量都会被推断为 `string` 类型：

```ts
function yell(str: any) {
  assertIsString(str)

  // 现在 TypeScript 知道 'str' 是一个 'string'。

  return str.toUppercase()
  //         ~~~~~~~~~~~
  // 错误：类型 'string' 上不存在属性 'toUppercase'。
  //        你指的是 'toUpperCase' 吗？
}
```

这些断言签名与编写类型谓词签名非常相似：

```ts
function isString(val: any): val is string {
  return typeof val === 'string'
}

function yell(str: any) {
  if (isString(str)) {
    return str.toUppercase()
  }
  throw 'Oops!'
}
```

就像类型谓词签名一样，这些断言签名的表达能力非常强大。
我们可以借此表达一些相当复杂的概念：

```ts
function assertIsDefined<T>(val: T): asserts val is NonNullable<T> {
  if (val === undefined || val === null) {
    throw new AssertionError(
      `Expected 'val' to be defined, but received ${val}`,
    )
  }
}
```

欲了解有关断言签名的更多信息，请[查看原始 Pull Request](https://github.com/microsoft/TypeScript/pull/32695)。

## 更好地支持返回 `never` 的函数

作为断言签名工作的一部分，TypeScript 需要对何时何处调用了哪些函数记录更多信息。
这也为我们提供了一个契机，以扩展对另一类函数的支持：返回 `never` 的函数。

任何返回 `never` 的函数的意图都是它永远不会返回。
这表明抛出了异常、发生了中止程序的错误条件，或者程序已经退出。
例如，[`@types/node` 中的 `process.exit(...)`](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/5299d372a220584e75a031c13b3d555607af13f8/types/node/globals.d.ts#l874) 就被指定为返回 `never`。

以往为了确保函数不会潜在返回 `undefined`，或者确保所有代码路径都有有效返回，TypeScript 需要某种语法标记——要么在函数末尾使用 `return`，要么使用 `throw`。
因此，用户不得不对这些失败处理函数使用 `return`：

```ts
function dispatch(x: string | number): SomeType {
  if (typeof x === 'string') {
    return doThingWithString(x)
  } else if (typeof x === 'number') {
    return doThingWithNumber(x)
  }
  return process.exit(1)
}
```

现在，当调用这些返回 `never` 的函数时，TypeScript 能够识别出它们会影响控制流图，并将其纳入分析考量：

```ts
function dispatch(x: string | number): SomeType {
  if (typeof x === 'string') {
    return doThingWithString(x)
  } else if (typeof x === 'number') {
    return doThingWithNumber(x)
  }
  process.exit(1)
}
```

与断言函数一样，你可以在[同一个 Pull Request 中了解更多信息](https://github.com/microsoft/TypeScript/pull/32695)。

## （更广泛的）递归类型别名

[演练场](/play/#example/recursive-type-references)

类型别名在如何“递归”引用自身方面一直存在限制。
其原因在于，类型别名的任何使用都需要能够将其自身替换为它所别名引用的内容。
在某些情况下这是不可能做到的，因此编译器会拒绝某些递归别名，例如：

```ts
type Foo = Foo
```

这是一个合理的限制，因为任何对 `Foo` 的使用都需要被替换为 `Foo`，而 `Foo` 又需要被替换为 `Foo`，接着又需要被替换为 `Foo`……嗯，你懂的！
最终，没有任何合理的具体类型可以替代 `Foo`。

这与[其他语言处理类型别名的方式](https://wikipedia.org/w/index.php?title=Recursive_data_type&oldid=913091335#in_type_synonyms)相当一致，但这确实在使用该特性的某些场景中给用户带来了一些困扰。
例如，在 TypeScript 3.6 及更早版本中，以下代码会导致错误：

```ts
type ValueOrArray<T> = T | Array<ValueOrArray<T>>
//   ~~~~~~~~~~~~
// 错误：类型别名 'ValueOrArray' 循环引用了自身。
```

这很奇怪，因为在技术上这种使用方式没有任何问题，而且用户总是可以通过引入一个接口来编写效果相同的代码：

```ts
type ValueOrArray<T> = T | ArrayOfValueOrArray<T>

interface ArrayOfValueOrArray<T> extends Array<ValueOrArray<T>> {}
```

由于接口（以及其他对象类型）引入了一层间接引用，并且其完整结构不需要被急迫地（eagerly）全量构建出来，因此 TypeScript 处理这种结构完全没有问题。

但引入接口这种变通方法对用户来说并不直观。
而且从原理上讲，直接使用 `Array` 的原版 `ValueOrArray` 并没有任何错误。
如果编译器能够更“惰性”一点，仅在必要时才计算 `Array` 的类型参数，那么 TypeScript 就能正确地表达这些类型。

这正是 TypeScript 3.7 所引入的改进。
在类型别名的“顶层”，TypeScript 会延迟解析类型参数以允许这些模式。

这意味着以下尝试表示 JSON 的代码……

```ts
type Json = string | number | boolean | null | JsonObject | JsonArray

interface JsonObject {
  [property: string]: Json
}

interface JsonArray extends Array<Json> {}
```

终于可以省去辅助接口并重写为：

```ts
type Json =
  string | number | boolean | null | { [property: string]: Json } | Json[]
```

这一放宽限制的新特性还允许我们在元组中递归引用类型别名。
以下过去会报错的代码现在已经成为合法的 TypeScript 代码：

```ts
type VirtualNode = string | [string, { [key: string]: any }, ...VirtualNode[]]

const myNode: VirtualNode = [
  'div',
  { id: 'parent' },
  ['div', { id: 'first-child' }, "I'm the first child"],
  ['div', { id: 'second-child' }, "I'm the second child"],
]
```

欲了解更多信息，可以[查阅原始 Pull Request](https://github.com/microsoft/TypeScript/pull/33050)。

## `--declaration` 与 `--allowJs`

TypeScript 中的 [`declaration`](/tsconfig#declaration) 标志允许我们从 TypeScript 源文件（即 `.ts` 和 `.tsx` 文件）生成 `.d.ts` 文件（声明文件）。
这些 `.d.ts` 文件的价值体现在几个重要方面：

首先，它们允许 TypeScript 在对其他项目进行类型检查时，无须重新检查原始源码。
其次，它们允许 TypeScript 与未针对 TypeScript 开发的现有 JavaScript 库进行互操作。
最后，还有一个常常被低估的好处：在由 TypeScript 支持的编辑器中，TypeScript _和_ JavaScript 用户都能从这些声明文件中受益，从而获得更出色的自动补全等体验。

遗憾的是，以往 [`declaration`](/tsconfig#declaration) 无法与允许混合输入 TypeScript 和 JavaScript 文件的 [`allowJs`](/tsconfig#allowJs) 标志搭配使用。
这是一个令人沮丧的限制，因为它意味着用户在迁移代码库时无法使用 [`declaration`](/tsconfig#declaration) 标志，即便这些代码已经添加了 JSDoc 注解。
TypeScript 3.7 改变了这一点，允许这两个选项协同工作！

该特性最深远的影响或许稍显微妙：在 TypeScript 3.7 中，用户可以使用带有 JSDoc 注解的 JavaScript 编写库，并同时为 TypeScript 用户提供支持。

其工作机制是：当使用 [`allowJs`](/tsconfig#allowJs) 时，TypeScript 会尽最大努力进行静态分析以理解常见的 JavaScript 模式；然而，某些模式在 JavaScript 中的表达方式并不一定与它们在 TypeScript 中的对应写法完全相同。
当开启 [`declaration`](/tsconfig#declaration) 生成时，TypeScript 会找到最佳方式，将 JSDoc 注释和 CommonJS 导出转换为输出 `.d.ts` 文件中合法的类型声明等内容。

例如，以下代码片段：

```js
const assert = require('assert')

module.exports.blurImage = blurImage

/**
 * 根据输入缓冲区生成模糊图像。
 *
 * @param input {Uint8Array}
 * @param width {number}
 * @param height {number}
 */
function blurImage(input, width, height) {
  const numPixels = width * height * 4
  assert(input.length === numPixels)
  const result = new Uint8Array(numPixels)

  // TODO

  return result
}
```

将生成如下的 `.d.ts` 文件：

```ts
/**
 * 根据输入缓冲区生成模糊图像。
 *
 * @param input {Uint8Array}
 * @param width {number}
 * @param height {number}
 */
export function blurImage(
  input: Uint8Array,
  width: number,
  height: number,
): Uint8Array
```

除了带有 `@param` 标签的基础函数之外，更复杂的场景也能得到良好支持，例如以下示例：

```js
/**
 * @callback Job
 * @returns {void}
 */

/** 将工作任务排队 */
export class Worker {
  constructor(maxDepth = 10) {
    this.started = false
    this.depthLimit = maxDepth
    /**
     * 注意：排队的任务可能会向队列添加更多项
     * @type {Job[]}
     */
    this.queue = []
  }
  /**
   * 向队列中添加一个工作项
   * @param {Job} work
   */
  push(work) {
    if (this.queue.length + 1 > this.depthLimit) throw new Error('Queue full!')
    this.queue.push(work)
  }
  /**
   * 如果队列尚未启动，则启动队列
   */
  start() {
    if (this.started) return false
    this.started = true
    while (this.queue.length) {
      ;/** @type {Job} */ (this.queue.shift())()
    }
    return true
  }
}
```

将被转换为如下的 `.d.ts` 文件：

```ts
/**
 * @callback Job
 * @returns {void}
 */
/** 将工作任务排队 */
export class Worker {
  constructor(maxDepth?: number)
  started: boolean
  depthLimit: number
  /**
   * 注意：排队的任务可能会向队列添加更多项
   * @type {Job[]}
   */
  queue: Job[]
  /**
   * 向队列中添加一个工作项
   * @param {Job} work
   */
  push(work: Job): void
  /**
   * 如果队列尚未启动，则启动队列
   */
  start(): boolean
}
export type Job = () => void
```

请注意，在同时使用这些标志时，TypeScript 并不一定要降级编译 `.js` 文件。
如果你仅仅希望 TypeScript 创建 `.d.ts` 文件，可以使用 [`emitDeclarationOnly`](/tsconfig#emitDeclarationOnly) 编译器选项。

欲了解更多详情，请[查看原始 Pull Request](https://github.com/microsoft/TypeScript/pull/32372)。

## `useDefineForClassFields` 标志与 `declare` 属性修饰符

当年 TypeScript 初次实现公共类字段时，我们尽最大努力做出的假设是：以下代码

```ts
class C {
  foo = 100
  bar: string
}
```

等同于构造函数体内类似的赋值操作：

```ts
class C {
  constructor() {
    this.foo = 100
  }
}
```

不幸的是，虽然在提案早期这似乎是它的演进方向，但公共类字段在标准化时极大概率会采用不同的语义。
相反，原始代码示例可能需要脱糖（de-sugar）为更接近以下形式的代码：

```ts
class C {
  constructor() {
    Object.defineProperty(this, 'foo', {
      enumerable: true,
      configurable: true,
      writable: true,
      value: 100,
    })
    Object.defineProperty(this, 'bar', {
      enumerable: true,
      configurable: true,
      writable: true,
      value: void 0,
    })
  }
}
```

尽管 TypeScript 3.7 默认没有更改任何现有的生成代码，但我们一直在循序渐进地推出变更，以帮助用户缓解未来可能发生的破坏性影响。
我们提供了一个名为 [`useDefineForClassFields`](/tsconfig#useDefineForClassFields) 的新标志，用以启用这种生成模式以及一些新的检查逻辑。

其中最大的两项变化如下：

- 声明使用 `Object.defineProperty` 进行初始化。
- 声明_总是_被初始化为 `undefined`，即使它们没有初始值设定项。

这可能会对使用了继承的现有代码带来相当大的连带影响。首先，基类中的 `set` 访问器将不会被触发——它们会被完全覆盖：

```ts
class Base {
  set data(value: string) {
    console.log('data changed to ' + value)
  }
}

class Derived extends Base {
  // 当使用 'useDefineForClassFields' 时，
  // 不再触发 'console.log'。
  data = 10
}
```

其次，使用类字段来特化基类属性也将不再有效：

```ts
interface Animal {
  animalStuff: any
}
interface Dog extends Animal {
  dogStuff: any
}

class AnimalHouse {
  resident: Animal
  constructor(animal: Animal) {
    this.resident = animal
  }
}

class DogHouse extends AnimalHouse {
  // 当使用 'useDefineForClassFields' 时，
  // 在调用 'super()' 之后将 'resident' 初始化为 'undefined'！
  resident: Dog

  constructor(dog: Dog) {
    super(dog)
  }
}
```

归结起来就是：将属性与访问器混合使用会导致问题，而在没有初始值设定项的情况下重新声明属性同样会出问题。

为了检测访问器相关的问题，TypeScript 3.7 现在会在 `.d.ts` 文件中输出 `get`/`set` 访问器，以便 TypeScript 能够检查被覆写的访问器。

受类字段变更影响的代码可以通过将字段初始值设定项转换为构造函数体内的赋值来规避该问题：

```ts
class Base {
  set data(value: string) {
    console.log('data changed to ' + value)
  }
}

class Derived extends Base {
  constructor() {
    this.data = 10
  }
}
```

为了帮助缓解第二个问题，你可以添加显式的初始值设定项，或者添加 `declare` 修饰符以表明该属性不应产生任何生成代码：

```ts
interface Animal {
  animalStuff: any
}
interface Dog extends Animal {
  dogStuff: any
}

class AnimalHouse {
  resident: Animal
  constructor(animal: Animal) {
    this.resident = animal
  }
}

class DogHouse extends AnimalHouse {
  declare resident: Dog
  //  ^^^^^^^
  // 'resident' 现在拥有 'declare' 修饰符，
  // 并且不会生成任何输出代码。

  constructor(dog: Dog) {
    super(dog)
  }
}
```

目前 [`useDefineForClassFields`](/tsconfig#useDefineForClassFields) 仅在目标为 ES5 及更高版本时可用，因为 ES3 中不存在 `Object.defineProperty`。
若要进行类似的问题检查，你可以创建一个以 ES5 为目标并使用 [`noEmit`](/tsconfig#noEmit) 的独立项目，以避免完整的构建。

欲了解更多信息，请[查看这些变更的原始 Pull Request](https://github.com/microsoft/TypeScript/pull/33509)。

我们强烈建议用户尝试使用 [`useDefineForClassFields`](/tsconfig#useDefineForClassFields) 标志，并在我们的议题追踪器或下方评论中反馈意见。
这也包括关于采用该标志难易程度的反馈，以便我们了解如何使迁移更加平滑。

## 基于项目引用的免构建编辑体验

TypeScript 的项目引用（project references）为我们提供了一种拆分代码库的简便方式，从而带来更快的编译速度。
然而遗憾的是，编辑一个依赖项尚未构建（或构建输出已过期）的项目时，编辑体验往往并不理想。

在 TypeScript 3.7 中，当打开一个包含依赖项的项目时，TypeScript 会自动改用源 `.ts`/`.tsx` 文件。
这意味着使用项目引用的项目现在将获得更出色的编辑体验，语义操作始终保持最新且“开箱即用”。
你可以通过编译器选项 [`disableSourceOfProjectReferenceRedirect`](/tsconfig#disableSourceOfProjectReferenceRedirect) 禁用此行为，这在非常庞大的项目中可能比较适用，因为在超大项目中该变更可能会对编辑性能产生影响。

你可以[通过查阅其 Pull Request 来了解有关此变更的更多信息](https://github.com/microsoft/TypeScript/pull/32028)。

## 未调用的函数检查

忘记调用函数是一种常见且危险的错误，尤其是当函数没有参数，或者其命名方式让人误以为它是一个属性而非函数时：

```ts
interface User {
  isAdministrator(): boolean
  notify(): void
  doNotDisturb?(): boolean
}

// 稍后...

// 损坏的代码，请勿使用！
function doAdminThing(user: User) {
  // 糟糕！
  if (user.isAdministrator) {
    sudo()
    editTheConfiguration()
  } else {
    throw new AccessDeniedError('User is not an admin')
  }
}
```

在此处，我们忘记调用 `isAdministrator`，导致代码错误地允许非管理员用户修改配置！

在 TypeScript 3.7 中，这会被识别为一个潜在的错误：

```ts
function doAdminThing(user: User) {
    if (user.isAdministrator) {
    //  ~~~~~~~~~~~~~~~~~~~~
    // 错误！此条件将始终返回 true，因为该函数始终已定义。
    //        你是不是想调用它？
```

此项检查属于破坏性变更，但正因如此，该检查非常保守。
此错误仅在 `if` 条件中抛出，且在以下情况下不会报错：属性为可选属性、未开启 [`strictNullChecks`](/tsconfig#strictNullChecks)，或者后续在 `if` 体内调用了该函数：

```ts
interface User {
  isAdministrator(): boolean
  notify(): void
  doNotDisturb?(): boolean
}

function issueNotification(user: User) {
  if (user.doNotDisturb) {
    // 正常，属性是可选的
  }
  if (user.notify) {
    // 正常，调用了该函数
    user.notify()
  }
}
```

如果你确实打算在不调用函数的情况下测试它，可以修改其定义以包含 `undefined`/`null`，或者使用 `!!` 写成类似 `if (!!user.isAdministrator)` 的形式，以表明该类型强制转换是有意的。

我们非常感谢 GitHub 用户 [@jwbay](https://github.com/jwbay)，他主动创建了[概念验证（PoC）](https://github.com/microsoft/TypeScript/pull/32802)，并不断迭代为我们带来了[当前版本](https://github.com/microsoft/TypeScript/pull/33178)。

## TypeScript 文件支持 `// @ts-nocheck`

TypeScript 3.7 允许我们在 TypeScript 文件的顶部添加 `// @ts-nocheck` 注释来禁用语义检查。
以往，该注释仅在启用 [`checkJs`](/tsconfig#checkJs) 的 JavaScript 源文件中有效，但我们已将支持扩展到 TypeScript 文件，以便让所有用户的迁移工作更加轻松。

## 分号格式化选项

TypeScript 的内置格式化工具现已支持在根据 JavaScript 自动分号插入（ASI）规则尾随分号可选的位置进行分号的插入与移除。该设置现已在 [Visual Studio Code Insiders](https://code.visualstudio.com/insiders/) 中提供，并将在 Visual Studio 16.4 Preview 2 的“工具选项”菜单中提供。

<img width="833" alt="New semicolon formatter option in VS Code" src="https://user-images.githubusercontent.com/3277153/65913194-10066e80-e395-11e9-8a3a-4f7305c397d5.png">

选择值 "insert" 或 "remove" 还会影响 TypeScript 服务提供的自动导入、提取类型以及其他生成代码的格式。保持默认值 "ignore" 则会使生成的代码匹配当前文件中检测到的分号偏好设置。

## 3.7 破坏性变更

### DOM 变更

[更新了 `lib.dom.d.ts` 中的类型](https://github.com/microsoft/TypeScript/pull/33627)。
这些改动主要是与可空性相关的正确性修正，但具体影响最终取决于你的代码库。

### 类字段变更应对措施

[如上所述](#the-usedefineforclassfields-flag-and-the-declare-property-modifier)，TypeScript 3.7 会在 `.d.ts` 文件中输出 `get`/`set` 访问器，这可能会给使用旧版 TypeScript（如 3.5 及更早版本）的使用方带来破坏性变更。
TypeScript 3.6 用户不会受到影响，因为该版本已针对此特性进行了前瞻性兼容。

虽然开启 [`useDefineForClassFields`](/tsconfig#useDefineForClassFields) 标志本身不属于破坏性变更，但在以下情况下可能会引发破坏：

- 在派生类中用属性声明覆写访问器
- 重新声明没有初始值设定项的属性声明

若要了解完整的影响，请阅读[上文关于 `useDefineForClassFields` 标志的小节](#the-usedefineforclassfields-flag-and-the-declare-property-modifier)。

### 函数真值检查

如上所述，当函数在 `if` 语句条件中看似未被调用时，TypeScript 现在会报错。
当在 `if` 条件中检查函数类型时，除非满足以下任一条件，否则都会报错：

- 被检查的值来自可选属性
- 禁用了 [`strictNullChecks`](/tsconfig#strictNullChecks)
- 该函数后续在 `if` 体内被调用

### 本地与导入的类型声明现在会产生冲突

由于以往的一个 bug，以下结构之前在 TypeScript 中是允许的：

```ts
// ./someOtherModule.ts
interface SomeType {
  y: string
}

// ./myModule.ts
import { SomeType } from './someOtherModule'
export interface SomeType {
  x: number
}

function fn(arg: SomeType) {
  console.log(arg.x) // 错误！'SomeType' 上不存在 'x'
}
```

在此处，`SomeType` 似乎同时来源于 `import` 声明和本地 `interface` 声明。
或许令人惊讶的是，在模块内部，`SomeType` 唯独指代通过 `import` 导入的定义，而本地声明的 `SomeType` 只有在被其他文件导入时才可用。
这种行为非常令人困惑，我们对现实中极少数此类代码案例的审查表明，开发者通常误以为发生的是其他行为。

在 TypeScript 3.7 中，[这现在会被正确地识别为重复标识符错误](https://github.com/microsoft/TypeScript/pull/31231)。
正确的修复方案取决于作者的原始意图，需要具体情况具体分析。
通常情况下，命名冲突是无意的，最佳修复方案是重命名导入的类型。
如果意图是扩充导入的类型，则应该编写规范的模块扩充（module augmentation）。

### 3.7 API 变更

为了支持上述递归类型别名模式，`TypeReference` 接口中移除了 `typeArguments` 属性。用户应改为使用 `TypeChecker` 实例上的 `getTypeArguments` 函数。
