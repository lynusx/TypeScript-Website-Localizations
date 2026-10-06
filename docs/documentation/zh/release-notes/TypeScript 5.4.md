---
title: TypeScript 5.4
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-5-4.html
oneline: TypeScript 5.4 发布说明
---

## 闭包中保留最后一次赋值后的类型收窄（Preserved Narrowing in Closures Following Last Assignment）

TypeScript 通常可以根据你执行的条件检查推断出变量的更具体类型。
这个过程被称为类型收窄（Narrowing）。

```ts
function uppercaseStrings(x: string | number) {
  if (typeof x === 'string') {
    // 此处 TypeScript 已知 'x' 是 'string'。
    return x.toUpperCase()
  }
}
```

一个常见的痛点是：这些收窄后的类型并不总能在函数闭包中得以保留。

```ts
function getUrls(url: string | URL, names: string[]) {
  if (typeof url === 'string') {
    url = new URL(url)
  }

  return names.map((name) => {
    url.searchParams.set('name', name)
    //  ~~~~~~~~~~~~
    // 错误！
    // 属性 'searchParams' 不存在于类型 'string | URL' 上。

    return url.toString()
  })
}
```

在这里，TypeScript 认为在回调函数中假设 `url` *确实*是一个 `URL` 对象并“不安全”，因为该变量在其他地方被修改过；
然而在该示例中，这个箭头函数*总是*在对 `url` 进行赋值之后才创建的，而且这也是对 `url` 的*最后一次*赋值。

TypeScript 5.4 充分利用了这一点，使类型收窄变得更加智能。
当形参和 `let` 变量在非[提升（hoisted）](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting)函数中使用时，类型检查器会寻找最后一次赋值的位置。
如果找到了该位置，TypeScript 就可以安全地从包含函数的外部作用域继承收窄后的类型。
这意味着上面的示例现在可以正常工作了。

注意，如果变量在任何嵌套函数内部被赋值，收窄分析就不会生效。
这是因为无法确切知道该函数是否会在后续被调用。

```ts
function printValueLater(value: string | undefined) {
  if (value === undefined) {
    value = 'missing!'
  }

  setTimeout(() => {
    // 修改 'value'（即使以不应影响其类型的方式），
    // 也会使闭包中的类型细化失效。
    value = value
  }, 500)

  setTimeout(() => {
    console.log(value.toUpperCase())
    //          ~~~~~
    // 错误！'value' 可能为 'undefined'。
  }, 1000)
}
```

