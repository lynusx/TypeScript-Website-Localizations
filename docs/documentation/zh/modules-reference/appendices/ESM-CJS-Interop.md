---
title: 模块 - ESM/CJS 互操作性
short: ESM/CJS 互操作性
layout: docs
permalink: /zh/docs/handbook/modules/appendices/esm-cjs-interop.html
oneline: 深入解析 ES 模块与 CommonJS 模块之间的互操作性
---

时间回到 2015 年，你正在编写一个将 ESM 转译为 CJS 的转译器。当时并没有关于该如何实现此功能的规范；你手中仅有一份关于 ES 模块之间应当如何相互交互的规范、对 CommonJS 模块之间如何相互交互的了解，以及善于琢磨解决问题的敏锐直觉。设想一个进行导出的 ES 模块：

```ts
export const A = {}
export const B = {}
export default 'Hello, world!'
```

你会如何将其转换为 CommonJS 模块？回顾一下，默认导出本质上只是具有特殊语法的具名导出，因此似乎只有一种选择：

```ts
exports.A = {}
exports.B = {}
exports.default = 'Hello, world!'
```

这是一个极佳的类比对应，而且它也能让你在导入端实现类似的对应：

```ts
import hello, { A, B } from './module'
console.log(hello, A, B)

// 转译为：

const module_1 = require('./module')
console.log(module_1.default, module_1.A, module_1.B)
```

到目前为止，CJS 世界中的一切都与 ESM 世界中的一切一一对应。将上述等价关系再推进一步，我们可以看到：

```ts
import * as mod from './module'
console.log(mod.default, mod.A, mod.B)

// 转译为：

const mod = require('./module')
console.log(mod.default, mod.A, mod.B)
```

你可能会注意到，在这种方案中，无法通过编写 ESM 导出来生成将函数、类或原始值赋值给 `exports` 的输出：

```ts
// @Filename: exports-function.js
module.exports = function hello() {
  console.log('Hello, world!')
}
```

但现有的 CommonJS 模块频繁采用这种形式。经由我们的转译器处理后，ESM 导入该如何访问这个模块呢？我们刚才已经确定，命名空间导入（`import *`）会转译为普通的 `require` 调用，因此我们可以支持如下输入：

```ts
import * as hello from './exports-function'
hello()

// 转译为：

const hello = require('./exports-function')
hello()
```

