---
title: TypeScript 4.7
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-4-7.html
oneline: TypeScript 4.7 发布说明
---

## Node.js 中的 ECMAScript 模块支持

过去几年中，Node.js 一直致力于支持 ECMAScript 模块（ESM）。这是一个非常艰难的特性，因为 Node.js 生态建立在另一个名为 CommonJS（CJS）的模块系统之上。在两者之间进行互操作带来了巨大的挑战，需要兼顾许多新特性；不过，Node.js 对 ESM 的支持在 Node.js 12 及更高版本中已基本实现。大约在 TypeScript 4.5 时，我们推出了仅在 nightly 版本中提供的 Node.js ESM 支持，以收集用户反馈并让库作者为更广泛的支持做好准备。

TypeScript 4.7 通过两个新的 `module` 设置提供了该功能：`node16` 和 `nodenext`。

```jsonc
{
  "compilerOptions": {
    "module": "node16",
  },
}
```

这些新模式带来了一些高层级特性，我们将在下文中进行探讨。

### `package.json` 中的 `type` 字段与新扩展名

Node.js 支持 [`package.json` 中的新设置](https://nodejs.org/api/packages.html#packages_package_json_and_file_extensions) `type`。`"type"` 可以设置为 `"module"` 或 `"commonjs"`。

```jsonc
{
  "name": "my-package",
  "type": "module",

  "//": "...",
  "dependencies": {},
}
```

该设置控制 `.js` 和 `.d.ts` 文件被解释为 ES 模块还是 CommonJS 模块，未设置时默认为 CommonJS。当一个文件被视为 ES 模块时，与 CommonJS 相比会有若干不同规则生效：

- 可以使用 `import`/`export` 语句。
- 可以使用顶层 `await`。
- 相对导入路径必须带有完整的文件扩展名（必须写成 `import "./foo.js"` 而不是 `import "./foo"`）。
- 导入可能会以不同方式解析 `node_modules` 中的依赖项。
- 某些类全局变量（如 `require` 和 `module`）不能直接使用。
- CommonJS 模块会根据特定的特殊规则进行导入。

稍后我们会回过头来讨论其中的一部分。

为了在这一体系中结合 TypeScript 的工作方式，`.ts` 和 `.tsx` 文件现在也遵循相同的机制。当 TypeScript 发现 `.ts`、`.tsx`、`.js` 或 `.jsx` 文件时，它会向上查找 `package.json` 以确定该文件是否为 ES 模块，并据此决定：

- 如何查找该文件所导入的其他模块
- 以及在生成输出时如何转换该文件

当 `.ts` 文件被编译为 ES 模块时，ECMAScript `import`/`export` 语句会在 `.js` 输出中保持原样；当编译为 CommonJS 模块时，它会生成与目前在 `--module commonjs` 下相同的输出。

这也意味着作为 ES 模块的 `.ts` 文件与作为 CJS 模块的 `.ts` 文件之间的路径解析方式有所不同。例如，假设你现在有以下代码：

```ts
// ./foo.ts
export function helper() {
  // ...
}

// ./bar.ts
import { helper } from './foo' // only works in CJS

helper()
```

这段代码在 CommonJS 模块中可以正常工作，但在 ES 模块中会失败，因为相对导入路径需要使用扩展名。因此，必须将其重写为使用 `foo.ts` 的*输出文件*扩展名——即 `bar.ts` 必须从 `./foo.js` 导入。

```ts
// ./bar.ts
import { helper } from './foo.js' // works in ESM & CJS

helper()
```

起初这可能略显繁琐，但 TypeScript 工具链（如自动导入和路径补全）通常会自动为你完成这项工作。

另外值得一提的是，这同样适用于 `.d.ts` 文件。当 TypeScript 在某个包中发现 `.d.ts` 文件时，会根据所在的包来对其进行解释。

### 新的文件扩展名

`package.json` 中的 `type` 字段很方便，因为它允许我们继续使用 `.ts` 和 `.js` 文件扩展名；但是，有时你编写的文件需要与 `type` 指定的模块格式不同。或者你可能更喜欢始终显式指定。

Node.js 支持两种扩展名来解决这个问题：`.mjs` 和 `.cjs`。`.mjs` 文件始终是 ES 模块，而 `.cjs` 文件始终是 CommonJS 模块，且无法被覆盖。

相应地，TypeScript 支持了两个新的源文件扩展名：`.mts` 和 `.cts`。当 TypeScript 将它们编译输出为 JavaScript 文件时，将分别输出为 `.mjs` 和 `.cjs`。

此外，TypeScript 还支持了两个新的声明文件扩展名：`.d.mts` 和 `.d.cts`。当 TypeScript 为 `.mts` 和 `.cts` 生成声明文件时，对应的扩展名将为 `.d.mts` 和 `.d.cts`。

使用这些扩展名完全是可选的，但即使你选择不将它们作为主要工作流的一部分，它们也往往非常有用。

### CommonJS 互操作性

Node.js 允许 ES 模块将 CommonJS 模块作为带有默认导出的 ES 模块进行导入。

```ts
// ./foo.cts
export function helper() {
  console.log('hello world!')
}

// ./bar.mts
import foo from './foo.cjs'

// prints "hello world!"
foo.helper()
```

在某些情况下，Node.js 还会从 CommonJS 模块合成具名导出，这会更加方便。在这些情况下，ES 模块可以使用“命名空间风格”的导入（即 `import * as foo from "..."`）或具名导入（即 `import { helper } from "..."`）。

```ts
// ./foo.cts
export function helper() {
  console.log('hello world!')
}

// ./bar.mts
import { helper } from './foo.cjs'

// prints "hello world!"
helper()
```

TypeScript 并不总能知道这些具名导出是否会被合成，但当从一个明确是 CommonJS 模块的文件导入时，TypeScript 会倾向于保持宽松并使用一些启发式规则。

关于互操作性，TypeScript 特有的一个语法是：

```ts
import foo = require('foo')
```

在 CommonJS 模块中，这纯粹归结为一个 `require()` 调用；而在 ES 模块中，它会导入 [`createRequire`](https://nodejs.org/api/module.html#module_module_createrequire_filename) 来达到相同的效果。这会降低代码在浏览器等不支持 `require()` 的运行时上的可移植性，但对互操作性通常很有用。相应地，你可以使用这种语法将上面的示例编写如下：

```ts
// ./foo.cts
export function helper() {
  console.log('hello world!')
}

// ./bar.mts
import foo = require('./foo.cjs')

foo.helper()
```

最后需要指出的是，从 CJS 模块导入 ESM 文件的唯一方法是使用动态 `import()` 调用。这可能会带来挑战，但这就是目前 Node.js 的行为。

你可以在此处[深入阅读关于 Node.js 中 ESM/CommonJS 互操作性的内容](https://nodejs.org/api/esm.html#esm_interoperability_with_commonjs)。

### `package.json` 中的 `exports`、`imports` 与自引用

Node.js 支持[在 `package.json` 中使用新字段 `"exports"` 定义入口点](https://nodejs.org/api/packages.html#packages_exports)。与在 `package.json` 中定义 `"main"` 相比，该字段功能更为强大，并且可以控制向使用者公开包的哪些部分。

下面是一个支持为 CommonJS 和 ESM 分别配置入口点的 `package.json`：

```jsonc
// package.json
{
  "name": "my-package",
  "type": "module",
  "exports": {
    ".": {
      // Entry-point for `import "my-package"` in ESM
      "import": "./esm/index.js",

      // Entry-point for `require("my-package") in CJS
      "require": "./commonjs/index.cjs",
    },
  },

  // CJS fall-back for older versions of Node.js
  "main": "./commonjs/index.cjs",
}
```

该特性包含诸多细节，[你可以在 Node.js 文档中阅读更多相关内容](https://nodejs.org/api/packages.html)。在此我们将重点讨论 TypeScript 如何支持它。

在 TypeScript 原有的 Node 支持中，它会查找 `"main"` 字段，然后查找与该入口对应的声明文件。例如，如果 `"main"` 指向 `./lib/index.js`，TypeScript 将查找名为 `./lib/index.d.ts` 的文件。包作者可以通过指定名为 `"types"` 的单独字段（例如 `"types": "./types/index.d.ts"`）来覆盖该行为。

新的支持对[导入条件（import conditions）](https://nodejs.org/api/packages.html)的处理方式类似。默认情况下，TypeScript 在导入条件上叠加了相同的规则——如果你从 ES 模块编写 `import`，它将查找 `import` 字段；如果从 CommonJS 模块编写，它将查找 `require` 字段。如果找到了，就会查找对应的声明文件。如果你需要为类型声明指向不同的位置，可以添加 `"types"` 导入条件。

```jsonc
// package.json
{
  "name": "my-package",
  "type": "module",
  "exports": {
    ".": {
      // Entry-point for `import "my-package"` in ESM
      "import": {
        // Where TypeScript will look.
        "types": "./types/esm/index.d.ts",

        // Where Node.js will look.
        "default": "./esm/index.js",
      },
      // Entry-point for `require("my-package") in CJS
      "require": {
        // Where TypeScript will look.
        "types": "./types/commonjs/index.d.cts",

        // Where Node.js will look.
        "default": "./commonjs/index.cjs",
      },
    },
  },

  // Fall-back for older versions of TypeScript
  "types": "./types/index.d.ts",

  // CJS fall-back for older versions of Node.js
  "main": "./commonjs/index.cjs",
}
```

> `"types"` 条件应该始终放在 `"exports"` 的最前面。

需要特别注意的是，CommonJS 入口点和 ES 模块入口点各自需要专属于自己的声明文件，即使两者之间的内容完全相同。每个声明文件都会根据其文件扩展名和 `package.json` 的 `"type"` 字段被解释为 CommonJS 模块或 ES 模块，并且所检测到的模块种类必须与 Node 为对应 JavaScript 文件检测到的模块种类相匹配，类型检查才能正确进行。如果试图使用单个 `.d.ts` 文件同时为 ES 模块入口点和 CommonJS 入口点提供类型定义，会导致 TypeScript 认为这些入口点中只有一个存在，从而给该包的使用者带来编译器错误。

TypeScript 还以类似的方式支持 [`package.json` 的 `"imports"` 字段](https://nodejs.org/api/packages.html#packages_imports)（查找对应文件旁边的声明文件），并支持[包使用自身名称进行自引用](https://nodejs.org/api/packages.html#packages_self_referencing_a_package_using_its_name)。这些特性的配置通常没有那么复杂，但同样得到了支持。

### 期待你的反馈！

随着我们继续推进 TypeScript 4.7 的开发，我们预计会对该功能进行更多文档完善和打磨。支持这些新特性是一项艰巨的任务，这也是我们希望尽早获得反馈的原因！请试用一下，并告诉我们它的表现如何。

欲了解更多信息，[你可以查看实现该功能的 PR](https://github.com/microsoft/TypeScript/pull/44501)。

## 控制模块检测策略

在 JavaScript 中引入模块时面临的一个问题是：现有的“脚本（script）”代码与新的模块代码之间存在歧义。模块中的 JavaScript 代码运行方式略有不同，作用域规则也不相同，因此工具必须对每个文件如何运行做出决策。例如，Node.js 要求模块入口点必须写在 `.mjs` 中，或者邻近的 `package.json` 带有 `"type": "module"`。TypeScript 在文件中发现任何 `import` 或 `export` 语句时就会将该文件视为模块，否则会假定 `.ts` 或 `.js` 文件是在全局作用域中运行的脚本文件。

这与 Node.js 的行为不太相符（在 Node.js 中 `package.json` 可以改变文件格式），也与 `--jsx` 设置为 `react-jsx` 时的行为不符（此时任何 JSX 文件都包含对 JSX 工厂的隐式导入）。这同样也不符合现代开发预期，因为大多数新的 TypeScript 代码都是以模块为前提编写的。

正因如此，TypeScript 4.7 引入了一个名为 `moduleDetection` 的新选项。`moduleDetection` 可以接受 3 个值：`"auto"`（默认值）、`"legacy"`（与 4.6 及更早版本相同的行为）和 `"force"`。

在 `"auto"` 模式下，TypeScript 不仅会查找 `import` 和 `export` 语句，还会检查：

- 在 `--module nodenext`/`--module node16` 下运行时，`package.json` 中的 `"type"` 字段是否设置为 `"module"`，以及
- 在 `--jsx react-jsx` 下运行时，当前文件是否为 JSX 文件

如果你希望每个文件都被视为模块，`"force"` 设置可确保将每个非声明文件都当作模块处理。无论 `module`、`moduleResolution` 和 `jsx` 如何配置，该规则均有效。

与此同时，`"legacy"` 选项则纯粹退回到旧行为，即仅通过查找 `import` 和 `export` 语句来确定文件是否为模块。

你可以在 [Pull Request](https://github.com/microsoft/TypeScript/pull/47495) 中了解有关此项变更的更多信息。

## 括号元素访问的控制流分析

TypeScript 4.7 现在可以在索引键为字面量类型和 unique symbol 时，收窄元素访问的类型。例如以下代码：

```ts
const key = Symbol()

const numberOrString = Math.random() < 0.5 ? 42 : 'hello'

const obj = {
  [key]: numberOrString,
}

if (typeof obj[key] === 'string') {
  let str = obj[key].toUpperCase()
}
```

此前，TypeScript 不会考虑针对 `obj[key]` 的任何类型守卫，根本无法得知 `obj[key]` 实际上是一个 `string`。相反，它会认为 `obj[key]` 仍然是 `string | number`，访问 `toUpperCase()` 就会引发错误。

TypeScript 4.7 现在能够识别出 `obj[key]` 是一个字符串。

这也意味着在 `--strictPropertyInitialization` 下，TypeScript 可以正确检查计算属性在构造函数体结束时是否已被初始化。

```ts
// 'key' has type 'unique symbol'
const key = Symbol()

class C {
  [key]: string

  constructor(str: string) {
    // oops, forgot to set 'this[key]'
  }

  screamString() {
    return this[key].toUpperCase()
  }
}
```

在 TypeScript 4.7 下，`--strictPropertyInitialization` 会报告一个错误，提示我们 `[key]` 属性在构造函数结束前并未被明确赋值。

我们衷心感谢 [Oleksandr Tarasiuk](https://github.com/a-tarasyuk) 提供了[这项改动](https://github.com/microsoft/TypeScript/pull/45974)！

## 对象和方法中函数推断的改进

TypeScript 4.7 现在可以对对象和数组内的函数执行更细粒度的推断。这使得这些函数的类型能够像普通参数一样，始终按照从左到右的方式顺畅传递。

```ts
declare function f<T>(arg: {
  produce: (n: string) => T
  consume: (x: T) => void
}): void

// Works
f({
  produce: () => 'hello',
  consume: (x) => x.toLowerCase(),
})

// Works
f({
  produce: (n: string) => n,
  consume: (x) => x.toLowerCase(),
})

// Was an error, now works.
f({
  produce: (n) => n,
  consume: (x) => x.toLowerCase(),
})

// Was an error, now works.
f({
  produce: function () {
    return 'hello'
  },
  consume: (x) => x.toLowerCase(),
})

// Was an error, now works.
f({
  produce() {
    return 'hello'
  },
  consume: (x) => x.toLowerCase(),
})
```

在上面的某些示例中，此前推断之所以失败，是因为在为 `T` 找到合适类型之前，获取其 `produce` 函数的类型会间接请求 `arg` 的类型。TypeScript 现在会收集可能对 `T` 的推断类型有贡献的函数，并对它们进行惰性推断。

欲了解更多信息，可以[查看我们对推断过程所做的具体修改](https://github.com/microsoft/TypeScript/pull/48538)。

## 实例化表达式

有时函数的泛型可能比我们需要的更为宽泛。例如，假设我们有一个 `makeBox` 函数：

```ts
interface Box<T> {
  value: T
}

function makeBox<T>(value: T) {
  return { value }
}
```

也许我们想要创建一组更具针对性的特化函数，专门用来制造 `Wrench` 和 `Hammer` 的 `Box`。在以前，我们必须将 `makeBox` 包装在其他函数中，或者为 `makeBox` 的别名指定显式类型：

```ts
function makeHammerBox(hammer: Hammer) {
  return makeBox(hammer)
}

// or...

const makeWrenchBox: (wrench: Wrench) => Box<Wrench> = makeBox
```

这些方式可行，但包装一个对 `makeBox` 的调用略显冗余，而编写 `makeWrenchBox` 的完整签名又会显得笨重。理想情况下，我们希望能够直接为 `makeBox` 起别名，同时替换其签名中的所有泛型参数。

TypeScript 4.7 正好支持了这一点！我们现在可以直接向函数和构造函数传递类型参数。

```ts
const makeHammerBox = makeBox<Hammer>
const makeWrenchBox = makeBox<Wrench>
```

因此，我们可以特化 `makeBox` 以接受更具体的类型，并拒绝其他任何类型：

```ts
const makeStringBox = makeBox<string>

// TypeScript correctly rejects this.
makeStringBox(42)
```

该逻辑同样适用于诸如 `Array`、`Map` 和 `Set` 之类的构造函数：

```ts
// Has type `new () => Map<string, Error>`
const ErrorMap = Map<string, Error>

// Has type `// Map<string, Error>`
const errorMap = new ErrorMap()
```

当向函数或构造函数传入类型参数时，它会生成一个新类型：该类型保留了所有具有兼容类型参数列表的签名，并用给定的类型参数替换对应的类型参数。任何其他签名都将被丢弃，因为 TypeScript 会认为它们不打算被使用。

有关此特性的更多信息，请[参阅 Pull Request](https://github.com/microsoft/TypeScript/pull/47607)。

## `infer` 类型变量上的 `extends` 约束

条件类型是一项高阶特性。它们允许我们针对类型的形状进行匹配和推断，并据此做出判断。例如，我们可以编写一个条件类型：如果元组类型包含类似 `string` 的类型，则返回其第一个元素。

```ts
type FirstIfString<T> = T extends [infer S, ...unknown[]]
  ? S extends string
    ? S
    : never
  : never

// string
type A = FirstIfString<[string, number, number]>

// "hello"
type B = FirstIfString<['hello', number, number]>

// "hello" | "world"
type C = FirstIfString<['hello' | 'world', boolean]>

// never
type D = FirstIfString<[boolean, number, string]>
```

`FirstIfString` 会匹配任何至少包含一个元素的元组，并将第一个元素的类型提取为 `S`。然后检查 `S` 是否与 `string` 兼容，如果兼容则返回该类型。

注意，编写此逻辑时我们不得不使用了两个条件类型。我们也可以将 `FirstIfString` 写成如下形式：

```ts
type FirstIfString<T> = T extends [string, ...unknown[]]
  ? // Grab the first type out of `T`
    T[0]
  : never
```

这样可行，但稍微偏向“手动”且不够声明式。我们没有单纯在类型上进行模式匹配并给首个元素命名，而是必须使用 `T[0]` 提取出 `T` 的第 `0` 个元素。如果面对的是比元组更复杂的类型，这种写法可能会棘手得多，因此 `infer` 能够简化这些场景。

使用嵌套条件先推断出某个类型然后再对推断出的类型进行匹配是非常常见的。为了避免第二层嵌套，TypeScript 4.7 现在允许你在任何 `infer` 类型上放置约束：

```ts
type FirstIfString<T> = T extends [infer S extends string, ...unknown[]]
  ? S
  : never
```

这样一来，当 TypeScript 匹配 `S` 时，它还会确保 `S` 必须是 `string`。如果 `S` 不是 `string`，则会走假分支（在这些情况下为 `never`）。

欲了解更多详情，可以[在 GitHub 上阅读此变更](https://github.com/microsoft/TypeScript/pull/48112)。

## 类型参数的可选型变标注

考虑以下类型：

```ts
interface Animal {
  animalStuff: any
}

interface Dog extends Animal {
  dogStuff: any
}

// ...

type Getter<T> = () => T

type Setter<T> = (value: T) => void
```

假设我们有两个不同的 `Getter` 实例。确定任意两个不同的 `Getter` 是否可以互相替代完全取决于 `T`。对于赋值 `Getter<Dog>`&nbsp;&rarr;&nbsp;`Getter<Animal>` 是否有效的情况，我们必须检查 `Dog`&nbsp;&rarr;&nbsp;`Animal` 是否有效。因为 `T` 的每个类型都只是在相同的“方向”上关联，所以我们说 `Getter` 类型在 `T` 上是*协变（covariant）*的。另一方面，检查 `Setter<Dog>`&nbsp;&rarr;&nbsp;`Setter<Animal>` 是否有效涉及检查 `Animal`&nbsp;&rarr;&nbsp;`Dog` 是否有效。方向上的这种“翻转”有点类似于数学中检查 &minus;*x*&nbsp;&lt;&nbsp;*&minus;y* 是否等同于检查 *y*&nbsp;&lt;&nbsp;*x*。当我们必须像这样翻转方向来比较 `T` 时，我们说 `Setter` 在 `T` 上是*逆变（contravariant）*的。

在 TypeScript 4.7 中，我们现在能够*显式*指定类型参数的型变（variance）。

因此现在，如果我们想显式声明 `Getter` 在 `T` 上是协变的，可以为其添加一个 `out` 修饰符：

```ts
type Getter<out T> = () => T
```

类似地，如果我们想显式声明 `Setter` 在 `T` 上是逆变的，可以为其添加一个 `in` 修饰符：

```ts
type Setter<in T> = (value: T) => void
```

这里之所以使用 `out` 和 `in`，是因为类型参数的型变取决于它是用于*输出（output）*还是*输入（input）*。与其思考型变概念，不如直接思考 `T` 是用于输出位置还是输入位置。

此外也存在同时使用 `in` 和 `out` 的情况：

```ts
interface State<in out T> {
  get: () => T
  set: (value: T) => void
}
```

当 `T` 同时用于输出和输入位置时，它就变成了*不变（invariant）*。两个不同的 `State<T>` 除非其 `T` 相同，否则无法互换。换句话说，`State<Dog>` 和 `State<Animal>` 不能相互替代。

从严格的技术角度来说，在一个纯粹的结构化类型系统中，类型参数及其型变其实并不重要——你只需将具体类型代入每个类型参数所在的位置，然后检查每个匹配的成员是否结构兼容即可。既然 TypeScript 使用的是结构化类型系统，那为什么我们还会对类型参数的型变感兴趣呢？为什么我们想要标注它们呢？

一个原因是，这对读者一目了然地显式看清类型参数的使用方式非常有帮助。对于复杂得多的类型，可能很难看出一个类型是用于读取、写入还是两者兼有。如果我们忘记说明该类型参数的使用方式，TypeScript 也会为我们提供帮助。例如，如果我们在 `State` 上忘记同时指定 `in` 和 `out`，就会收到一个错误：

```ts
interface State<out T> {
  //          ~~~~~
  // error!
  // Type 'State<sub-T>' is not assignable to type 'State<super-T>' as implied by variance annotation.
  //   Types of property 'set' are incompatible.
  //     Type '(value: sub-T) => void' is not assignable to type '(value: super-T) => void'.
  //       Types of parameters 'value' and 'value' are incompatible.
  //         Type 'super-T' is not assignable to type 'sub-T'.
  get: () => T
  set: (value: T) => void
}
```

另一个原因则是准确性与速度！TypeScript 已经尝试推断类型参数的型变作为一项优化。通过这种方式，它可以在合理的时间内对较大的结构类型进行类型检查。提前计算型变允许类型检查器跳过更深层次的比较，只比较类型参数，这比一遍又一遍地比较类型的完整结构要*快得多*。但是通常在某些情况下，这种计算仍然相当昂贵，并且计算可能会发现无法精确解析的循环引用，这意味着类型的型变没有明确的答案：

```ts
type Foo<T> = {
  x: T
  f: Bar<T>
}

type Bar<U> = (x: Baz<U[]>) => void

type Baz<V> = {
  value: Foo<V[]>
}

declare let foo1: Foo<unknown>
declare let foo2: Foo<string>

foo1 = foo2 // Should be an error but isn't ❌
foo2 = foo1 // Error - correct ✅
```

提供显式标注可以加快这些循环引用处的类型检查并提供更好的准确性。例如，在上述示例中将 `T` 标记为不变有助于阻止有问题的赋值：

```diff
- type Foo<T> = {
+ type Foo<in out T> = {
      x: T;
      f: Bar<T>;
  }
```

我们并不一定建议为每个类型参数都标注其型变；例如，可能会（但不建议）将型变设置得比实际需要的更严格，如果某项实际上只是协变、逆变甚至独立无关的，TypeScript 也不会阻止你将其标记为不变。因此，如果你确实选择添加显式型变标记，我们建议深思熟虑并准确地使用它们。

但如果你正在处理深度递归类型，尤其是库作者，你可能会对利用这些标注让用户受益感兴趣。这些标注可以在准确性和类型检查速度上带来双重提升，甚至可能改善用户的代码编辑体验。判断型变计算何时成为类型检查时间的瓶颈可以通过实验完成，并借助我们的 [analyze-trace](https://github.com/microsoft/typescript-analyze-trace) 等工具进行分析。

有关此特性的更多详细信息，可以[在 Pull Request 中阅读](https://github.com/microsoft/TypeScript/pull/48240)。

## 使用 `moduleSuffixes` 自定义解析策略

TypeScript 4.7 现在支持 `moduleSuffixes` 选项来自定义模块说明符（module specifiers）的查找方式。

```jsonc
{
  "compilerOptions": {
    "moduleSuffixes": [".ios", ".native", ""],
  },
}
```

基于上述配置，如下形式的导入：

```ts
import * as foo from './foo'
```

将尝试依次查找相对文件 `./foo.ios.ts`、`./foo.native.ts`，最后查找 `./foo.ts`。

<aside>

注意，`moduleSuffixes` 中的空字符串 `""` 是必需的，以便 TypeScript 也能查找到 `./foo.ts`。在某种意义上，`moduleSuffixes` 的默认值就是 `[""]`。

</aside>

该特性对于 React Native 项目非常有用，其中每个目标平台都可以使用带有不同 `moduleSuffixes` 的独立 `tsconfig.json`。

[`moduleSuffixes` 选项](https://github.com/microsoft/TypeScript/pull/48189)由 [Adam Foxman](https://github.com/afoxman) 贡献，在此深表感谢！

## resolution-mode

在 Node 的 ECMAScript 解析策略下，包含文件的模式以及你使用的语法决定了导入如何解析；然而，有时需要在 ECMAScript 模块中引用 CommonJS 模块的类型，反之亦然。

TypeScript 现在允许使用 `/// <reference types="..." />` 指令：

```ts
/// <reference types="pkg" resolution-mode="require" />

// or

/// <reference types="pkg" resolution-mode="import" />
```

此外，在 TypeScript 的 nightly 版本中，`import type` 可以指定导入断言（import assertion）来实现类似的功能：

```ts
// Resolve `pkg` as if we were importing with a `require()`
import type { TypeFromRequire } from 'pkg' assert {
  'resolution-mode': 'require',
}

// Resolve `pkg` as if we were importing with an `import`
import type { TypeFromImport } from 'pkg' assert {
  'resolution-mode': 'import',
}

export interface MergedType extends TypeFromRequire, TypeFromImport {}
```

这些导入断言也可以用于 `import()` 类型：

```ts
export type TypeFromRequire = import('pkg', {
  assert: { 'resolution-mode': 'require' },
}).TypeFromRequire

export type TypeFromImport = import('pkg', {
  assert: { 'resolution-mode': 'import' },
}).TypeFromImport

export interface MergedType extends TypeFromRequire, TypeFromImport {}
```

`import type` 和 `import()` 语法仅在 [TypeScript 的 nightly 构建版本](https://www.typescriptlang.org/docs/handbook/nightly-builds.html)中支持 `resolution-mode`。你很可能会收到如下错误：

```
Resolution mode assertions are unstable. Use nightly TypeScript to silence this error. Try updating with 'npm install -D typescript@next'.
```

如果你确实在 TypeScript 的 nightly 版本中使用了此功能，[欢迎在此 issue 中提供反馈](https://github.com/microsoft/TypeScript/issues/49055)。

你可以查看分别针对[引用指令](https://github.com/microsoft/TypeScript/pull/47732)和[类型导入断言](https://github.com/microsoft/TypeScript/pull/47807)的相应改动。

## 跳转到源码定义（Go to Source Definition）

TypeScript 4.7 包含对名为“跳转到源码定义（Go To Source Definition）”的新实验性编辑器命令的支持。它与“跳转到定义（Go To Definition）”类似，但它绝不会返回声明文件内部的结果。相反，它会尝试查找对应的*实现*文件（例如 `.js` 或 `.ts` 文件），并在那里查找定义——即使这些文件通常被 `.d.ts` 文件遮蔽。

当你需要查看从库中导入的函数的实现、而不是其在 `.d.ts` 文件中的类型声明时，这往往非常方便。

![在 yargs 包的使用上执行“Go to Source Definition”命令会将编辑器跳转到 yargs 中的 index.cjs 文件。](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2022/05/go-to-source-definition-4-7-v1.gif)

你可以在最新版本的 Visual Studio Code 中体验这一新命令。不过请注意，该功能仍处于预览阶段，存在一些已知限制。在某些情况下，TypeScript 会使用启发式规则猜测哪个 `.js` 文件与给定的定义结果相对应，因此这些结果可能不够精确。Visual Studio Code 目前也尚未标明某个结果是否属于猜测，但这是我们正在合作推进的方向。

你可以留下关于该功能的反馈、阅读已知限制，或在[我们专门的反馈 issue](https://github.com/microsoft/TypeScript/issues/49003) 中了解更多信息。

## 感知分组的“整理导入”（Group-Aware Organize Imports）

TypeScript 为 JavaScript 和 TypeScript 提供了“整理导入（Organize Imports）”编辑器功能。遗憾的是，它之前可能显得有些生硬，往往只是简单粗暴地对导入语句进行排序。

例如，如果你在以下文件上运行“整理导入”：

```ts
// local code
import * as bbb from './bbb'
import * as ccc from './ccc'
import * as aaa from './aaa'

// built-ins
import * as path from 'path'
import * as child_process from 'child_process'
import * as fs from 'fs'

// some code...
```

你会得到类似如下的结果：

```ts
// local code
import * as child_process from 'child_process'
import * as fs from 'fs'
// built-ins
import * as path from 'path'
import * as aaa from './aaa'
import * as bbb from './bbb'
import * as ccc from './ccc'

// some code...
```

这……并不理想。诚然，我们的导入按路径排序了，注释和换行符也保留了下来，但并没有按照我们预期的形式排列。很多时候，如果我们按特定方式对导入进行了分组，我们希望能够保持这种分组。

TypeScript 4.7 以感知分组的方式执行“整理导入”。在上述代码上运行它，看起来会更符合你的预期：

```ts
// local code
import * as aaa from './aaa'
import * as bbb from './bbb'
import * as ccc from './ccc'

// built-ins
import * as child_process from 'child_process'
import * as fs from 'fs'
import * as path from 'path'

// some code...
```

我们要向 [Minh Quy](https://github.com/MQuy) 致谢，是他提供了[这项功能](https://github.com/microsoft/TypeScript/pull/48330)。

## 对象方法代码片段补全

TypeScript 现在为对象字面量方法提供了代码片段补全功能。当在对象中补全成员时，TypeScript 不仅会提供仅包含方法名的常规补全项，还会为完整的方法定义提供单独的补全项！

![从对象补全完整的方法签名](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2022/05/object-method-completions-4-7-v2.gif)

更多详情，请[参阅实现该功能的 Pull Request](https://github.com/microsoft/TypeScript/pull/48168)。

## 破坏性变更

### `lib.d.ts` 更新

尽管 TypeScript 竭力避免重大破坏性变更，但内置库哪怕是很小的变化也可能引发问题。我们预计 DOM 和 `lib.d.ts` 的更新不会带来重大破坏，但可能会有一些细微影响。

### JSX 中更严格的展开运算符检查

在 JSX 中编写 `...spread` 时，TypeScript 现在会实施更严格的检查，以确保给定的类型确实是一个对象。因此，具有 `unknown` 和 `never` 类型的值（以及更少见的裸 `null` 和 `undefined`）不能再展开到 JSX 元素中。

例如在以下示例中：

```tsx
import * as React from 'react'

interface Props {
  stuff?: string
}

function MyComponent(props: unknown) {
  return <div {...props} />
}
```

现在会收到类似如下的错误：

```
Spread types may only be created from object types.
```

这使得该行为与对象字面量中的展开操作更加一致。

欲了解更多详情，请[查看 GitHub 上的改动](https://github.com/microsoft/TypeScript/pull/48570)。

### 模板字符串表达式更严格的检查

当在模板字符串中使用 `symbol` 值时，会在 JavaScript 中触发运行时错误：

```js
let str = `hello ${Symbol()}`
// TypeError: Cannot convert a Symbol value to a string
```

因此，TypeScript 也会报错；此外，TypeScript 现在还会检查被某种方式约束为 symbol 的泛型值是否被用于模板字符串中：

```ts
function logKey<S extends string | symbol>(key: S): S {
  // Now an error.
  console.log(`${key} is the key`)
  return key
}

function get<T, K extends keyof T>(obj: T, key: K) {
  // Now an error.
  console.log(`Grabbing property '${key}'.`)
  return obj[key]
}
```

TypeScript 现在会给出以下错误：

```
Implicit conversion of a 'symbol' to a 'string' will fail at runtime. Consider wrapping this expression in 'String(...)'.
```

在某些情况下，你可以按照错误信息的建议，将表达式包装在 `String` 调用中来解决此问题：

```ts
function logKey<S extends string | symbol>(key: S): S {
  // No longer an error.
  console.log(`${String(key)} is the key`)
  return key
}
```

而在其他情况下，这个错误可能过于吹毛求疵，你甚至可能在使用 `keyof` 时根本不想允许 `symbol` 键。在这种情况下，你可以切换为 `string & keyof ...`：

```ts
function get<T, K extends string & keyof T>(obj: T, key: K) {
  // No longer an error.
  console.log(`Grabbing property '${key}'.`)
  return obj[key]
}
```

欲了解更多信息，你可以[查看实现该功能的 Pull Request](https://github.com/microsoft/TypeScript/pull/44578)。

### `LanguageServiceHost` 上的 `readFile` 方法不再是可选的

如果你正在创建 `LanguageService` 实例，那么所提供的 `LanguageServiceHost` 将需要提供 `readFile` 方法。此变更是为了支持新的 `moduleDetection` 编译器选项所必需的。

你可以[在此处阅读关于该变更的更多信息](https://github.com/microsoft/TypeScript/pull/47495)。

### 只读元组拥有只读的 `length` 属性

只读元组现在将其 `length` 属性视为 `readonly`。对于固定长度的元组，这几乎从未引起注意，但对于末尾带有可选元素和剩余元素类型的元组，此前是一处疏忽。

因此，以下代码现在将报错：

```ts
function overwriteLength(tuple: readonly [string, string, string]) {
  // Now errors.
  tuple.length = 7
}
```

你可以在此处[深入了解此变更](https://github.com/microsoft/TypeScript/pull/47717)。
