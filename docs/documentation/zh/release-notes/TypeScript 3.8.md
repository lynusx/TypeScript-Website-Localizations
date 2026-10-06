---
title: TypeScript 3.8
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-3-8.html
oneline: TypeScript 3.8 发布说明
---

## 仅类型导入与导出

这项功能大多数用户可能并不需要关心；但如果你在使用 [`isolatedModules`](/tsconfig#isolatedModules)、TypeScript 的 `transpileModule` API 或 Babel 时遇到过问题，这项功能就与你息息相关了。

TypeScript 3.8 为仅类型导入与导出引入了新的语法。

```ts
import type { SomeThing } from './some-module.js'

export type { SomeThing }
```

`import type` 仅导入用于类型注解和声明的声明项。它**总是**会被完全擦除，因此在运行时不会留下任何痕迹。类似地，`export type` 仅提供可用于类型上下文的导出，同样会被从 TypeScript 的编译输出中完全擦除。

需要特别注意的是，类在运行时具有值，而在设计时具有类型，其使用方式取决于上下文。当使用 `import type` 导入一个类时，你无法对其进行继承等操作。

```ts
import type { Component } from 'react'

interface ButtonProps {
  // ...
}

class Button extends Component<ButtonProps> {
  //               ~~~~~~~~~
  // error! 'Component' only refers to a type, but is being used as a value here.
  // ...
}
```

如果你之前用过 Flow，会发现该语法十分相似。不同之处在于，我们添加了一些限制以避免可能产生歧义的代码。

```ts
// Is only 'Foo' a type? Or every declaration in the import?
// We just give an error because it's not clear.

import type Foo, { Bar, Baz } from "some-module";
//     ~~~~~~~~~~~~~~~~~~~~~~
// error! A type-only import can specify a default import or named bindings, but not both.
```

与 `import type` 配套，TypeScript 3.8 还添加了一个新的编译器标志，用于控制未在运行时使用的导入的处理方式：[`importsNotUsedAsValues`](/tsconfig#importsNotUsedAsValues)。
该标志接受 3 个不同的值：

- `remove`：即当前丢弃这些导入的行为。这仍然是默认行为，属于非破坏性变更。
- `preserve`：**保留**所有值未被使用的导入。这会保留这些导入及其副作用。
- `error`：保留所有导入（与 `preserve` 选项相同），但当某个值导入仅用作类型时会报错。如果你希望确保不会意外导入任何值，同时明确显式保留副作用导入，该选项会很有用。

有关该功能的更多信息，你可以[查看 Pull Request](https://github.com/microsoft/TypeScript/pull/35200)，以及有关放宽 `import type` 声明导入项使用场景的[相关变更](https://github.com/microsoft/TypeScript/pull/36092/)。

## ECMAScript 私有字段

TypeScript 3.8 带来了对 ECMAScript 私有字段的支持，该特性属于 [Stage 3 类字段提案（stage-3 class fields proposal）](https://github.com/tc39/proposal-class-fields/) 的一部分。

```ts
class Person {
  #name: string

  constructor(name: string) {
    this.#name = name
  }

  greet() {
    console.log(`Hello, my name is ${this.#name}!`)
  }
}

let jeremy = new Person('Jeremy Bearimy')

jeremy.#name
//     ~~~~~
// Property '#name' is not accessible outside class 'Person'
// because it has a private identifier.
```

与常规属性（即使是使用 `private` 修饰符声明的属性）不同，私有字段有以下几条规则需要注意：

- 私有字段以 `#` 字符开头。有时我们称其为**私有名称（private names）**。
- 每个私有字段名称的作用域都严格唯一限定在声明它的类中。
- TypeScript 的访问修饰符（如 `public` 或 `private`）不能用于私有字段。
- 私有字段在包含它的类之外无法被访问甚至无法被探测到——对 JS 用户也是如此！有时我们称其为**硬私有（hard privacy）**。

除了“硬私有”之外，私有字段的另一个优势就是刚才提到的唯一性。例如，常规属性声明很容易在子类中被意外覆盖。

```ts
class C {
  foo = 10

  cHelper() {
    return this.foo
  }
}

class D extends C {
  foo = 20

  dHelper() {
    return this.foo
  }
}

let instance = new D()
// 'this.foo' refers to the same property on each instance.
console.log(instance.cHelper()) // prints '20'
console.log(instance.dHelper()) // prints '20'
```

使用私有字段时，你完全无需担心这一点，因为每个字段名称对于包含它的类来说都是唯一的。

```ts
class C {
  #foo = 10

  cHelper() {
    return this.#foo
  }
}

class D extends C {
  #foo = 20

  dHelper() {
    return this.#foo
  }
}

let instance = new D()
// 'this.#foo' refers to a different field within each class.
console.log(instance.cHelper()) // prints '10'
console.log(instance.dHelper()) // prints '20'
```

另一处值得注意的是，在任何其他类型的对象上访问私有字段都将抛出 `TypeError`！

```ts
class Square {
  #sideLength: number

  constructor(sideLength: number) {
    this.#sideLength = sideLength
  }

  equals(other: any) {
    return this.#sideLength === other.#sideLength
  }
}

const a = new Square(100)
const b = { sideLength: 100 }

// Boom!
// TypeError: attempted to get private field on non-instance
// This fails because 'b' is not an instance of 'Square'.
console.log(a.equals(b))
```

最后，对于纯 `.js` 文件用户，私有字段在被赋值之前**必须**先进行声明。

```js
class C {
  // No declaration for '#foo'
  // :(

  constructor(foo: number) {
    // SyntaxError!
    // '#foo' needs to be declared before writing to it.
    this.#foo = foo;
  }
}
```

JavaScript 一向允许用户访问未声明的属性，而 TypeScript 则始终要求对类属性进行声明。对于私有字段，无论是在 `.js` 还是 `.ts` 文件中编写代码，都必须先进行声明。

```js
class C {
  /** @type {number} */
  #foo;

  constructor(foo: number) {
    // This works.
    this.#foo = foo;
  }
}
```

有关该实现的更多信息，你可以[查看原始 Pull Request](https://github.com/Microsoft/TypeScript/pull/30829)。

### 我应该使用哪一种？

作为 TypeScript 用户，我们收到了很多关于应该使用哪种私有属性的疑问，最常见的问题是：“我应该使用 `private` 关键字，还是使用 ECMAScript 的 hash/井号（`#`）私有字段？”
这取决于具体需求！

对于属性而言，TypeScript 的 `private` 修饰符会被完全擦除——这意味着在运行时，它的表现与普通属性完全相同，没有任何方式能看出它是用 `private` 修饰符声明的。使用 `private` 关键字时，私有性仅在编译时/设计时强制执行，对于 JavaScript 调用者而言完全是基于意图约定的。

```ts
class C {
  private foo = 10
}

// This is an error at compile time,
// but when TypeScript outputs .js files,
// it'll run fine and print '10'.
console.log(new C().foo) // prints '10'
//                  ~~~
// error! Property 'foo' is private and only accessible within class 'C'.

// TypeScript allows this at compile-time
// as a "work-around" to avoid the error.
console.log(new C()['foo']) // prints '10'
```

这种“软私有（soft privacy）”的好处在于，它可以帮助使用者临时绕过无法访问某些 API 的限制，并且可以在任何运行时环境中工作。

另一方面，ECMAScript 的 `#` 私有字段在类外部是完全无法访问的。

```ts
class C {
  #foo = 10
}

console.log(new C().#foo) // SyntaxError
//                  ~~~~
// TypeScript reports an error *and*
// this won't work at runtime!

console.log(new C()['#foo']) // prints undefined
//          ~~~~~~~~~~~~~~~
// TypeScript reports an error under 'noImplicitAny',
// and this prints 'undefined'.
```

这种硬私有对于严格确保没有任何人能够使用你的内部实现非常有用。如果你是库的作者，移除或重命名私有字段永远不会导致破坏性变更。

正如我们提到的，另一个好处是使用 ECMAScript 的 `#` 私有字段使子类化更加容易，因为它们是**真正**私有的。使用 ECMAScript `#` 私有字段时，子类永远不需要担心字段命名冲突。而在使用 TypeScript 的 `private` 属性声明时，用户仍需格外小心，避免覆盖父类中声明的同名属性。

还需要考虑的另一点是代码的目标运行环境。除非编译目标为 ECMAScript 2015 (ES6) 或更高版本，否则 TypeScript 目前无法支持此特性。这是因为我们的降级编译实现使用 `WeakMap` 来强制实现私有性，而 `WeakMap` 无法在不引起内存泄漏的前提下进行 polyfill。相比之下，TypeScript 的 `private` 属性声明支持所有编译目标——甚至是 ECMAScript 3！

最后的考量是运行速度：`private` 属性与任何普通属性没有任何区别，因此无论目标运行时为何，访问它们的速度都与常规属性访问一样快。相反，由于 `#` 私有字段降级编译使用了 `WeakMap`，使用它们可能会更慢。虽然某些运行时环境可能会针对 `#` 私有字段的实际实现进行优化，甚至拥有极速的 `WeakMap` 实现，但并非所有运行时环境都是如此。

## `export * as ns` 语法

通常的做法是拥有一个统一的入口点，将另一个模块的所有成员作为一个命名空间成员导出。

```ts
import * as utilities from './utilities.js'
export { utilities }
```

这种模式非常普遍，以至于 ECMAScript 2020 最近新增了一种语法来支持该模式！

```ts
export * as utilities from './utilities.js'
```

这是对 JavaScript 的一项很好的开发体验改进，TypeScript 3.8 实现了这一语法。当你的模块目标版本低于 `es2020` 时，TypeScript 会输出类似于第一个代码片段的内容。

## 顶层 `await`（Top-Level await）

TypeScript 3.8 支持了即将到来的便捷 ECMAScript 新特性——“顶层 `await`”。

JavaScript 用户过去通常需要引入一个 `async` 函数才能使用 `await`，并在定义该函数后立即调用它。

```js
async function main() {
  const response = await fetch('...')
  const greeting = await response.text()
  console.log(greeting)
}

main().catch((e) => console.error(e))
```

这是因为以往在 JavaScript 中（以及大多数具有类似特性的其他语言中），`await` 仅允许在 `async` 函数体内使用。然而，通过顶层 `await`，我们可以在模块的顶层直接使用 `await`。

```ts
const response = await fetch('...')
const greeting = await response.text()
console.log(greeting)

// Make sure we're a module
export {}
```

请注意其中的细节：顶层 `await` 仅在**模块**的顶层有效，并且只有当 TypeScript 检测到 `import` 或 `export` 时，文件才会被视为模块。在某些简单场景下，你可能需要编写 `export {}` 这样的样板代码以确保其被识别为模块。

目前，顶层 `await` 可能还无法在你预期的所有环境中运行。目前只有当编译器选项 [`target`](/tsconfig#target) 为 `es2017` 或更高版本，且 `module` 为 `esnext` 或 `system` 时，才能使用顶层 `await`。在部分环境和打包工具中的支持可能有限，或者需要开启实验性支持。

有关我们实现的更多信息，你可以[查看原始 Pull Request](https://github.com/microsoft/TypeScript/pull/35813)。

## 用于 `target` 和 `module` 的 `es2020`

TypeScript 3.8 支持将 `es2020` 作为 `module` 和 [`target`](/tsconfig#target) 的选项值。这将保留较新的 ECMAScript 2020 特性，如可选链、空值合并、`export * as ns` 以及动态 `import(...)` 语法。这也意味着 `bigint` 字面量现在在 `esnext` 之下拥有了一个稳定的 [`target`](/tsconfig#target) 目标。

## JSDoc 属性修饰符

TypeScript 3.8 通过开启 [`allowJs`](/tsconfig#allowJs) 标志来支持 JavaScript 文件，并且通过 [`checkJs`](/tsconfig#checkJs) 选项或在 `.js` 文件顶部添加 `// @ts-check` 注释来支持对这些 JavaScript 文件的**类型检查**。

由于 JavaScript 文件没有专门用于类型检查的语法，TypeScript 利用了 JSDoc。TypeScript 3.8 能够识别几个新的用于属性的 JSDoc 标签。

首先是访问修饰符：`@public`、`@private` 和 `@protected`。这些标签的作用与 TypeScript 中的 `public`、`private` 和 `protected` 完全一致。

```js
// @ts-check

class Foo {
  constructor() {
    /** @private */
    this.stuff = 100
  }

  printStuff() {
    console.log(this.stuff)
  }
}

new Foo().stuff
//        ~~~~~
// error! Property 'stuff' is private and only accessible within class 'Foo'.
```

- `@public` 始终是默认隐含的，可以省略，表示属性可以从任何地方访问。
- `@private` 表示属性只能在包含它的类内部使用。
- `@protected` 表示属性只能在包含它的类及其所有派生子类中使用，但不能在包含类的不同实例上使用。

接下来，我们还添加了 `@readonly` 修饰符，以确保属性仅在初始化期间被写入。

```js
// @ts-check

class Foo {
  constructor() {
    /** @readonly */
    this.stuff = 100
  }

  writeToStuff() {
    this.stuff = 200
    //   ~~~~~
    // Cannot assign to 'stuff' because it is a read-only property.
  }
}

new Foo().stuff++
//        ~~~~~
// Cannot assign to 'stuff' because it is a read-only property.
```

## Linux 上更好的目录监视与 `watchOptions`

TypeScript 3.8 引入了一种监视目录的新策略，这对于高效捕获 `node_modules` 的变更至关重要。

作为背景说明：在 Linux 等操作系统上，TypeScript 会在 `node_modules` 及其众多子目录上安装目录监视器（相对于文件监视器而言），以检测依赖项的变更。这是因为可用文件监视器的数量通常远少于 `node_modules` 中的文件总数，而需要跟踪的目录数量则少得多。

较早版本的 TypeScript 会**立即**在文件夹上安装目录监视器，这在启动时没有问题；但在执行 `npm install` 期间，`node_modules` 内会发生大量变动，从而使 TypeScript 不堪重负，经常导致编辑器会话变得极度卡顿。为了防止这种情况，TypeScript 3.8 会在安装目录监视器之前稍作等待，让这些高频变动的目录有时间稳定下来。

由于不同项目可能适用于不同的策略，并且这种新方法可能并不完全契合你的工作流，TypeScript 3.8 在 `tsconfig.json` 和 `jsconfig.json` 中引入了一个新的 `watchOptions` 字段，允许用户告诉编译器/语言服务应使用哪些监视策略来跟踪文件和目录。

```jsonc tsconfig
{
  // Some typical compiler options
  "compilerOptions": {
    "target": "es2020",
    "moduleResolution": "node",
    // ...
  },

  // NEW: Options for file/directory watching
  "watchOptions": {
    // Use native file system events for files and directories
    "watchFile": "useFsEvents",
    "watchDirectory": "useFsEvents",

    // Poll files for updates more frequently
    // when they're updated a lot.
    "fallbackPolling": "dynamicPriority",
  },
}
```

`watchOptions` 包含 4 个可配置的新选项：

- [`watchFile`](/tsconfig#watchFile)：监视单个文件的策略。可以设置为：

  - `fixedPollingInterval`：以固定时间间隔每秒检查每个文件的变更数次。
  - `priorityPollingInterval`：以固定时间间隔每秒检查每个文件的变更数次，但使用启发式规则降低检查某些类型文件的频率。
  - `dynamicPriorityPolling`：使用动态队列，对较少修改的文件降低检查频率。
  - `useFsEvents`（默认值）：尝试使用操作系统/文件系统的原生事件来监听文件变更。
  - `useFsEventsOnParentDirectory`：尝试使用操作系统/文件系统的原生事件来监听文件所在目录的变更。这可以使用更少的文件监视器，但准确性可能会有所降低。

- [`watchDirectory`](/tsconfig#watchDirectory)：在缺乏递归文件监视功能的系统下监视整个目录树的策略。可以设置为：

  - `fixedPollingInterval`：以固定时间间隔每秒检查每个目录的变更数次。
  - `dynamicPriorityPolling`：使用动态队列，对较少修改的目录降低检查频率。
  - `useFsEvents`（默认值）：尝试使用操作系统/文件系统的原生事件来监听目录变更。

- [`fallbackPolling`](/tsconfig#fallbackPolling)：使用文件系统事件时，该选项指定当系统用尽原生文件监视器和/或不支持原生文件监视器时所使用的轮询策略。可以设置为：
  - `fixedPollingInterval`：_（参见上文。）_
  - `priorityPollingInterval`：_（参见上文。）_
  - `dynamicPriorityPolling`：_（参见上文。）_
  - `synchronousWatchDirectory`：禁用目录的延迟监视。延迟监视在可能同时发生大量文件变更时（例如运行 `npm install` 导致的 `node_modules` 变更）非常有用，但在某些特殊配置下，你可能希望使用此标志将其禁用。

有关这些变更的更多信息，请[前往 GitHub 查看 Pull Request](https://github.com/microsoft/TypeScript/pull/35615) 以了解更多内容。

## “快速且松散”的增量检查策略（Fast and Loose Incremental Checking）

TypeScript 3.8 引入了一个名为 [`assumeChangesOnlyAffectDirectDependencies`](/tsconfig#assumeChangesOnlyAffectDirectDependencies) 的新编译器选项。当启用此选项时，TypeScript 将避免重新检查/重新构建所有真正可能受影响的文件，而只重新检查/重新构建发生变更的文件以及直接导入它们的文件。

例如，假设文件 `fileD.ts` 导入了 `fileC.ts`，后者导入了 `fileB.ts`，而 `fileB.ts` 又导入了 `fileA.ts`，如下所示：

```
fileA.ts <- fileB.ts <- fileC.ts <- fileD.ts
```

在 `--watch` 模式下，`fileA.ts` 的变更通常意味着 TypeScript 至少需要重新检查 `fileB.ts`、`fileC.ts` 和 `fileD.ts`。而在 [`assumeChangesOnlyAffectDirectDependencies`](/tsconfig#assumeChangesOnlyAffectDirectDependencies) 下，`fileA.ts` 的变更意味着只有 `fileA.ts` 和 `fileB.ts` 需要重新检查。

在类似 Visual Studio Code 这样的代码库中，这使得某些文件变更后的重新构建时间从约 14 秒减少到约 1 秒。虽然我们并不一定向所有代码库推荐此选项，但如果你的代码库极其庞大，并且愿意将完整项目的错误检查推迟到稍后进行（例如通过 `tsconfig.fullbuild.json` 或在 CI 中进行专门构建），你可能会对它感兴趣。

更多详情，请[查看原始 Pull Request](https://github.com/microsoft/TypeScript/pull/35711)。