我们的输出在运行时可以工作，但我们遇到了规范合规性问题：根据 JavaScript 规范，命名空间导入始终解析为一个[_模块命名空间对象（Module Namespace Object）_](https://tc39.es/ecma262/#sec-module-namespace-objects)，即一个成员为该模块所有导出项的对象。在这种情况下，`require` 将返回函数 `hello`，但 `import *` 绝不可能返回一个函数。我们所假设的对应关系在此失效了。

此时值得退一步来明确我们的**目标**究竟是什么。模块机制一旦纳入 ES2015 规范，支持将 ESM 降级为 CJS 的转译器便应运而生，让用户早在运行时提供支持之前就能采用这一新语法。人们甚至普遍认为，编写 ESM 代码是让新项目“面向未来”的好方法。要使这一假设成立，就必须存在一条无缝迁移路径：一旦运行时实现了原生支持，就可以直接原生执行 ESM 输入，而不再需要执行转译器生成的 CJS 输出。其目标是找到一种将 ESM 降级为 CJS 的方法，以便未来在运行时中，任何或所有转译输出都可以被其真实的 ESM 输入直接替换，而不会产生任何可被观察到的行为变化。

遵循规范，转译器很容易找到一组转换规则，使其转译后的 CommonJS 输出语义与 ESM 输入的规范语义相匹配（箭头代表导入）：

![A flowchart with two similar flows side-by-side. Left: ESM. Right: ESM transpiled to CJS. In the ESM flow: "Importing module" flows to "Imported module" through arrow labeled "specified behavior". In the ESM transpiled to CJS flow: "Importing module" flows to "Imported module" through arrow labeled "designed based on spec".](../diagrams/esm-cjs-interop.md-1.svg)

然而，CommonJS 模块（以原生 CommonJS 编写，而非由 ESM 转译而来）在 Node.js 生态中早已根深蒂固，因此以 ESM 编写并转译为 CJS 的模块不可避免地会开始“导入”以 CommonJS 编写的模块。然而，ES2015 并没有规范这种互操作行为，且在任何实际运行时中也尚不存在该行为。

![A flowchart with three areas side-by-side. Left: ESM. Middle: True CJS. Right: ESM transpiled to CJS. Left: ESM "Importing module" flows to ESM "Imported module" through arrow labeled "specified behavior," and to True CJS "Imported module" through dotted arrow labeled "unspecified behavior." Right: ESM transpiled to CJS "Importing module" flows to ESM transpiled to CJS "Imported module" through arrow labeled "designed based on spec," and to True CJS "Imported module" through dotted arrow labeled "❓🤷‍♂️❓"](../diagrams/esm-cjs-interop.md-2.svg)

即使转译器作者什么都不做，转译代码中输出的 `require` 调用与现有 CJS 模块中定义的 `exports` 之间的既有语义本身也会产生一种行为。为了让用户在运行时支持原生 ESM 后能够无缝过渡，这种行为必须与运行时最终选择实现的行为完全一致。

预测运行时将支持何种互操作行为，并不仅限于 ESM 导入“原生 CJS”模块。ESM 能否将“由 ESM 转译而来的 CJS”与“原生 CJS”区分开来，以及 CJS 能否 `require` ES 模块，也都是未明确的。甚至 ESM 导入是否会采用与 CJS `require` 调用相同的模块解析算法也是未知的。所有这些变量都必须被准确预测，才能为转译器用户提供一条通往原生 ESM 的无缝迁移之路。

## `allowSyntheticDefaultImports` 与 `esModuleInterop`

让我们回到前文提到的规范合规性问题，即 `import *` 转译为 `require`：

```ts
// 根据规范是无效的：
import * as hello from './exports-function'
hello()

// 但转译后可以正常工作：
const hello = require('./exports-function')
hello()
```

当 TypeScript 最初支持编写和转译 ES 模块时，编译器通过对任何将其 `exports` 不是类命名空间对象的模块进行命名空间导入时报错，来应对该问题：

```ts
import * as hello from './exports-function'
// TS2497              ^^^^^^^^^^^^^^^^^^^^
// 外部模块 '"./exports-function"' 解析为一个非模块实体，
// 无法使用此结构导入。
```

当时唯一的变通方案是让用户回退到使用旧版的 TypeScript 导入语法（表示 CommonJS 的 `require`）：

```ts
import hello = require('./exports-function')
```

迫使用户退回到非 ESM 语法，本质上等于承认：“我们不知道未来是否或者如何能通过 ESM 导入来访问像 `"./exports-function"` 这样的 CJS 模块，但我们清楚它**绝不能**通过 `import *` 来访问，即便在现有的转译方案下它在运行时可以工作。”这未能达成在不作修改的情况下将该文件迁移到真实 ESM 的目标，但允许 `import *` 链接到一个函数的替代做法同样也无法达成这一目标。在如今的 TypeScript 中，如果禁用了 `allowSyntheticDefaultImports` 和 `esModuleInterop`，这依然是其默认行为。

> 遗憾的是，这还是略微简化了事实——TypeScript 并没有通过这个报错完全避开合规性问题，因为只要函数声明与命名空间声明合并（即便该命名空间是空的），它就允许对该函数进行命名空间导入并保留其调用签名。因此，虽然导出一个纯函数的模块会被识别为“非模块实体（non-module entity）”：
>
> ```ts
> declare function $(selector: string): any
> export = $ // 无法对其进行 `import *` 👍
> ```
>
> 但一个本应毫无意义的改动却能让这一无效导入通过类型检查且不报错：
>
> ```ts
> declare namespace $ {}
> declare function $(selector: string): any
> export = $ // 允许对其进行 `import *` 并调用它 😱
> ```

与此同时，其他转译器正在探索解决同一问题的方法。当时的思考过程大致如下：

1. 要导入一个导出函数或原始值的 CJS 模块，我们显然需要使用默认导入。命名空间导入是不合法的，而具名导入在这里也没有意义。
2. 很可能这意味着实现 ESM/CJS 互操作的运行时会选择让 CJS 模块的默认导入**始终**直接链接到整个 `exports`，而不仅仅是在 `exports` 为函数或原始值时才这样做。
3. 因此，原生 CJS 模块的默认导入应该表现得就像 `require` 调用一样。但我们需要一种方法来区分原生 CJS 模块与我们转译出来的 CJS 模块，以便我们仍然可以将 `export default "hello"` 转译为 `exports.default = "hello"`，并让**该**模块的默认导入链接到 `exports.default`。简而言之，对我们自己转译的模块进行默认导入需要按一种方式工作（以模拟 ESM 到 ESM 的导入），而对任何其他现有 CJS 模块的默认导入则需要按另一种方式工作（以模拟我们认为 ESM 到 CJS 导入将具备的工作方式）。
4. 当我们把 ES 模块转译为 CJS 时，我们在输出中添加一个特殊的额外字段：
   ```ts
   exports.A = {}
   exports.B = {}
   exports.default = 'Hello, world!'
   // 特殊的标志！
   exports.__esModule = true
   ```
   这样在转译默认导入时就可以对该字段进行检查：
   ```ts
   // import hello from "./module";
   const _mod = require('./module')
   const hello = _mod.__esModule ? _mod.default : _mod
   ```

`__esModule` 标记最早出现在 Traceur 中，随后很快被 Babel、SystemJS 和 Webpack 采用。TypeScript 在 1.8 中添加了 `allowSyntheticDefaultImports`，允许类型检查器将默认导入直接链接到缺少 `export default` 声明的任何模块类型的 `exports`，而非 `exports.default`。该标志并没有改变导入或导出的输出方式，但它允许默认导入体现出其他转译器处理它们的方式。具体而言，它允许使用默认导入来解析“非模块实体”，而在以前使用 `import *` 则会报错：

```ts
// 错误：
import * as hello from './exports-function'

// 旧的变通方案：
import hello = require('./exports-function')

// 使用 `allowSyntheticDefaultImports` 的新方式：
import hello from './exports-function'
```

这通常足以让 Babel 和 Webpack 用户编写已在这些系统中正常运行的代码而不会引发 TypeScript 报错，但这只是一个权宜之计，遗留了几个未解决的问题：

1. Babel 及其他工具会根据目标模块上是否存在 `__esModule` 属性来改变其默认导入行为，但 `allowSyntheticDefaultImports` 仅在目标模块的类型中未找到默认导出时才启用**回退（fallback）**行为。如果目标模块具有 `__esModule` 标记但**没有**默认导出，就会产生不一致。转译器和打包工具仍然会将此类模块的默认导入链接到其 `exports.default`（其值为 `undefined`），在理想情况下 TypeScript 应当对此报错，因为真正的 ESM 导入如果无法链接就会报错。但在启用了 `allowSyntheticDefaultImports` 的情况下，TypeScript 会认为对此类模块的默认导入会链接到整个 `exports` 对象，从而允许将具名导出作为其属性进行访问。
2. `allowSyntheticDefaultImports` 并没有改变命名空间导入的类型推断，导致了一种奇怪的不一致：两者都可以使用，并且具有相同的类型：
   ```ts
   // @Filename: exportEqualsObject.d.ts
   declare const obj: object
   export = obj

   // @Filename: main.ts
   import objDefault from './exportEqualsObject'
   import * as objNamespace from './exportEqualsObject'

   // 这在运行时应该为 true，但 TypeScript 报错：
   objNamespace.default === objDefault
   //           ^^^^^^^ 属性 'default' 不存在于类型 'typeof import("./exportEqualsObject")' 上。
   ```
3. 最重要的是，`allowSyntheticDefaultImports` 并没有改变 `tsc` 生成的 JavaScript 代码。因此，虽然只要代码被送入 Babel 或 Webpack 等其他工具中，该标志就能提供更准确的类型检查，但对于使用 `tsc` 生成 `--module commonjs` 代码并在 Node.js 中运行的用户来说，它带来了真正的风险。如果他们在遇到 `import *` 报错时启用了 `allowSyntheticDefaultImports`，表面上看似乎修复了问题，但实际上这只是掩盖了构建时的报错，输出的代码在 Node 中运行时依然会崩溃。

TypeScript 在 2.7 中引入了 `esModuleInterop` 标志，改进了导入的类型检查，以解决 TypeScript 的分析与现有转译器及打包工具所用互操作行为之间剩余的不一致；更关键的是，它采用了转译器在多年前就已采用的基于 `__esModule` 条件判断的 CommonJS 代码输出。（另一个用于 `import *` 的新输出辅助函数确保了结果始终是一个去除了调用签名的对象，彻底解决了前述“resolves to a non-module entity”报错未能完全规避的规范合规性问题。）最终，在该新标志启用后，TypeScript 的类型检查、TypeScript 的代码输出以及转译和打包生态系统的其余部分在 CJS/ESM 互操作方案上达成了一致——该方案既符合规范，而且可能在未来被 Node 所采纳。

## Node.js 中的互操作性

Node.js 在 v12 中默认正式提供了对 ES 模块的支持（无需标志）。就像打包工具和转译器多年前所做的那样，Node.js 为 CommonJS 模块的 `exports` 对象赋予了一个“合成默认导出（synthetic default export）”，允许从 ESM 通过默认导入来访问模块的全部内容：

```ts
// @Filename: export.cjs
module.exports = { hello: 'world' }

// @Filename: import.mjs
import greeting from './export.cjs'
greeting.hello // "world"
```

这是实现无缝迁移的一大胜利！但遗憾的是，相似之处大致也就到此为止了。

### 不支持 `__esModule` 检测（“双重 default”问题）

Node.js 无法通过识别 `__esModule` 标记来改变其默认导入行为。因此，带有“默认导出”的转译模块，在被另一个转译模块“导入”时表现为一种行为，而在 Node.js 中被真正的 ES 模块导入时则表现为另一种行为：

```ts
// @Filename: node_modules/dependency/index.js
exports.__esModule = true
exports.default = function doSomething() {
  /*...*/
}

// @Filename: transpile-vs-run-directly.{js/mjs}
import doSomething from 'dependency'
// 转译后可以运行，但在 Node.js ESM 中不是一个函数：
doSomething()
// 转译后不存在，但在 Node.js ESM 中可以运行：
doSomething.default()
```

转译环境下的默认导入只有在目标模块缺少 `__esModule` 标志时才会生成合成默认导出，而 Node.js 则**始终**合成一个默认导出，导致转译模块上出现了“双重 default（double default）”。

### 不可靠的具名导出

除了将 CommonJS 模块的 `exports` 对象作为默认导出提供之外，Node.js 还尝试寻找 `exports` 的属性并将其作为具名导入暴露出来。如果能够正常工作，该行为与打包工具和转译器是一致的；然而，Node.js 使用[句法静态分析（syntactic analysis）](https://github.com/nodejs/cjs-module-lexer)在任何代码执行之前合成具名导出，而转译模块则是在运行时解析其具名导入。其结果是，在转译模块中可以正常工作的 CJS 模块导入，在 Node.js 中可能无法正常工作：

```ts
// @Filename: named-exports.cjs
exports.hello = 'world'
exports['worl' + 'd'] = 'hello'

// @Filename: transpile-vs-run-directly.{js/mjs}
import { hello, world } from './named-exports.cjs'
// `hello` 正常工作，但 `world` 在 Node.js 中缺失 💥

import mod from './named-exports.cjs'
mod.world
// 从 default 访问属性始终有效 ✅
```

### 在 Node.js v22 之前无法 `require` 真正的 ES 模块

原生 CommonJS 模块可以 `require` 一个由 ESM 转译为 CJS 的模块，因为它们在运行时都是 CommonJS。但在早于 v22.12.0 的 Node.js 版本中，如果 `require` 解析到了 ES 模块，程序就会崩溃。这意味着已发布的库无法在不破坏其 CommonJS（原生或转译）使用者的情况下，从转译模块迁移到原生 ESM：

```ts
// @Filename: node_modules/dependency/index.js
export function doSomething() {
  /* ... */
}

// @Filename: dependent.js
import { doSomething } from 'dependency'
// ✅ 如果依赖方和被依赖方均经过转译，则正常工作
// ✅ 如果依赖方和被依赖方均为真正的 ESM，则正常工作
// ✅ 如果依赖方是真正的 ESM 且被依赖方经过转译，则正常工作
// 💥 如果依赖方经过转译且被依赖方是真正的 ESM，则会崩溃
```

### 不同的模块解析算法

Node.js 引入了一套用于解析 ESM 导入的新模块解析算法，它与长期用于解析 `require` 调用的算法有着显著差异。虽然这与 CJS 和 ES 模块之间的互操作性没有直接关联，但这种差异也是从转译模块向原生 ESM 无缝迁移可能无法实现的另一个原因：

```ts
// @Filename: add.js
export function add(a, b) {
  return a + b
}

// @Filename: math.js
export * from './add'
//            ^^^^^^^
// 转译为 CJS 时可以工作，
// 但在 Node.js ESM 中
// 必须为 "./add.js"。
```

## 结论

显然，从转译模块到 ESM 的无缝迁移是不可能的，至少在 Node.js 中是如此。那么这对我们意味着什么呢？

### 设置正确的 `module` 编译器选项至关重要

由于不同宿主之间的互操作规则各不相同，TypeScript 只有理解它所看到的每个文件代表哪种模块，以及应当对其应用哪套规则，才能提供正确的类型检查行为。这正是 `module` 编译器选项的意义所在。（特别是，要在 Node.js 中运行的代码受到比打包工具处理的代码更严格的规则约束。除非将 `module` 设置为 `node16`、`node18` 或 `nodenext`，否则编译器的输出不会针对 Node.js 兼容性进行检查。）

### 包含 CommonJS 代码的应用程序应始终启用 `esModuleInterop`

在由 `tsc` 输出 JavaScript 文件的 TypeScript **应用程序**（相对于供他人使用的库）中，是否启用 `esModuleInterop` 并不会产生重大后果。针对某些类型模块编写导入语句的方式会有所改变，但 TypeScript 的类型检查与代码输出是保持同步的，因此无报错的代码在任一模式下运行都应当是安全的。在这种情况下，不启用 `esModuleInterop` 的弊端在于，它允许你编写语义明显违背 ECMAScript 规范的 JavaScript 代码，这会混淆关于命名空间导入的直觉认知，并增加未来迁移到原生 ES 模块运行时的难度。

另一方面，在经由第三方转译器或打包工具处理的应用程序中，启用 `esModuleInterop` 则显得更为关键。所有主流打包工具和转译器都采用类似 `esModuleInterop` 的代码输出策略，因此 TypeScript 需要调整其类型检查以与之保持匹配。（编译器始终会推导 `tsc` 生成的 JavaScript 文件中将发生什么，因此即使使用其他工具代替 `tsc`，也仍应尽可能将影响代码输出的编译器选项配置为贴合该工具的输出结果。）

应避免在不启用 `esModuleInterop` 的情况下单独使用 `allowSyntheticDefaultImports`。它改变了编译器的类型检查行为，却未改变 `tsc` 生成的代码，可能导致输出不安全的 JavaScript 代码。此外，它所引入的检查调整仅仅是 `esModuleInterop` 所引入调整的不完整版本。即使不使用 `tsc` 生成代码，启用 `esModuleInterop` 也比单独启用 `allowSyntheticDefaultImports` 更好。

有人反对在启用 `esModuleInterop` 时 `tsc` 的 JavaScript 输出中包含 `__importDefault` 和 `__importStar` 辅助函数，原因要么是这略微增加了磁盘上的输出体积，要么是这些辅助函数所采用的互操作算法通过检查 `__esModule` 似乎歪曲了 Node.js 的互操作行为，从而导致前面讨论过的隐患。这两项反对意见都可以在不妥协接受禁用 `esModuleInterop` 带来的缺陷检查行为的前提下得到至少部分解决。首先，可以使用 `importHelpers` 编译器选项从 `tslib` 导入辅助函数，而不是将它们内联到每个需要它们的文件中。关于第二项反对意见，我们来看最后一个例子：

```ts
// @Filename: node_modules/transpiled-dependency/index.js
exports.__esModule = true
exports.default = function doSomething() {
  /* ... */
}
exports.something = 'something'

// @Filename: node_modules/true-cjs-dependency/index.js
module.exports = function doSomethingElse() {
  /* ... */
}

// @Filename: src/sayHello.ts
export default function sayHello() {
  /* ... */
}
export const hello = 'hello'

// @Filename: src/main.ts
import doSomething from 'transpiled-dependency'
import doSomethingElse from 'true-cjs-dependency'
import sayHello from './sayHello.js'
```

假设我们正在将 `src` 编译为 CommonJS 以供 Node.js 使用。在不开启 `allowSyntheticDefaultImports` 或 `esModuleInterop` 的情况下，从 `"true-cjs-dependency"` 导入 `doSomethingElse` 会报错，而其他导入则不会。要在不更改任何编译器选项的情况下修复该错误，可以将导入修改为 `import doSomethingElse = require("true-cjs-dependency")`。然而，取决于该模块的类型声明（未展示）是如何编写的，你可能也可以编写并调用一个命名空间导入，而这在语言层面属于违背规范的行为。如果启用了 `esModuleInterop`，展示的所有导入都不会报错（且均可调用），而无效的命名空间导入则会被捕获并报错。

如果我们决定在 Node.js 中将 `src` 迁移到原生 ESM（例如在根目录 package.json 中添加 `"type": "module"`），会发生什么变化呢？第一个导入（从 `"transpiled-dependency"` 导入的 `doSomething`）将不再能被调用——它出现了“双重 default”问题，我们必须调用 `doSomething.default()` 而非 `doSomething()`。（在 `--module node16`—`nodenext` 下，TypeScript 能够理解并捕获该问题。）但值得注意的是，**第二个**导入 `doSomethingElse`（在编译为 CommonJS 时需要 `esModuleInterop` 才能工作）在原生 ESM 中能够完全正常工作。

如果要对此有所微词，那绝不是因为 `esModuleInterop` 对第二个导入的处理。它所做的改变（既允许默认导入，又阻止可调用的命名空间导入）完全契合 Node.js 真实的 ESM/CJS 互操作策略，并使向真实 ESM 的迁移变得更加容易。真正的问题（如果有的话）在于 `esModuleInterop` 似乎没能为**第一个**导入提供一条无缝迁移路径。但这一问题并不是由启用 `esModuleInterop` 引入的；第一个导入完全未受其影响。不幸的是，在不破坏 `main.ts` 与 `sayHello.ts` 之间语义约定的前提下，这个问题是无法解决的，因为 `sayHello.ts` 的 CommonJS 输出在结构上与 `transpiled-dependency/index.js` 完全相同。如果 `esModuleInterop` 改变了转译后 `doSomething` 导入的工作方式，使其与在 Node.js ESM 中的工作方式一致，那么它也会以相同的方式改变 `sayHello` 导入的行为，从而导致输入的代码违反 ESM 语义（从而依然阻止 `src` 目录在不做修改的情况下迁移到 ESM）。

如我们所见，从转译模块到原生 ESM 不存在无缝迁移路径。但 `esModuleInterop` 是朝着正确方向迈出的一步。对于那些仍然希望最大限度减少模块语法转换和避免引入导入辅助函数的人来说，启用 `verbatimModuleSyntax` 是比禁用 `esModuleInterop` 更好的选择。`verbatimModuleSyntax` 强制在输出 CommonJS 的文件中使用 `import mod = require("mod")` 和 `export = ns` 语法，从而避免了我们讨论过的所有导入歧义，其代价是牺牲了向原生 ESM 迁移的便利性。

### 库代码需要特殊考量

作为 CommonJS 发布的库应避免使用默认导出，因为在不同工具和运行时之间，访问这些转译后导出项的方式各不相同，其中某些方式会让用户感到困惑。一个由 `tsc` 转译为 CommonJS 的默认导出，在 Node.js 中可以通过默认导入的 default 属性进行访问：

```js
import pkg from 'pkg'
pkg.default()
```

在大多数打包工具或转译 ESM 中则可以通过默认导入本身进行访问：

```js
import pkg from 'pkg'
pkg()
```

而在原生 CommonJS 中则可以通过 `require` 调用的 default 属性进行访问：

```js
const pkg = require('pkg')
pkg.default()
```

如果用户不得不去访问默认导入的 `.default` 属性，他们会察觉到模块配置有异样（misconfigured module smell）；而如果他们试图编写既能在 Node.js 中运行又能在打包工具中运行的代码，就可能会陷入困境。某些第三方 TypeScript 转译器提供了一些选项来改变默认导出的输出方式以缓解这种差异，但它们并不生成自己的声明（`.d.ts`）文件，这就导致了运行时行为与类型检查之间的脱节，进一步让用户感到困惑和沮丧。需要作为 CommonJS 发布的库不应使用默认导出，而应针对具有单一主要导出的模块使用 `export =`，针对具有多个导出的模块使用具名导出：

```diff
- export default function doSomething() { /* ... */ }
+ export = function doSomething() { /* ... */ }
```

（发布声明文件的）库还应格外注意，确保所编写的类型在各种编译器选项下均无报错。例如，有可能写出两个相互继承的接口，使得它们只有在禁用 `strictNullChecks` 时才能成功编译。如果库发布了这样的类型，就会迫使其所有使用者也必须禁用 `strictNullChecks`。`esModuleInterop` 可能会让类型声明包含类似的具有“传染性”的默认导入：

```ts
// @Filename: /node_modules/dependency/index.d.ts
import express from 'express'
declare function doSomething(req: express.Request): any
export = doSomething
```

假定该默认导入**仅在**启用 `esModuleInterop` 时才能工作，而在未开启该选项的用户引用该文件时会导致报错。用户**或许**确实应该开启 `esModuleInterop`，但库以这种方式强制用户的配置通常被认为是不妥的做法。更好的做法是库发布如下形式的声明文件：

```ts
import express = require('express')
// ...
```

类似这样的例子催生了一种传统观点，即库**不**应该启用 `esModuleInterop`。这个建议是一个合理的开端，但我们已经看到，在启用 `esModuleInterop` 时，命名空间导入的类型会发生变化，这可能会**引入**错误。因此，无论库在编译时是否开启 `esModuleInterop`，都有可能写出让自身选择具有传染性的语法。

希望精益求精以确保最大兼容性的库作者，应当针对不同的编译器选项矩阵来验证其声明文件。但是，使用 `verbatimModuleSyntax` 可以彻底避开 `esModuleInterop` 的问题，因为它强制要求输出 CommonJS 的文件使用 CommonJS 风格的导入和导出语法。此外，由于 `esModuleInterop` 只影响 CommonJS，随着未来越来越多的库转向仅发布 ESM，该问题的重要性也将逐渐降低。

<!--

https://github.com/babel/babel/issues/493
https://github.com/babel/babel/issues/95
https://github.com/nodejs/node/pull/16675
https://github.com/nodejs/ecmascript-modules/pull/31
https://github.com/google/traceur-compiler/pull/785#issuecomment-35633727
https://github.com/microsoft/TypeScript/pull/2460
https://github.com/systemjs/systemjs/commit/3b3b03a4b8ffc0f71fab263ef9d5c70f0adc5339
https://github.com/microsoft/TypeScript/pull/5577
https://github.com/microsoft/TypeScript/pull/19675
https://github.com/microsoft/TypeScript/issues/16093
https://github.com/nodejs/modules/issues/139
https://github.com/microsoft/TypeScript/issues/54212

-->
