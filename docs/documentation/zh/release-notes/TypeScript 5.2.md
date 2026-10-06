---
title: TypeScript 5.2
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-5-2.html
oneline: TypeScript 5.2 发布说明
---

## `using` 声明与显式资源管理

TypeScript 5.2 增加了对 ECMAScript 即将推出的[显式资源管理](https://github.com/tc39/proposal-explicit-resource-management)（Explicit Resource Management）特性的支持。让我们一起了解这一特性的设计背景及其带来的能力。

在创建对象之后，通常需要执行某种“清理”工作。例如，可能需要关闭网络连接、删除临时文件，或者释放内存。

设想这样一个函数：它创建一个临时文件，对其进行读写等各种操作，随后将其关闭并删除。

```ts
import * as fs from 'fs'

export function doSomeWork() {
  const path = '.some_temp_file'
  const file = fs.openSync(path, 'w+')

  // use file...

  // Close the file and delete it.
  fs.closeSync(file)
  fs.unlinkSync(path)
}
```

这段代码没有问题，但如果我们需要提前退出该怎么办？

```ts
export function doSomeWork() {
  const path = '.some_temp_file'
  const file = fs.openSync(path, 'w+')

  // use file...
  if (someCondition()) {
    // do some more work...

    // Close the file and delete it.
    fs.closeSync(file)
    fs.unlinkSync(path)
    return
  }

  // Close the file and delete it.
  fs.closeSync(file)
  fs.unlinkSync(path)
}
```

我们开始看到重复的清理代码，而这些代码很容易被遗漏。此外，如果抛出错误，也无法保证文件一定会被关闭和删除。将代码包裹在 `try`/`finally` 块中可以解决这个问题：

```ts
export function doSomeWork() {
  const path = '.some_temp_file'
  const file = fs.openSync(path, 'w+')

  try {
    // use file...

    if (someCondition()) {
      // do some more work...
      return
    }
  } finally {
    // Close the file and delete it.
    fs.closeSync(file)
    fs.unlinkSync(path)
  }
}
```

尽管这种方式更加健壮，但也给代码增加了不少“噪声”。如果在 `finally` 块中添加更多清理逻辑，还可能踩到其他潜在陷阱——例如，前面的异常可能导致后续其他资源无法被正常释放。这正是[显式资源管理](https://github.com/tc39/proposal-explicit-resource-management)提案旨在解决的问题。该提案的核心思想是将资源释放（即我们尝试处理的这类清理工作）作为 JavaScript 中的一等公民概念来支持。

这一切始于引入一个新的内置 `symbol`：`Symbol.dispose`，我们可以创建包含以 `Symbol.dispose` 命名的方法的对象。为了方便起见，TypeScript 定义了一个新的全局类型 `Disposable` 来描述这些对象。

```ts
class TempFile implements Disposable {
  #path: string
  #handle: number

  constructor(path: string) {
    this.#path = path
    this.#handle = fs.openSync(path, 'w+')
  }

  // other methods

  [Symbol.dispose]() {
    // Close the file and delete it.
    fs.closeSync(this.#handle)
    fs.unlinkSync(this.#path)
  }
}
```

随后我们便可以调用这些方法：

```ts
export function doSomeWork() {
  const file = new TempFile('.some_temp_file')

  try {
    // ...
  } finally {
    file[Symbol.dispose]()
  }
}
```

单纯将清理逻辑移到 `TempFile` 本身并不能带来太多好处；我们基本上只是把所有的清理工作从 `finally` 块搬进了一个方法中，而这从来都是可行的。但是，为该方法提供一个众所周知的“统一名称”，意味着 JavaScript 能够在此基础上构建其他语言特性。

这就引出了该特性的重头戏之一：`using` 声明！`using` 是一个新的关键字，用于声明新的固定绑定，类似于 `const`。关键区别在于：使用 `using` 声明的变量，其 `Symbol.dispose` 方法会在所在作用域结束时被自动调用！

因此，我们的代码可以直接简化为：

```ts
export function doSomeWork() {
  using file = new TempFile('.some_temp_file')

  // use file...

  if (someCondition()) {
    // do some more work...
    return
  }
}
```

瞧——完全没有 `try`/`finally` 块！至少表面上看不到。在功能层面上，`using` 声明为我们完成的正是这一逻辑，但我们不再需要亲自编写繁琐的样板代码。

你可能熟悉 [C# 中的 `using` 声明](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-8.0/using)、[Python 中的 `with` 语句](https://docs.python.org/3/reference/compound_stmts.html#the-with-statement)或 [Java 中的 `try`-with-resources 声明](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html)。它们都与 JavaScript 新增的 `using` 关键字类似，提供了在作用域结束时显式释放对象的类似方式。

`using` 声明会在包含它们的作用域的最末尾、或在发生“提前返回”（例如 `return` 或抛出错误）之前执行清理。此外，它们的释放顺序遵循后进先出（FILO），类似于栈结构。

```ts
function loggy(id: string): Disposable {
  console.log(`Creating ${id}`)

  return {
    [Symbol.dispose]() {
      console.log(`Disposing ${id}`)
    },
  }
}

function func() {
  using a = loggy('a')
  using b = loggy('b')
  {
    using c = loggy('c')
    using d = loggy('d')
  }
  using e = loggy('e')
  return

  // Unreachable.
  // Never created, never disposed.
  using f = loggy('f')
}

func()
// Creating a
// Creating b
// Creating c
// Creating d
// Disposing d
// Disposing c
// Creating e
// Disposing e
// Disposing b
// Disposing a
```

`using` 声明具备容错机制；如果在作用域执行过程中抛出错误，该错误会在资源释放完成后重新抛出。另一方面，函数体本身可能按预期执行完毕，但 `Symbol.dispose` 却可能抛出异常。在这种情况下，该异常同样会被重新抛出。

但如果释放前的逻辑和释放过程中的逻辑同时抛出错误，会发生什么呢？针对这种情况，ECMAScript 引入了 `SuppressedError` 作为 `Error` 的新子类型。它包含一个 `suppressed` 属性（保存先前被压制的错误）和一个 `error` 属性（保存最后抛出的错误）。

```ts
class ErrorA extends Error {
  name = 'ErrorA'
}
class ErrorB extends Error {
  name = 'ErrorB'
}

function throwy(id: string) {
  return {
    [Symbol.dispose]() {
      throw new ErrorA(`Error from ${id}`)
    },
  }
}

function func() {
  using a = throwy('a')
  throw new ErrorB('oops!')
}

try {
  func()
} catch (e: any) {
  console.log(e.name) // SuppressedError
  console.log(e.message) // An error was suppressed during disposal.

  console.log(e.error.name) // ErrorA
  console.log(e.error.message) // Error from a

  console.log(e.suppressed.name) // ErrorB
  console.log(e.suppressed.message) // oops!
}
```

你可能已经注意到，以上示例中使用的都是同步方法。然而，很多资源释放都涉及*异步*操作，在继续执行其他代码之前，我们需要等待这些异步操作完成。

正因如此，规范还新增了 `Symbol.asyncDispose`，这也引出了另一项核心特性——`await using` 声明。它们与 `using` 声明类似，关键在于它们会查找需要被 `await` 的释放操作。它们调用由 `Symbol.asyncDispose` 命名的方法，不过也可以兼容具有 `Symbol.dispose` 的对象。为方便起见，TypeScript 还引入了一个名为 `AsyncDisposable` 的全局类型，用于描述任何具有异步释放方法的对象。

```ts
async function doWork() {
  // Do fake work for half a second.
  await new Promise((resolve) => setTimeout(resolve, 500))
}

function loggy(id: string): AsyncDisposable {
  console.log(`Constructing ${id}`)
  return {
    async [Symbol.asyncDispose]() {
      console.log(`Disposing (async) ${id}`)
      await doWork()
    },
  }
}

async function func() {
  await using a = loggy('a')
  await using b = loggy('b')
  {
    await using c = loggy('c')
    await using d = loggy('d')
  }
  await using e = loggy('e')
  return

  // Unreachable.
  // Never created, never disposed.
  await using f = loggy('f')
}

func()
// Constructing a
// Constructing b
// Constructing c
// Constructing d
// Disposing (async) d
// Disposing (async) c
// Constructing e
// Disposing (async) e
// Disposing (async) b
// Disposing (async) a
```

如果你期望他人以一致的方式执行清理逻辑，依据 `Disposable` 和 `AsyncDisposable` 来定义类型可以使你的代码更加易用。实际上，生态中已有大量现存类型拥有 `dispose()` 或 `close()` 方法。例如，Visual Studio Code API 就定义了[自己的 `Disposable` 接口](https://code.visualstudio.com/api/references/vscode-api#Disposable)。浏览器以及 Node.js、Deno、Bun 等运行时中的 API，也可能会选择针对已经具有清理方法的对象（如文件句柄、连接等）支持 `Symbol.dispose` 和 `Symbol.asyncDispose`。

对于库作者而言这或许很棒，但在某些应用场景下可能略显沉重。如果你需要做很多临时性的清理工作，为此专门创建一个新类型可能会带来过度抽象和最佳实践方面的疑虑。例如，再次回顾我们的 `TempFile` 示例：

```ts
class TempFile implements Disposable {
  #path: string
  #handle: number

  constructor(path: string) {
    this.#path = path
    this.#handle = fs.openSync(path, 'w+')
  }

  // other methods

  [Symbol.dispose]() {
    // Close the file and delete it.
    fs.closeSync(this.#handle)
    fs.unlinkSync(this.#path)
  }
}

export function doSomeWork() {
  using file = new TempFile('.some_temp_file')

  // use file...

  if (someCondition()) {
    // do some more work...
    return
  }
}
```

我们原本只想确保调用两个函数——但这是最佳书写方式吗？我们应该在构造函数中调用 `openSync`、还是创建 `open()` 方法、抑或是自己传入句柄？我们是否需要为每一个可能的操作都暴露方法，还是直接将属性公开？

这就引出了该特性的最后一个亮点：`DisposableStack` 和 `AsyncDisposableStack`。这些对象既适用于一次性清理，也适用于任意数量的组合清理工作。`DisposableStack` 是一个包含多种跟踪 `Disposable` 对象方法的对象，并且可以传入自定义函数来执行任意清理工作。我们还可以将其赋值给 `using` 变量，因为——你猜怎么着——_它们本身也是 `Disposable`_！因此，最初的示例可以改写为：

```ts
function doSomeWork() {
  const path = '.some_temp_file'
  const file = fs.openSync(path, 'w+')

  using cleanup = new DisposableStack()
  cleanup.defer(() => {
    fs.closeSync(file)
    fs.unlinkSync(path)
  })

  // use file...

  if (someCondition()) {
    // do some more work...
    return
  }

  // ...
}
```

在此，`defer()` 方法接收一个回调函数，当 `cleanup` 被释放时，该回调就会被执行。通常情况下，`defer`（以及 `DisposableStack` 的其他方法，如 `use` 和 `adopt`）应该在创建资源后立即调用。正如其名称所示，`DisposableStack` 会像栈一样按照后进先出的顺序释放其跟踪的所有内容，因此在创建值后立即调用 `defer` 有助于避免奇怪的依赖问题。`AsyncDisposableStack` 的工作方式与之类似，但它可以跟踪 `async` 函数与 `AsyncDisposable`，并且其自身也是一个 `AsyncDisposable`。

`defer` 方法在很多方面类似于 [Go](https://go.dev/tour/flowcontrol/12)、[Swift](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/statements/#Defer-Statement)、[Zig](https://ziglang.org/documentation/master/#defer)、[Odin](https://odin-lang.org/docs/overview/#defer-statement) 等语言中的 `defer` 关键字，其约定也是相通的。

由于该特性非常新，大多数运行时目前还不提供原生支持。要使用它，你需要为以下项提供运行时 polyfill：

- `Symbol.dispose`
- `Symbol.asyncDispose`
- `DisposableStack`
- `AsyncDisposableStack`
- `SuppressedError`

不过，如果你只对 `using` 和 `await using` 感兴趣，通常只需对内置的 `symbol` 提供 polyfill 即可。在多数情况下，如下简单代码便已足够：

```ts
Symbol.dispose ??= Symbol('Symbol.dispose')
Symbol.asyncDispose ??= Symbol('Symbol.asyncDispose')
```

此外，你还需要将编译目标 `target` 设置为 `es2022` 或更低版本，并将 `lib` 设置配置为包含 `"esnext"` 或 `"esnext.disposable"`。

```json
{
  "compilerOptions": {
    "target": "es2022",
    "lib": ["es2022", "esnext.disposable", "dom"]
  }
}
```

关于该特性的更多信息，[请查看 GitHub 上的相关工作](https://github.com/microsoft/TypeScript/pull/54505)！

## 装饰器元数据

TypeScript 5.2 实现了 [ECMAScript 即将推出的装饰器元数据（Decorator Metadata）特性](https://github.com/tc39/proposal-decorator-metadata)。

该特性的核心思想，是让装饰器能够轻松在其所修饰的类或类内部成员上创建与消费元数据。

当使用装饰器函数时，它们现在可以通过上下文（context）对象访问新增的 `metadata` 属性。`metadata` 属性只保存一个普通对象。由于 JavaScript 允许随意添加属性，它可以被用作由各个装饰器共同更新的字典对象。或者，由于同一个类中各个被装饰部分所拿到的 `metadata` 对象都是完全相同的，它还可以用作 `Map` 的键。在类内部或类上的所有装饰器运行完毕后，可以通过类上的 `Symbol.metadata` 访问该对象。

```ts
interface Context {
  name: string
  metadata: Record<PropertyKey, unknown>
}

function setMetadata(_target: any, context: Context) {
  context.metadata[context.name] = true
}

class SomeClass {
  @setMetadata
  foo = 123

  @setMetadata
  accessor bar = 'hello!'

  @setMetadata
  baz() {}
}

const ourMetadata = SomeClass[Symbol.metadata]

console.log(JSON.stringify(ourMetadata))
// { "bar": true, "baz": true, "foo": true }
```

这在许多不同的场景中都非常有用。元数据可以附加用于调试、序列化或通过装饰器实现依赖注入等多种用途。由于每个被装饰的类都会创建独立的元数据对象，框架既可以将其私下用作 `Map` 或 `WeakMap` 的键，也可以根据需要直接在其上附加属性。

例如，假设我们希望使用装饰器来跟踪在使用 `JSON.stringify` 时哪些属性和访问器是可序列化的，如下所示：

```ts
import { serialize, jsonify } from './serializer'

class Person {
  firstName: string
  lastName: string

  @serialize
  age: number

  @serialize
  get fullName() {
    return `${this.firstName} ${this.lastName}`
  }

  toJSON() {
    return jsonify(this)
  }

  constructor(firstName: string, lastName: string, age: number) {
    // ...
  }
}
```

在这里，我们的目的是只有标记了 `@serialize` 装饰器的 `age` 和 `fullName` 才会被序列化。为此我们定义了一个 `toJSON` 方法，它会直接调用 `jsonify`，而 `jsonify` 则使用了由 `@serialize` 创建的元数据。

以下是 `./serialize.ts` 模块可能的定义示例：

```ts
const serializables = Symbol()

type Context =
  | ClassAccessorDecoratorContext
  | ClassGetterDecoratorContext
  | ClassFieldDecoratorContext

export function serialize(_target: any, context: Context): void {
  if (context.static || context.private) {
    throw new Error('Can only serialize public instance members.')
  }
  if (typeof context.name === 'symbol') {
    throw new Error('Cannot serialize symbol-named properties.')
  }

  const propNames = ((context.metadata[serializables] as
    string[] | undefined) ??= [])
  propNames.push(context.name)
}

export function jsonify(instance: object): string {
  const metadata = instance.constructor[Symbol.metadata]
  const propNames = metadata?.[serializables] as string[] | undefined
  if (!propNames) {
    throw new Error('No members marked with @serialize.')
  }

  const pairStrings = propNames.map((key) => {
    const strKey = JSON.stringify(key)
    const strValue = JSON.stringify((instance as any)[key])
    return `${strKey}: ${strValue}`
  })

  return `{ ${pairStrings.join(', ')} }`
}
```

该模块使用了一个名为 `serializables` 的局部 `symbol` 来存储和检索标记为 `@serializable` 的属性名称。在每次调用 `@serializable` 时，它都会在元数据上存储这些属性名称的列表。当调用 `jsonify` 时，会从元数据中获取属性列表，并用于从实例中检索实际值，最终将这些名称和值序列化。

从技术上讲，使用 `symbol` 会使这些数据可被外部访问。另一种替代方案是使用 `WeakMap`，将元数据对象作为键。这可以保持数据的私有性，并且在这种情况下碰巧使用了更少的类型断言，但在其他方面非常相似。

```ts
const serializables = new WeakMap<object, string[]>()

type Context =
  | ClassAccessorDecoratorContext
  | ClassGetterDecoratorContext
  | ClassFieldDecoratorContext

export function serialize(_target: any, context: Context): void {
  if (context.static || context.private) {
    throw new Error('Can only serialize public instance members.')
  }
  if (typeof context.name !== 'string') {
    throw new Error('Can only serialize string properties.')
  }

  let propNames = serializables.get(context.metadata)
  if (propNames === undefined) {
    serializables.set(context.metadata, (propNames = []))
  }
  propNames.push(context.name)
}

export function jsonify(instance: object): string {
  const metadata = instance.constructor[Symbol.metadata]
  const propNames = metadata && serializables.get(metadata)
  if (!propNames) {
    throw new Error('No members marked with @serialize.')
  }
  const pairStrings = propNames.map((key) => {
    const strKey = JSON.stringify(key)
    const strValue = JSON.stringify((instance as any)[key])
    return `${strKey}: ${strValue}`
  })

  return `{ ${pairStrings.join(', ')} }`
}
```

需要说明的是，这些实现并未处理子类化与继承。这留给读者作为练习（你可能会发现其中一种写法在处理继承时比另一种更轻松！）。

由于该特性依然很新，大多数运行时目前还不提供原生支持。要使用它，你需要为 `Symbol.metadata` 提供 polyfill。在多数情况下，如下简单代码便已足够：

```ts
Symbol.metadata ??= Symbol('Symbol.metadata')
```

此外，你还需要将编译目标 `target` 设置为 `es2022` 或更低版本，并将 `lib` 设置配置为包含 `"esnext"` 或 `"esnext.decorators"`。

```json
{
  "compilerOptions": {
    "target": "es2022",
    "lib": ["es2022", "esnext.decorators", "dom"]
  }
}
```

感谢 [Oleksandr Tarasiuk](https://github.com/a-tarasyuk) 为 TypeScript 5.2 贡献了[装饰器元数据的实现](https://github.com/microsoft/TypeScript/pull/54657)！

<!-- TODO: Why is there a conditional type around the existence of `Symbol.metadata`? -->

## 具名与匿名元组元素混用

元组类型此前就已支持为每个元素提供可选的标签或名称。

```ts
type Pair<T> = [first: T, second: T]
```

这些标签不会改变你对它们的操作权限——它们纯粹用于提高可读性以及为工具链提供更好的支持。

然而，TypeScript 此前有一条规则：元组不能混用带标签和不带标签的元素。换言之，元组中的元素要么全都没有标签，要么全都必须有标签。

```ts
// ✅ fine - no labels
type Pair1<T> = [T, T]

// ✅ fine - all fully labeled
type Pair2<T> = [first: T, second: T]

// ❌ previously an error
type Pair3<T> = [first: T, T]
//                         ~
// Tuple members must all have names
// or all not have names.
```

对于剩余元素而言，这可能很繁琐，因为我们被迫不得不加上类似 `rest` 或 `tail` 这样的标签。

```ts
// ❌ previously an error
type TwoOrMore_A<T> = [first: T, second: T, ...T[]]
//                                          ~~~~~~
// Tuple members must all have names
// or all not have names.

// ✅
type TwoOrMore_B<T> = [first: T, second: T, rest: ...T[]]
```

这也意味着该限制必须在类型系统内部强制执行，从而导致 TypeScript 在某些情况下会丢失标签信息。

```ts
type HasLabels = [a: string, b: string]
type HasNoLabels = [number, number]
type Merged = [...HasNoLabels, ...HasLabels]
//   ^ [number, number, string, string]
//
//     'a' and 'b' were lost in 'Merged'
```

在 TypeScript 5.2 中，元组标签“非全即无”的限制已被解除。现在，在将具名元组展开到未具名元组中时，语言也能够保留标签。

衷心感谢 [Josh Goldberg](https://github.com/JoshuaKGoldberg) 与 [Mateusz Burzyński](https://github.com/Andarist)[共同合作解除了这一限制](https://github.com/microsoft/TypeScript/pull/53356)。

## 数组联合类型的方法调用更加简便

在先前版本的 TypeScript 中，在数组的联合类型上调用方法可能会非常痛苦。

```ts
declare let array: string[] | number[]

array.filter((x) => !!x)
//    ~~~~~~ error!
// This expression is not callable.
//   Each member of the union type '...' has signatures,
//   but none of those signatures are compatible
//   with each other.
```

在此示例中，TypeScript 会尝试查看 `filter` 的各个版本在 `string[]` 和 `number[]` 之间是否相互兼容。由于缺乏合理的策略，TypeScript 只能两手一摊表示“这行不通”。

在 TypeScript 5.2 中，在放弃之前，数组的联合类型会被作为一种特殊情况来处理。系统会根据每个成员的元素类型构建一个新的数组类型，然后在该新类型上调用该方法。

以上述示例为例，`string[] | number[]` 会被转换为 `(string | number)[]`（或 `Array<string | number>`），并在该类型上调用 `filter`。需要说明的一个微小差异是：`filter` 将产生 `Array<string | number>` 而不是 `string[] | number[]`；但对于新生成的值，发生意外的风险相对较低。

这意味着像 `filter`、`find`、`some`、`every` 和 `reduce` 等大量方法，现在都可以在数组联合类型上正常调用，而此前则无法做到。

欲了解更多细节，请[查阅对应的 Pull Request](https://github.com/microsoft/TypeScript/pull/53489)。

## 仅类型导入路径支持 TypeScript 实现文件扩展名

TypeScript 现在允许在仅类型导入（type-only import）路径中包含声明文件*和*实现文件的扩展名，无论是否启用了 `allowImportingTsExtensions`。

这意味着你现在可以编写使用 `.ts`、`.mts`、`.cts` 和 `.tsx` 文件扩展名的 `import type` 语句。

```ts
import type { JustAType } from './justTypes.ts'

export function f(param: JustAType) {
  // ...
}
```

这也意味着在 TypeScript 以及配合 JSDoc 的 JavaScript 中均可使用的 `import()` 类型，同样可以使用这些文件扩展名。

```js
/**
 * @param {import("./justTypes.ts").JustAType} param
 */
export function f(param) {
  // ...
}
```

更多信息[请参阅此处的变更](https://github.com/microsoft/TypeScript/pull/54746)。

## 对象成员的逗号自动补全

向对象添加新属性时，很容易忘记添加逗号。此前，如果你漏掉了逗号并请求自动补全，TypeScript 会给出令人困惑且无关的较差补全结果。

TypeScript 5.2 现在能够在缺少逗号时优雅地提供对象成员补全。而且为了避免直接抛出语法错误，它还*会自动*补全缺失的逗号。

![Properties in an object literal are completed despite missing a comma after a prior property. When the property name is completed, the missing comma is automatically inserted.](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2023/06/comma-completions-5-2-beta.gif)

更多信息[请参阅此处的实现](https://github.com/microsoft/TypeScript/pull/52899)。

## 内联变量重构

TypeScript 5.2 现在新增了一项重构功能，可将变量的内容内联到所有使用该变量的位置。

![A variable called 'path' initialized to a string, having both of its usages replaced](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2023/06/inline-variable-5-2-beta.gif).

使用“内联变量”（inline variable）重构会消除该变量，并将所有对该变量的使用替换为其初始化表达式。请注意，这可能会导致该初始化表达式的副作用在不同的时间运行，且运行次数与该变量被使用的次数相同。

更多详情[请参阅对应的 Pull Request](https://github.com/microsoft/TypeScript/pull/54281)。

<!-- Inlay Parameter Hints -->

## 优化进行中的类型兼容性检查

由于 TypeScript 是结构化类型系统，有时需要按成员逐一比较类型；然而，递归类型在此会引发一些问题。例如：

```ts
interface A {
  value: A
  other: string
}

interface B {
  value: B
  other: number
}
```

在检查类型 `A` 是否与类型 `B` 兼容时，TypeScript 最终会检查 `A` 和 `B` 中 `value` 的类型是否分别兼容。此时，类型系统需要停止进一步深入检查并转去检查其他成员。为此，类型系统必须跟踪何时任意两个类型已经处于关联比较中。

此前，TypeScript 已经维护了一个类型对栈，并通过遍历该栈来确定这些类型是否正在关联。当该栈较浅时这不是问题；但当栈较深时，呃，[问题就来了](https://accidentallyquadratic.tumblr.com/)。

在 TypeScript 5.3 中，一个简单的 `Set` 有助于跟踪此信息。这使得使用 [drizzle](https://github.com/drizzle-team/drizzle-orm) 库的一个上报测试用例的耗时减少了 33% 以上！

```
Benchmark 1: old
  Time (mean ± σ):      3.115 s ±  0.067 s    [User: 4.403 s, System: 0.124 s]
  Range (min … max):    3.018 s …  3.196 s    10 runs

Benchmark 2: new
  Time (mean ± σ):      2.072 s ±  0.050 s    [User: 3.355 s, System: 0.135 s]
  Range (min … max):    1.985 s …  2.150 s    10 runs

Summary
  'new' ran
    1.50 ± 0.05 times faster than 'old'
```

[关于此变更的更多内容请参阅此处](https://github.com/microsoft/TypeScript/pull/55224)。

## 破坏性变更与正确性修复

TypeScript 尽力避免引入不必要的破坏性变更；然而，我们偶尔必须进行一些修正和改进，以便更好地分析代码。

### `lib.d.ts` 变更

为 DOM 生成的类型可能会对你的代码库产生影响。欲了解更多信息，请[参阅 TypeScript 5.2 的 DOM 更新](https://github.com/microsoft/TypeScript/pull/54725)。

### `labeledElementDeclarations` 可能包含 `undefined` 元素

为了[支持具名与匿名元素的混用](https://github.com/microsoft/TypeScript/pull/53356)，TypeScript 的 API 发生了微调。在元素未具名的每个位置，`TupleType` 的 `labeledElementDeclarations` 属性现在可能保存 `undefined`。

```diff
  interface TupleType {
-     labeledElementDeclarations?: readonly (NamedTupleMember | ParameterDeclaration)[];
+     labeledElementDeclarations?: readonly (NamedTupleMember | ParameterDeclaration | undefined)[];
  }
```

### 在较新的 Node.js 设置下 `module` 与 `moduleResolution` 必须匹配

`--module` 和 `--moduleResolution` 选项各自支持 `node16` 和 `nodenext` 设置。这些实际上是应该在任何较新 Node.js 项目中使用的“现代 Node.js”设置。我们发现，当这两个选项在是否使用 Node.js 相关设置上不一致时，项目实际上处于配置错误的状态。

在 TypeScript 5.2 中，当在 `--module` 和 `--moduleResolution` 选项中的任意一个使用 `node16` 或 `nodenext` 时，TypeScript 现在要求另一个选项也必须使用类似的 Node.js 相关设置。在设置不一致的情况下，你可能会收到如下两种错误信息之一：

```
Option 'moduleResolution' must be set to 'NodeNext' (or left unspecified) when option 'module' is set to 'NodeNext'.
```

或

```
Option 'module' must be set to 'Node16' when option 'moduleResolution' is set to 'Node16'.
```

因此，像 `--module esnext --moduleResolution node16` 这样的配置将被拒绝——你可能更适合单独使用 `--module nodenext`，或者使用 `--module esnext --moduleResolution bundler`。

更多信息[请参阅此处的变更](https://github.com/microsoft/TypeScript/pull/54567)。

### 合并符号的导出一致性检查

当两个声明合并时，它们在是否都被导出这一点上必须保持一致。由于先前存在的一个缺陷，TypeScript 遗漏了环境上下文（ambient context）中的特定情况，例如声明文件或 `declare module` 块。例如，在如下代码中，`replaceInFile` 被声明了一次作为导出的函数，又被声明了一次作为未导出的命名空间，此前 TypeScript 并不会报错：

```ts
declare module 'replace-in-file' {
  export function replaceInFile(config: unknown): Promise<unknown[]>
  export {}

  namespace replaceInFile {
    export function sync(config: unknown): unknown[]
  }
}
```

在环境模块中，添加 `export { ... }` 或类似 `export default ...` 的结构会隐式改变所有声明是否自动导出的行为。TypeScript 现在能够更加一致地识别这种容易令人困惑的语义，并对 `replaceInFile` 的所有声明在修饰符上需要保持一致这一要求进行检查，报错如下：

```
Individual declarations in merged declaration 'replaceInFile' must be all exported or all local.
```

更多信息[请参阅此处的变更](https://github.com/microsoft/TypeScript/pull/54659)。
