---
title: TypeScript 4.2
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-4-2.html
oneline: TypeScript 4.2 发布说明
---

## 更智能的类型别名保留

TypeScript 提供了一种为类型声明新名称的方式，称为类型别名（type alias）。
如果你编写的一组函数都要处理 `string | number | boolean`，可以通过编写类型别名来避免重复书写这些类型。

```ts
type BasicPrimitive = number | string | boolean
```

在打印类型时，TypeScript 一直以来都是通过一套规则和推测来决定何时复用类型别名。
例如下面这段代码：

```ts
export type BasicPrimitive = number | string | boolean

export function doStuff(value: BasicPrimitive) {
  let x = value
  return x
}
```

如果我们在 Visual Studio、Visual Studio Code 或 [TypeScript 演练场](https://www.typescriptlang.org/play?ts=4.1.3#code/KYDwDg9gTgLgBDAnmYcBCBDAzgSwMYAKUOAtjjDgG6oC8cAdgK4kBGwUcAPnFjMfQHMucFhAgAbYBnoBuAFBzQkWHABmjengoR6cACYQAyjEarVACkoZxjYAC502fEVLkqwAJRwA3nLj+4SXgQODorG2B5ALgoYBMoXRB5AF8gA)等编辑器中将鼠标悬停在 `x` 上，会弹出一个快速信息提示面板，显示类型为 `BasicPrimitive`。
同样，如果为该文件生成声明文件（`.d.ts` 输出），TypeScript 也会表明 `doStuff` 的返回类型是 `BasicPrimitive`。

然而，如果我们返回 `BasicPrimitive` 或 `undefined` 会发生什么？

```ts
export type BasicPrimitive = number | string | boolean

export function doStuff(value: BasicPrimitive) {
  if (Math.random() < 0.5) {
    return undefined
  }

  return value
}
```

我们可以[在 TypeScript 4.1 演练场中](https://www.typescriptlang.org/play?ts=4.1.3#code/KYDwDg9gTgLgBDAnmYcBCBDAzgSwMYAKUOAtjjDgG6oC8cAdgK4kBGwUcAPnFjMfQHMucFhAgAbYBnoBuALAAoRQHplcABIRqHCPTgByACYQAyjEYAzC-pHBxEAO4IIPYKgcALDPAAqyYCZ4xGDwhjhYYOIYiFhwFtAIHqhQwOZQekgoAHQqagDqqGQCHvBe1HCgKHgwwIZw5M5wYPzw2Lm5cJ2YuITEZBTl3Iz0hsAWOPS1HR0sjPBs9k5+KIHB8AAsWQBMADT18BO8UnVhEVExcG0Kqh2dTKzswrz8QtyiElJ6QyNjE1PXykUlWg8Asw2qOF0cGMZksFgAFJQMOJGMAAFzobD4IikchUYAASjgAG9FJ1yTgLHB4QBZbweLJQaTGEjwokAHjgAAYsgBWImkhTk4WdFJpPTDUbjSaGeRC4UAX0UZOFYsY6TgSJRwDlcAVQA)看到实际效果。
虽然我们可能希望 TypeScript 将 `doStuff` 的返回类型显示为 `BasicPrimitive | undefined`，但它实际显示的却是 `string | number | boolean | undefined`！
这是怎么回事呢？

这与 TypeScript 内部表示类型的方式有关。
当由一个或多个联合类型创建联合类型时，TypeScript 总是会将这些类型*规范化（normalize）*为一个新的扁平联合类型——但这样做会丢失原始信息。
类型检查器必须在 `string | number | boolean | undefined` 的每种类型组合中进行查找，以推断可能使用了哪些类型别名；即便如此，也可能存在多个指向 `string | number | boolean` 的类型别名。

在 TypeScript 4.2 中，内部机制变得更加智能。
我们通过保留类型最初的书写方式和构造过程的部分信息，来追踪类型的构造方式。
我们还会追踪并区分指向其他别名实例的类型别名！

能够根据你在代码中的使用方式回显类型，意味着作为 TypeScript 用户，你可以避免看到那些冗长臃肿的类型展示。这通常也会带来更好的 `.d.ts` 文件输出、更清晰的错误提示，以及编辑器内更友好的悬停信息和签名帮助。
这有助于让初学者感到 TypeScript 更加易于上手。

欲了解更多信息，请查看[改进保留联合类型别名多种场景的首个 Pull Request](https://github.com/microsoft/TypeScript/pull/42149)，以及[保留间接别名的第二个 Pull Request](https://github.com/microsoft/TypeScript/pull/42284)。

## 元组类型中更灵活的 Rest 元素

在 TypeScript 中，元组类型用于为具有固定长度和特定元素类型的数组建模。

```ts
// A tuple that stores a pair of numbers
let a: [number, number] = [1, 2]

// A tuple that stores a string, a number, and a boolean
let b: [string, number, boolean] = ['hello', 42, true]
```

随着时间的推移，TypeScript 的元组类型变得越来越强大，因为它们还被用于对 JavaScript 中的参数列表等结构进行建模。
因此，元组类型不仅支持可选元素和 rest 元素，还可以带有标签以改善工具支持和可读性。

```ts twoslash
// A tuple that has either one or two strings.
let c: [string, string?] = ['hello']
c = ['hello', 'world']

// A labeled tuple that has either one or two strings.
let d: [first: string, second?: string] = ['hello']
d = ['hello', 'world']

// A tuple with a *rest element* - holds at least 2 strings at the front,
// and any number of booleans at the back.
let e: [string, string, ...boolean[]]

e = ['hello', 'world']
e = ['hello', 'world', false]
e = ['hello', 'world', true, false, true]
```

在 TypeScript 4.2 中，rest 元素的使用方式得到了专门扩展。
在先前的版本中，TypeScript 只允许在元组类型的最末尾位置放置 `...rest` 元素。

而现在，除了少数限制外，rest 元素可以出现在元组内的*任意位置*。

```ts twoslash
let foo: [...string[], number]

foo = [123]
foo = ['hello', 123]
foo = ['hello!', 'hello!', 'hello!', 123]

let bar: [boolean, ...string[], boolean]

bar = [true, false]
bar = [true, 'some text', false]
bar = [true, 'some', 'separated', 'text', false]
```

唯一的限制是：rest 元素可以放置在元组中的任意位置，只要其后不紧跟另一个可选元素或 rest 元素即可。
换句话说，每个元组中只能有一个 rest 元素，且 rest 元素之后不能出现可选元素。

```ts twoslash
// @errors: 1265 1266
interface Clown {
  /*...*/
}
interface Joker {
  /*...*/
}

let StealersWheel: [...Clown[], 'me', ...Joker[]]

let StringsAndMaybeBoolean: [...string[], boolean?]
```

这些非末尾的 rest 元素可以用来为接收任意数量的前置参数、后跟若干固定参数的函数建模。

```ts twoslash
declare function doStuff(
  ...args: [...names: string[], shouldCapitalize: boolean]
): void

doStuff(/*shouldCapitalize:*/ false)
doStuff('fee', 'fi', 'fo', 'fum', /*shouldCapitalize:*/ true)
```

尽管 JavaScript 本身没有任何语法来表达前置 rest 形参，但我们仍然可以通过使用*包含前置 rest 元素的元组类型*来声明 `...args` rest 形参，从而将 `doStuff` 声明为接收前置参数的函数。
这有助于为现实中现有的许多 JavaScript 模式建模！

欲了解更多详情，请[参见原始 Pull Request](https://github.com/microsoft/TypeScript/pull/41544)。

## 更严格的 `in` 运算符类型检查

在 JavaScript 中，如果将非对象类型作为 `in` 运算符的右操作数，会在运行时抛出错误。
TypeScript 4.2 确保在设计时即可捕获该错误。

```ts twoslash
// @errors: 2361 2322
'foo' in 42
```

在大多数情况下，该检查相当保守，因此如果你收到了相关错误，通常意味着代码本身存在问题。

非常感谢外部贡献者 [Jonas Hübotter](https://github.com/jonhue) 提交的 [Pull Request](https://github.com/microsoft/TypeScript/pull/41928)！

## `--noPropertyAccessFromIndexSignature`

当 TypeScript 最初引入索引签名时，你只能通过“括号”元素访问语法（如 `person["name"]`）来获取由索引签名声明的属性。

```ts twoslash
interface SomeType {
  /** This is an index signature. */
  [propName: string]: any
}

function doStuff(value: SomeType) {
  let x = value['someProperty']
}
```

在需要处理具有任意属性的对象时，这种方式显得较为繁琐。
例如，设想一个常见的 API 场景，开发者可能会误在属性名末尾多拼写一个 `s`：

```ts twoslash
interface Options {
  /** File patterns to be excluded. */
  exclude?: string[]

  /**
   * It handles any extra properties that we haven't declared as type 'any'.
   */
  [x: string]: any
}

function processOptions(opts: Options) {
  // Notice we're *intentionally* accessing `excludes`, not `exclude`
  if (opts.excludes) {
    console.error('The option `excludes` is not valid. Did you mean `exclude`?')
  }
}
```

为了方便处理这类情况，TypeScript 在早前版本中允许当类型包含字符串索引签名时，使用“点”属性访问语法（如 `person.name`）。
这也使得将现有 JavaScript 代码迁移到 TypeScript 变得更加轻松。

然而，放宽这一限制也意味着更容易拼错显式声明的属性。

```ts twoslash
interface Options {
  /** File patterns to be excluded. */
  exclude?: string[]

  /**
   * It handles any extra properties that we haven't declared as type 'any'.
   */
  [x: string]: any
}
// ---cut---
function processOptions(opts: Options) {
  // ...

  // Notice we're *accidentally* accessing `excludes` this time.
  // Oops! Totally valid.
  for (const excludePattern of opts.excludes) {
    // ...
  }
}
```

在某些情况下，用户希望显式启用索引签名访问——即当点语法访问的属性未对应任何显式属性声明时，编译器能够报错。

为此，TypeScript 引入了一个名为 [`noPropertyAccessFromIndexSignature`](/tsconfig#noPropertyAccessFromIndexSignature) 的新编译选项。
在此模式下，将恢复 TypeScript 以往报错的行为。
该新选项不属于 [`strict`](/tsconfig#strict) 严格模式家族，因为我们认为某些代码库比其他代码库更能从中受益。

你可以通过阅读对应的 [Pull Request](https://github.com/microsoft/TypeScript/pull/40171/) 详细了解该特性。
同时我们还要非常感谢提交此 Pull Request 的 [Wenlu Wang](https://github.com/Kingwl)！

## `abstract` 构造签名

TypeScript 允许我们将类标记为*抽象（abstract）*。
这告诉 TypeScript 该类仅用于被继承，并且其实例化需要子类填补特定的未实现成员。

```ts twoslash
// @errors: 2511
abstract class Shape {
  abstract getArea(): number
}

new Shape()

class Square extends Shape {
  #sideLength: number

  constructor(sideLength: number) {
    super()
    this.#sideLength = sideLength
  }

  getArea() {
    return this.#sideLength ** 2
  }
}

// Works fine.
new Square(42)
```

为了确保无法直接 `new` 实例化 `abstract` 类的限制得到一致贯彻，你不能将 `abstract` 类赋值给任何期望普通构造签名的类型。

```ts twoslash
// @errors: 2322
abstract class Shape {
  abstract getArea(): number
}
// ---cut---
interface HasArea {
  getArea(): number
}

let Ctor: new () => HasArea = Shape
```

当我们试图运行类似 `new Ctor` 的代码时，这无疑是正确的行为；但如果我们的目的是编写 `Ctor` 的子类，这种限制就显得过于严格了。

```ts twoslash
// @errors: 2345
abstract class Shape {
  abstract getArea(): number
}

interface HasArea {
  getArea(): number
}

function makeSubclassWithArea(Ctor: new () => HasArea) {
  return class extends Ctor {
    getArea() {
      return 42
    }
  }
}

let MyShape = makeSubclassWithArea(Shape)
```

它在配合诸如 `InstanceType` 这样的内置工具类型使用时也会出现问题：

```ts twoslash
// @errors: 2344
abstract class Shape {
  abstract getArea(): number
}
// ---cut---
type MyInstance = InstanceType<typeof Shape>
```

为此，TypeScript 4.2 允许你在构造签名上指定 `abstract` 修饰符。

```ts twoslash {5}
abstract class Shape {
  abstract getArea(): number
}
// ---cut---
interface HasArea {
  getArea(): number
}

// Works!
let Ctor: abstract new () => HasArea = Shape
```

在构造签名上添加 `abstract` 修饰符表明可以传入 `abstract` 构造函数。
这并不会阻止你传入其他“具体（concrete）”的类或构造函数——它只是表明无意直接调用该构造函数，因此传入任意一种类都是安全的。

该特性使我们能够以支持抽象类的方式编写*混入工厂（mixin factory）*。
例如在以下代码片段中，我们可以将混入函数 `withStyles` 与 `abstract` 类 `SuperClass` 一起使用：

```ts twoslash
abstract class SuperClass {
  abstract someMethod(): void
  badda() {}
}

type AbstractConstructor<T> = abstract new (...args: any[]) => T

function withStyles<T extends AbstractConstructor<object>>(Ctor: T) {
  abstract class StyledClass extends Ctor {
    getStyles() {
      // ...
    }
  }
  return StyledClass
}

class SubClass extends withStyles(SuperClass) {
  someMethod() {
    this.someMethod()
  }
}
```

请注意，`withStyles` 演示了一条特定规则：当一个类（如 `StyledClass`）继承自一个泛型且受抽象构造函数约束的值（如 `Ctor`）时，该类也必须声明为 `abstract`。
这是因为无法预知传入的类是否包含*更多*的抽象成员，因此无法确定子类是否实现了所有抽象成员。

你可以[在 Pull Request 中](https://github.com/microsoft/TypeScript/pull/36392)了解关于抽象构造签名的更多内容。

## 使用 `--explainFiles` 了解项目结构

TypeScript 用户经常会遇到一个出奇常见的疑问：“为什么 TypeScript 会包含这个文件？”。
推断项目包含哪些文件实际上是一个复杂的过程：某个特定版本的 `lib.d.ts` 为何被使用、`node_modules` 中的某些文件为何被包含，以及明明指定了 [`exclude`](/tsconfig#exclude) 某些文件为何仍然被包含在内，背后都有很多可能的原因。

为此，TypeScript 现在提供了一个 [`explainFiles`](/tsconfig#explainFiles) 编译选项。

```sh
tsc --explainFiles
```

使用此选项时，TypeScript 编译器将输出非常详细的信息，说明某个文件为何会最终包含在程序中。
为了更方便查看，你可以将输出重定向到文件，或者通过管道传递给便于查看的工具程序。

```sh
# Forward output to a text file
tsc --explainFiles > explanation.txt

# Pipe output to a utility program like `less`, or an editor like VS Code
tsc --explainFiles | less

tsc --explainFiles | code -
```

通常情况下，输出会首先列出包含各个 `lib.d.ts` 文件的原因，接着是本地文件，最后是 `node_modules` 文件。

```
TS_Compiler_Directory/4.2.2/lib/lib.es5.d.ts
  Library referenced via 'es5' from file 'TS_Compiler_Directory/4.2.2/lib/lib.es2015.d.ts'
TS_Compiler_Directory/4.2.2/lib/lib.es2015.d.ts
  Library referenced via 'es2015' from file 'TS_Compiler_Directory/4.2.2/lib/lib.es2016.d.ts'
TS_Compiler_Directory/4.2.2/lib/lib.es2016.d.ts
  Library referenced via 'es2016' from file 'TS_Compiler_Directory/4.2.2/lib/lib.es2017.d.ts'
TS_Compiler_Directory/4.2.2/lib/lib.es2017.d.ts
  Library referenced via 'es2017' from file 'TS_Compiler_Directory/4.2.2/lib/lib.es2018.d.ts'
TS_Compiler_Directory/4.2.2/lib/lib.es2018.d.ts
  Library referenced via 'es2018' from file 'TS_Compiler_Directory/4.2.2/lib/lib.es2019.d.ts'
TS_Compiler_Directory/4.2.2/lib/lib.es2019.d.ts
  Library referenced via 'es2019' from file 'TS_Compiler_Directory/4.2.2/lib/lib.es2020.d.ts'
TS_Compiler_Directory/4.2.2/lib/lib.es2020.d.ts
  Library referenced via 'es2020' from file 'TS_Compiler_Directory/4.2.2/lib/lib.esnext.d.ts'
TS_Compiler_Directory/4.2.2/lib/lib.esnext.d.ts
  Library 'lib.esnext.d.ts' specified in compilerOptions

... More Library References...

foo.ts
  Matched by include pattern '**/*' in 'tsconfig.json'
```

目前，我们不对输出格式做任何保证——它可能会随时间变化。
顺便一提，如果你有任何建议，我们非常乐意进一步改进这一格式！

欲了解更多信息，请[查看原始 Pull Request](https://github.com/microsoft/TypeScript/pull/40011)！

## 逻辑表达式中未调用函数检查的改进

感谢 [Alex Tarasyuk](https://github.com/a-tarasyuk) 的进一步改进，TypeScript 的未调用函数检查现已扩展至 `&&` 和 `||` 表达式中。

在启用了 [`strictNullChecks`](/tsconfig#strictNullChecks) 的情况下，以下代码现在会报错：

```ts
function shouldDisplayElement(element: Element) {
  // ...
  return true
}

function getVisibleItems(elements: Element[]) {
  return elements.filter((e) => shouldDisplayElement && e.children.length)
  //                          ~~~~~~~~~~~~~~~~~~~~
  // This condition will always return true since the function is always defined.
  // Did you mean to call it instead.
}
```

欲了解更多详情，请[在此查看 Pull Request](https://github.com/microsoft/TypeScript/issues/40197)。

## 解构变量可显式标记为未使用

感谢 [Alex Tarasyuk](https://github.com/a-tarasyuk) 提交的另一个 Pull Request，你现在可以通过在解构变量名前添加下划线前缀（`_` 字符）来将其显式标记为未使用。

```ts
let [_first, second] = getValues()
```

以前，如果 `_first` 在后续代码中从未被使用，TypeScript 会在开启 [`noUnusedLocals`](/tsconfig#noUnusedLocals) 时报错。
现在，TypeScript 会识别出 `_first` 是有意以下划线命名的，表示无意使用该变量。

欲了解更多详情，请查看[完整的变更内容](https://github.com/microsoft/TypeScript/pull/41378)。

## 放宽可选属性与字符串索引签名之间的规则

字符串索引签名是一种为字典类对象定义类型的方式，允许使用任意键进行访问：

```ts twoslash
const movieWatchCount: { [key: string]: number } = {}

function watchMovie(title: string) {
  movieWatchCount[title] = (movieWatchCount[title] ?? 0) + 1
}
```

当然，对于字典中尚不存在的任何电影名称，`movieWatchCount[title]` 的值都将是 `undefined`（TypeScript 4.1 添加了 [`noUncheckedIndexedAccess`](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-1.html#checked-indexed-accesses---nouncheckedindexedaccess) 选项，使从这类索引签名中读取时类型包含 `undefined`）。
尽管很显然 `movieWatchCount` 中肯定存在某些未包含的字符串键，但由于 `undefined` 的存在，之前版本的 TypeScript 认为可选对象属性无法赋值给在其他方面兼容的索引签名。

```ts twoslash
type WesAndersonWatchCount = {
  'Fantastic Mr. Fox'?: number
  'The Royal Tenenbaums'?: number
  'Moonrise Kingdom'?: number
  'The Grand Budapest Hotel'?: number
}

declare const wesAndersonWatchCount: WesAndersonWatchCount
const movieWatchCount: { [key: string]: number } = wesAndersonWatchCount
//    ~~~~~~~~~~~~~~~ error!
// Type 'WesAndersonWatchCount' is not assignable to type '{ [key: string]: number; }'.
//    Property '"Fantastic Mr. Fox"' is incompatible with index signature.
//      Type 'number | undefined' is not assignable to type 'number'.
//        Type 'undefined' is not assignable to type 'number'. (2322)
```

TypeScript 4.2 允许这种赋值。但是，它*不*允许赋值类型中包含 `undefined` 的非可选属性，也不允许向特定键写入 `undefined`：

```ts twoslash
// @errors: 2322
type BatmanWatchCount = {
  'Batman Begins': number | undefined
  'The Dark Knight': number | undefined
  'The Dark Knight Rises': number | undefined
}

declare const batmanWatchCount: BatmanWatchCount

// Still an error in TypeScript 4.2.
const movieWatchCount: { [key: string]: number } = batmanWatchCount

// Still an error in TypeScript 4.2.
// Index signatures don't implicitly allow explicit `undefined`.
movieWatchCount["It's the Great Pumpkin, Charlie Brown"] = undefined
```

新规则同样不适用于数字索引签名，因为它们被假定为类似数组且密集的：

```ts twoslash
// @errors: 2322
declare let sortOfArrayish: { [key: number]: string }
declare let numberKeys: { 42?: string }

sortOfArrayish = numberKeys
```

你可以[通过阅读原始 PR](https://github.com/microsoft/TypeScript/pull/41921) 进一步了解这一变更。

## 声明缺失的辅助函数

感谢 [Alexander Tarasyuk](https://github.com/a-tarasyuk) 提交的[社区 Pull Request](https://github.com/microsoft/TypeScript/pull/41215)，我们现在拥有了一个基于调用点快速生成新函数和新方法的快速修复（Quick Fix）！

![调用未声明的函数 foo，并通过快速修复搭建新文件内容的示意图](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2021/01/addMissingFunction-4.2.gif)

## 破坏性改动

我们始终致力于在每个版本中尽可能减少破坏性改动。
TypeScript 4.2 包含一些破坏性改动，但我们相信这些改动在升级时是可控的。

### `lib.d.ts` 更新

与每个 TypeScript 版本一样，`lib.d.ts` 的声明（特别是为 Web 上下文生成的声明）发生了一些改动。
改动涉及多个方面，其中关于 `Intl` 和 `ResizeObserver` 的变更可能影响最大。

### `noImplicitAny` 错误适用于松散的 `yield` 表达式

当捕获了 `yield` 表达式的返回值，但 TypeScript 无法直接推断出其期望接收的类型时（即该 `yield` 表达式没有上下文类型），TypeScript 现在会报告隐式 `any` 错误。

```ts twoslash
// @errors: 7057
function* g1() {
  const value = yield 1
}

function* g2() {
  // No error.
  // The result of `yield 1` is unused.
  yield 1
}

function* g3() {
  // No error.
  // `yield 1` is contextually typed by 'string'.
  const value: string = yield 1
}

function* g4(): Generator<number, void, string> {
  // No error.
  // TypeScript can figure out the type of `yield 1`
  // from the explicit return type of `g4`.
  const value = yield 1
}
```

更多详情请参阅[对应的变更](https://github.com/microsoft/TypeScript/pull/41348)。

### 扩大未调用函数的检查范围

如前所述，在使用 [`strictNullChecks`](/tsconfig#strictNullChecks) 时，未调用函数检查现在会在 `&&` 和 `||` 表达式中一致生效。
这可能会引发新的编译报错，但通常表明现有代码中存在逻辑错误。

### JavaScript 中的类型实参不再被解析为类型实参

JavaScript 中原本就不允许使用类型实参，但在 TypeScript 4.2 中，解析器将以更符合规范的方式对其进行解析。
因此，在 JavaScript 文件中编写如下代码时：

```ts
f<T>(100)
```

TypeScript 会将其解析为如下 JavaScript：

```js
f < T > 100
```

如果你使用 TypeScript 的 API 来解析 JavaScript 文件中的类型结构（例如在尝试解析 Flow 文件时），这可能会对你产生影响。

关于具体检查内容的更多细节，请参阅 [Pull Request](https://github.com/microsoft/TypeScript/pull/41928)。

### 展开操作中元组大小的限制

在 TypeScript 中，可以通过任意展开语法（`...`）构造元组类型。

```ts
// Tuple types with spread elements
type NumStr = [number, string]
type NumStrNumStr = [...NumStr, ...NumStr]

// Array spread expressions
const numStr = [123, 'hello'] as const
const numStrNumStr = [...numStr, ...numStr] as const
```

有时这些元组类型可能会意外变得极其庞大，导致类型检查耗时过长。
为了防止类型检查过程挂起（这在编辑器场景下体验尤为糟糕），TypeScript 引入了一个限制器以避免执行所有不必要的工作。

你可以[查看此 Pull Request](https://github.com/microsoft/TypeScript/pull/42448) 了解更多详情。

### 导入路径中不能使用 `.d.ts` 扩展名

在 TypeScript 4.2 中，导入路径的扩展名中包含 `.d.ts` 现在属于错误。

```ts
// must be changed to something like
//   - "./foo"
//   - "./foo.js"
import { Foo } from './foo.d.ts'
```

相反，你的导入路径应当反映加载器在运行时的行为。
可以使用以下任意一种导入方式代替：

```ts
import { Foo } from './foo'
import { Foo } from './foo.js'
import { Foo } from './foo/index.js'
```

### 回退模板字面量推断

这项变更移除了 TypeScript 4.2 Beta 中的一项功能。
如果你尚未升级到上一个稳定版本之后，你将不会受到影响，但你可能仍对该改动感兴趣。

TypeScript 4.2 Beta 版本包含了一项针对模板字符串推断的改动。
在该改动中，模板字符串字面量要么被赋予模板字符串类型，要么简化为多个字符串字面量类型。
当赋值给可变变量时，这些类型随后会*拓宽（widen）*为 `string`。

```ts
declare const yourName: string

// 'bar' is constant.
// It has type '`hello ${string}`'.
const bar = `hello ${yourName}`

// 'baz' is mutable.
// It has type 'string'.
let baz = `hello ${yourName}`
```

这类似于字符串字面量推断的工作方式：

```ts
// 'bar' has type '"hello"'.
const bar = 'hello'

// 'baz' has type 'string'.
let baz = 'hello'
```

因此，我们原本认为让模板字符串表达式具有模板字符串类型会更具“一致性”；
然而从我们收集到的反馈来看，这并不总是符合预期的。

因此，我们回退了这一特性（以及潜在的破坏性改动）。
如果你*确实*希望模板字符串表达式获得类似字面量的类型，可以随时在其末尾添加 `as const`。

```ts
declare const yourName: string

// 'bar' has type '`hello ${string}`'.
const bar = `hello ${yourName}` as const
//                              ^^^^^^^^

// 'baz' has type 'string'.
const baz = `hello ${yourName}`
```

### `visitNode` 中的 `lift` 回调使用不同类型

TypeScript 包含一个接收 `lift` 函数的 `visitNode` 函数。
`lift` 现在期望接收 `readonly Node[]` 而非 `NodeArray<Node>`。
从技术上讲，这是一项 API 破坏性改动，你可以[在此](https://github.com/microsoft/TypeScript/pull/42000)了解更多详情。