这将使大量典型的 JavaScript 代码更容易表达。
你可以[在 GitHub 上阅读有关此更改的更多信息](https://github.com/microsoft/TypeScript/pull/56908)。

## `NoInfer` 工具类型

在调用泛型函数时，TypeScript 能够根据你传入的实参来推断类型参数。

```ts
function doSomething<T>(arg: T) {
  // ...
}

// 我们可以显式声明 'T' 应为 'string'。
doSomething<string>('hello!')

// 我们也可以直接让 'T' 的类型被推断出来。
doSomething('hello!')
```

然而，面临的一个挑战是：并不总是清楚应该推断出的“最佳”类型是什么。
这可能会导致 TypeScript 拒绝合法的调用、接受存疑的调用，或者在捕获错误时给出更糟糕的错误信息。

例如，设想一个接收颜色名称列表以及可选默认颜色的 `createStreetLight` 函数。

```ts
function createStreetLight<C extends string>(colors: C[], defaultColor?: C) {
  // ...
}

createStreetLight(['red', 'yellow', 'green'], 'red')
```

当我们传入一个不存在于原始 `colors` 数组中的 `defaultColor` 时会发生什么？
在这个函数中，`colors` 应该作为“事实来源（source of truth）”，并描述哪些值可以传递给 `defaultColor`。

```ts
// 糟糕！这并非所期望的，但被允许！
createStreetLight(['red', 'yellow', 'green'], 'blue')
```

在此次调用中，类型推断认为 `"blue"` 与 `"red"`、`"yellow"` 或 `"green"` 一样是一个合法的类型。
因此 TypeScript 没有拒绝该调用，而是将 `C` 的类型推断为 `"red" | "yellow" | "green" | "blue"`。
可以说这里的类型推断完全出乎了我们的意料！

目前人们应对这种情况的一种方法是添加一个由现有类型参数约束的独立类型参数。

```ts
function createStreetLight<C extends string, D extends C>(
  colors: C[],
  defaultColor?: D,
) {}

createStreetLight(['red', 'yellow', 'green'], 'blue')
//                                            ~~~~~~
// 错误！
// 类型 '"blue"' 的参数不能赋值给类型 '"red" | "yellow" | "green" | undefined' 的参数。
```

这种方法可行，但略显别扭，因为 `D` 可能在 `createStreetLight` 的签名中不会用于任何其他地方。
虽然*在这种情况下*不算太糟，但在类型签名中只使用一次类型参数通常是一种代码异味（code smell）。

这就是为什么 TypeScript 5.4 引入了全新的 `NoInfer<T>` 工具类型。
使用 `NoInfer<...>` 包裹类型可以向 TypeScript 发出信号，告知其不要深入匹配内部类型来寻找类型推断的候选者。

使用 `NoInfer`，我们可以将 `createStreetLight` 重写为如下形式：

```ts
function createStreetLight<C extends string>(
  colors: C[],
  defaultColor?: NoInfer<C>,
) {
  // ...
}

createStreetLight(['red', 'yellow', 'green'], 'blue')
//                                            ~~~~~~
// 错误！
// 类型 '"blue"' 的参数不能赋值给类型 '"red" | "yellow" | "green" | undefined' 的参数。
```

将 `defaultColor` 的类型排除在推断探索之外，意味着 `"blue"` 绝不会成为推断候选，因此类型检查器可以将其拒绝。

你可以在[实现该特性的 Pull Request](https://github.com/microsoft/TypeScript/pull/56794) 中查看具体的改动，同时感谢 [Mateusz Burzyński](https://github.com/Andarist) 提供的[初始实现](https://github.com/microsoft/TypeScript/pull/52968)！

## `Object.groupBy` 与 `Map.groupBy`

TypeScript 5.4 为 JavaScript 全新的 `Object.groupBy` 和 `Map.groupBy` 静态方法添加了类型声明。

`Object.groupBy` 接收一个可迭代对象，以及一个决定每个元素应当放入哪个“分组”的函数。
该函数需要为每个不同的分组生成一个“键（key）”，`Object.groupBy` 会使用该键来创建一个对象，其中每个键都映射到一个包含原始元素的数组。

因此以下 JavaScript 代码：

```js
const array = [0, 1, 2, 3, 4, 5]

const myObj = Object.groupBy(array, (num, index) => {
  return num % 2 === 0 ? 'even' : 'odd'
})
```

基本上等同于编写以下代码：

```js
const myObj = {
  even: [0, 2, 4],
  odd: [1, 3, 5],
}
```

`Map.groupBy` 与之类似，但生成的是 `Map` 而不是普通对象。
如果你需要 `Map` 的行为保证、正在处理期望接收 `Map` 的 API，或者需要使用任意类型的键进行分组（而不仅限于 JavaScript 中可用作属性名的键），那么它会更加适用。

```js
const myObj = Map.groupBy(array, (num, index) => {
  return num % 2 === 0 ? 'even' : 'odd'
})
```

正如前文所述，你也可以通过等效的方式创建 `myObj`：

```js
const myObj = new Map()

myObj.set('even', [0, 2, 4])
myObj.set('odd', [1, 3, 5])
```

请注意，在上述 `Object.groupBy` 的示例中，生成的对象的所有属性都是可选的。

```ts
interface EvenOdds {
    even?: number[];
    odd?: number[];
}

const myObj: EvenOdds = Object.groupBy(...);

myObj.even;
//    ~~~~
// 在 'strictNullChecks' 下访问此处会报错。
```

这是因为通常无法保证 `groupBy` 会生成*所有*的键。

另外还要注意，只有将 `target` 配置为 `esnext` 或调整 `lib` 设置，才能访问这些方法。
我们预计它们最终将在稳定的 `es2024` 目标版本下可用。

感谢 [Kevin Gibbons](https://github.com/bakkot) [为这些 `groupBy` 方法添加了类型声明](https://github.com/microsoft/TypeScript/pull/56805)。

## 在 `--moduleResolution bundler` 和 `--module preserve` 中支持 `require()` 调用

TypeScript 提供了一个名为 `bundler` 的 `moduleResolution` 选项，旨在模拟现代打包器确定导入路径所指代文件的方式。
该选项的限制之一是它必须与 `--module esnext` 配合使用，从而导致无法使用 `import ... = require(...)` 语法。

```ts
// 此前会报错
import myModule = require('module/path')
```

如果你只打算编写标准的 ECMAScript `import`，这看起来可能不是什么大问题；但当使用带有[条件导出（conditional exports）](https://nodejs.org/api/packages.html#conditional-exports)的包时，就会产生区别。

在 TypeScript 5.4 中，当将 `module` 设置为名为 `preserve` 的新选项时，现在可以使用 `require()` 了。

结合使用 `--module preserve` 与 `--moduleResolution bundler`，可以更准确地模拟打包器以及诸如 Bun 等运行时所允许的行为，以及它们执行模块查找的方式。
事实上，当使用 `--module preserve` 时，`--moduleResolution` 将被隐式设置为 `bundler`（同时隐式启用 `--esModuleInterop` 和 `--resolveJsonModule`）。

```json5
{
  compilerOptions: {
    module: 'preserve',
    // ^ also implies:
    // "moduleResolution": "bundler",
    // "esModuleInterop": true,
    // "resolveJsonModule": true,

    // ...
  },
}
```

在 `--module preserve` 下，ECMAScript `import` 始终会原样输出，而 `import ... = require(...)` 将输出为 `require()` 调用（尽管在实践中你可能根本不会使用 TypeScript 进行输出，因为你很可能会使用打包器来处理代码）。
无论包含该代码的文件的扩展名是什么，这一点都成立。
因此以下代码的输出：

```ts
import * as foo from 'some-package/foo'
import bar = require('some-package/bar')
```

看起来大致如下：

```js
import * as foo from 'some-package/foo'
var bar = require('some-package/bar')
```

这也意味着你所选择的语法将决定如何匹配[条件导出（conditional exports）](https://nodejs.org/api/packages.html#conditional-exports)。
因此在上面的示例中，如果 `some-package` 的 `package.json` 如下所示：

```json5
{
  name: 'some-package',
  version: '0.0.1',
  exports: {
    './foo': {
      import: './esm/foo-from-import.mjs',
      require: './cjs/foo-from-require.cjs',
    },
    './bar': {
      import: './esm/bar-from-import.mjs',
      require: './cjs/bar-from-require.cjs',
    },
  },
}
```

TypeScript 会将这些路径分别解析为 `[...]/some-package/esm/foo-from-import.mjs` 和 `[...]/some-package/cjs/bar-from-require.cjs`。

欲了解更多信息，你可以[在此阅读有关这些新设置的内容](https://github.com/microsoft/TypeScript/pull/56785)。

## 导入属性与断言的类型检查

导入属性和断言现在会根据全局的 `ImportAttributes` 类型进行检查。
这意味着运行时现在可以更准确地描述导入属性。

```ts
// 在某个全局文件中。
interface ImportAttributes {
  type: 'json'
}

// 在其他某个模块中
import * as ns from 'foo' with { type: 'not-json' }
//                                     ~~~~~~~~~~
// 错误！
//
// 类型 '{ type: "not-json"; }' 不能赋值给类型 'ImportAttributes'。
//  属性 'type' 的类型不兼容。
//    类型 '"not-json"' 不能赋值给类型 '"json"'。
```

感谢 [Oleksandr Tarasiuk](https://github.com/a-tarasyuk) 提供了[此项更改](https://github.com/microsoft/TypeScript/pull/56034)。

## 添加缺失参数的快速修复

TypeScript 现在提供了一个快速修复（Quick Fix），用于向传入过多实参调用的函数中添加新参数。

![A quick fix being offered when someFunction calls someHelperFunction with 2 more arguments than are expected.](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2024/01/add-missing-params-5-4-beta-before.png)

![The missing arguments have been added to someHelperFunction after the quick fix was applied.](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2024/01/add-missing-params-5-4-beta-after.png)

当需要将一个新参数逐层传递给多个现有函数时，这非常实用，而在以往这往往相当繁琐。

[该快速修复](https://github.com/microsoft/TypeScript/pull/56411)由 [Oleksandr Tarasiuk](https://github.com/a-tarasyuk) 贡献。

## TypeScript 5.0 弃用项即将到来的变更

TypeScript 5.0 弃用了以下选项和行为：

- `charset`
- `target: ES3`
- `importsNotUsedAsValues`
- `noImplicitUseStrict`
- `noStrictGenericChecks`
- `keyofStringsOnly`
- `suppressExcessPropertyErrors`
- `suppressImplicitAnyIndexErrors`
- `out`
- `preserveValueImports`
- 项目引用中的 `prepend`
- 隐式受操作系统影响的 `newLine`

为了继续使用它们，使用 TypeScript 5.0 及更新版本的开发者此前必须指定一个名为 `ignoreDeprecations` 的新选项，其值为 `"5.0"`。

然而，TypeScript 5.4 将是这些选项能够继续正常发挥作用的最后一个版本。
到 TypeScript 5.5（预计为 2024 年 6 月）时，它们将变成硬性错误，使用它们的代码将必须完成迁移。

欲了解更多信息，你可以[在 GitHub 上阅读该计划](https://github.com/microsoft/TypeScript/issues/51909)，其中包含了关于如何妥善改造代码库的建议。

## 值得注意的行为变更

本节重点介绍在任何版本升级过程中都应予以确认和理解的一系列值得关注的变更。
它有时会突出弃用项、移除项以及新的限制。
此外也可能包含功能改进方面的错误修复，但这些修复也可能通过引入新错误而对现有的构建产生影响。

### `lib.d.ts` 变更

为 DOM 生成的类型可能会对你代码库的类型检查产生影响。
欲了解更多信息，请[参阅 TypeScript 5.4 的 DOM 更新](https://github.com/microsoft/TypeScript/pull/57027)。

### 条件类型约束更加精确

以下代码在函数 `foo` 中不再允许声明第二个变量。

```ts
type IsArray<T> = T extends any[] ? true : false

function foo<U extends object>(x: IsArray<U>) {
  let first: true = x // 错误
  let second: false = x // 错误，但此前不会报错
}
```

此前，当 TypeScript 检查 `second` 的初始化表达式时，需要确定 `IsArray<U>` 是否可赋值给单元类型 `false`。
虽然 `IsArray<U>` 并没有显而易见的兼容性，但 TypeScript 还会查看该类型的*约束（constraint）*。
在形如 `T extends Foo ? TrueBranch : FalseBranch`（其中 `T` 为泛型）的条件类型中，类型系统会检查 `T` 的约束，将其代入 `T` 本身，然后决定进入 true 分支还是 false 分支。

但这种行为并不准确，因为它过于急切（eager）。
即使 `T` 的约束不可赋值给 `Foo`，也不代表它在实例化时不会接收一个可赋值给 `Foo` 的具体类型。
因此更正确的行为是：在无法证明 `T` *从不*或*始终*继承自 `Foo` 的情况下，为条件类型的约束生成一个联合类型。

TypeScript 5.4 采用了这种更加精确的行为。
在实践中这意味着你可能会开始发现某些条件类型实例不再与其分支兼容。

[你可以在此阅读具体的相关变更](https://github.com/microsoft/TypeScript/pull/56004)。

### 更激进地化简类型变量与原始类型之间的交叉类型

TypeScript 现在会更激进地化简类型变量与原始类型之间的交叉类型，具体取决于该类型变量的约束与这些原始类型的重叠情况。

```ts
declare function intersect<T, U>(x: T, y: U): T & U

function foo<T extends 'abc' | 'def'>(x: T, str: string, num: number) {
  // 原为 'T & string'，现在仅为 'T'
  let a = intersect(x, str)

  // 原为 'T & number'，现在仅为 'never'
  let b = intersect(x, num)

  // 原为 '(T & "abc") | (T & "def")'，现在仅为 'T'
  let c = Math.random() < 0.5 ? intersect(x, 'abc') : intersect(x, 'def')
}
```

欲了解更多信息，请[参阅此处的更改](https://github.com/microsoft/TypeScript/pull/56515)。

### 改进对带插值的模板字符串的检查

TypeScript 现在能更精确地检查字符串是否可以赋值给模板字符串类型的占位符插槽。

```ts
function a<T extends { id: string }>() {
  let x: `-${keyof T & string}`

  // 以前会报错，现在不会了。
  x = '-id'
}
```

这种行为更符合预期，但在使用条件类型等语法结构的代码中可能会引发破坏，因为在这些场景下规则的变化很容易显现。

更多详情请[参见该更改](https://github.com/microsoft/TypeScript/pull/56598)。

### 仅类型导入与本地值冲突时报错

此前在开启 `isolatedModules` 的情况下，如果对 `Something` 的导入仅引用了一个类型，TypeScript 会允许以下代码：

```ts
import { Something } from './some/path'

let Something = 123
```

然而，对于单文件编译器而言，假定丢弃该 `import` 是否“安全”并不可靠，即便该代码在运行时必然失败。
在 TypeScript 5.4 中，此代码将触发类似如下的错误：

```
Import 'Something' conflicts with local value, so must be declared with a type-only import when 'isolatedModules' is enabled.
```

修复方法是进行本地重命名，或者正如错误提示所述，为导入添加 `type` 修饰符：

```ts
import type { Something } from './some/path'

// or

import { type Something } from './some/path'
```

[查看关于该更改本身的更多信息](https://github.com/microsoft/TypeScript/pull/56354)。

### 新增枚举可赋值性限制

当两个枚举具有相同的声明名称和枚举成员名称时，此前它们总是被视为兼容的；
然而，即使成员的值已知，TypeScript 也会静默允许它们拥有不同的值。

TypeScript 5.4 收紧了这一限制，要求当枚举成员的值已知时，它们的值必须完全相同。

```ts
namespace First {
  export enum SomeEnum {
    A = 0,
    B = 1,
  }
}

namespace Second {
  export enum SomeEnum {
    A = 0,
    B = 2,
  }
}

function foo(x: First.SomeEnum, y: Second.SomeEnum) {
  // 两者以前兼容 —— 现在不再如此，
  // TypeScript 会报类似如下的错误：
  //
  //  'SomeEnum.B' 的各个声明在其值上存在差异，期望为 '1' 但获得了 '2'。
  x = y
  y = x
}
```

此外，当其中一个枚举成员不具有静态已知的值时，也引入了新的限制。
在这些情况下，另一个枚举必须至少是隐式数值型的（例如它没有静态解析的初始值设定项），或者是显式数值型的（意味着 TypeScript 能够将其值解析为数值）。
在实际应用中，这意味着字符串枚举成员仅与具有相同值的其他字符串枚举兼容。

```ts
namespace First {
  export declare enum SomeEnum {
    A,
    B,
  }
}

namespace Second {
  export declare enum SomeEnum {
    A,
    B = 'some known string',
  }
}

function foo(x: First.SomeEnum, y: Second.SomeEnum) {
  // 两者以前兼容 —— 现在不再如此，
  // TypeScript 会报类似如下的错误：
  //
  //  'SomeEnum.B' 的一个值是字符串 '"some known string"'，而另一个值被假定为未知的数字值。
  x = y
  y = x
}
```

欲了解更多信息，请[参阅引入该更改的 Pull Request](https://github.com/microsoft/TypeScript/pull/55924)。

### 枚举成员名称限制

TypeScript 不再允许枚举成员使用 `Infinity`、`-Infinity` 或 `NaN` 作为名称。

```ts
// 对以下所有情况均报错：
//
//  枚举成员不能具有数字名称。
enum E {
  Infinity = 0,
  '-Infinity' = 1,
  NaN = 2,
}
```

[在此查看更多详情](https://github.com/microsoft/TypeScript/pull/56161)。

### 更好地保留包含 `any` rest 元素的元组上的映射类型

此前，将带有 `any` 的映射类型应用于元组时会产生 `any` 元素类型。
这是不符合预期的，现已修复。

```ts
Promise.all(['', ...([] as any)]).then((result) => {
  const head = result[0] // 5.3: any, 5.4: string
  const tail = result.slice(1) // 5.3 any, 5.4: any[]
})
```

欲了解更多信息，请参阅[此修复](https://github.com/microsoft/TypeScript/pull/57031)，以及[围绕行为变更的后续讨论](https://github.com/microsoft/TypeScript/issues/57389)与[进一步调整](https://github.com/microsoft/TypeScript/issues/57389)。

### 代码生成变更

虽然这本身不属于破坏性变更，但开发者可能隐式依赖了 TypeScript 的 JavaScript 或声明文件生成输出。
以下是值得注意的变更：

- [在类型参数被遮蔽（shadowed）时更频繁地保留其名称](https://github.com/microsoft/TypeScript/pull/55820)
- [将 async 函数的复杂参数列表移至降级生成器函数体中](https://github.com/microsoft/TypeScript/pull/56296)
- [不要在函数声明中移除绑定别名](https://github.com/microsoft/TypeScript/pull/57020)
- [在 ImportTypeNode 中时，ImportAttributes 应当经历相同的代码生成阶段](https://github.com/microsoft/TypeScript/pull/56395)
