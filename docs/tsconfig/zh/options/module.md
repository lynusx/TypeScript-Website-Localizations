---
display: 'Module'
oneline: '指定生成什么模块代码。'
---

设置程序的模块系统。有关更多信息，请参阅 [TypeScript `module` 选项背后的理论](/docs/handbook/modules/theory.html#the-module-output-format)和[其参考页面](/docs/handbook/modules/reference.html#the-module-compiler-option)。对于现代 Node.js 项目，你很可能需要 `"nodenext"`；而对于需要打包的代码，通常使用 `preserve` 或 `esnext`。

修改 `module` 会影响 [`moduleResolution`](#moduleResolution)，[后者同样有参考页面](/docs/handbook/modules/reference.html#the-moduleresolution-compiler-option)。

以下是该文件的一些示例输出：

```ts twoslash
// @filename: constants.ts
export const valueOfPi = 3.142
// ---cut---
// @filename: index.ts
import { valueOfPi } from './constants'

export const twoPi = valueOfPi * 2
```

#### `CommonJS`

```ts twoslash
// @showEmit
// @module: commonjs
// @noErrors
import { valueOfPi } from './constants'

export const twoPi = valueOfPi * 2
```

#### `UMD`

```ts twoslash
// @showEmit
// @module: umd
// @noErrors
import { valueOfPi } from './constants'

export const twoPi = valueOfPi * 2
```

#### `AMD`

```ts twoslash
// @showEmit
// @module: amd
// @noErrors
import { valueOfPi } from './constants'

export const twoPi = valueOfPi * 2
```

#### `System`

```ts twoslash
// @showEmit
// @module: system
// @noErrors
import { valueOfPi } from './constants'

export const twoPi = valueOfPi * 2
```

#### `ESNext`

```ts twoslash
// @showEmit
// @module: esnext
// @noErrors
import { valueOfPi } from './constants'

export const twoPi = valueOfPi * 2
```

#### `ES2015`/`ES6`/`ES2020`/`ES2022`

```ts twoslash
// @showEmit
// @module: es2015
// @noErrors
import { valueOfPi } from './constants'

export const twoPi = valueOfPi * 2
```

除了 `ES2015`/`ES6` 的基础功能外，`ES2020` 还添加了对[动态 `import`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import) 和 [`import.meta`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import.meta) 的支持；而 `ES2022` 进一步增加了对[顶层 `await`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await#top_level_await) 的支持。

#### `node16`/`node18`/`node20`/`nodenext`

`node16`、`node18`、`node20` 和 `nodenext` 模式集成了 Node 的[原生 ECMAScript 模块支持](https://nodejs.org/api/esm.html)。生成的 JavaScript 会根据文件扩展名以及距离最近的 `package.json` 中的 `type` 设置，采用 `CommonJS` 或 `ES2020` 输出。模块解析的方式也有所不同。你可以在[用户手册](/docs/handbook/esm-node.html)和[模块参考](/docs/handbook/modules/reference.html#node16-node18-node20-nodenext)中了解更多信息。

- `node16` 从 TypeScript 4.7 开始支持。
- `node18` 从 TypeScript 5.8 开始支持，作为 `node16` 的替代方案，并新增了对 import attributes 的支持。
- `node20` 新增了对 require(ESM) 的支持。
- `nodenext` 从 TypeScript 4.7 开始支持，但其行为会随着 Node.js 最新的稳定版本而变化。`--module nodenext` 会隐式启用浮动的 `--target esnext`。

#### `preserve`

在 `--module preserve`（于 TypeScript 5.4 [新增](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-4.html#support-for-require-calls-in---moduleresolution-bundler-and---module-preserve)）中，输入文件中编写的 ECMAScript import 和 export 会在输出中被原样保留，而 CommonJS 风格的 `import x = require("...")` 和 `export = ...` 语句则会被输出为 CommonJS 的 `require` 和 `module.exports`。换言之，每个单独的 import 或 export 语句的格式都会得到保留，而不会在整个编译过程（甚至整个文件）中被强制转换为统一的格式。

```ts twoslash
// @showEmit
// @module: preserve
// @noErrors
import { valueOfPi } from './constants'
import constants = require('./constants')

export const piSquared = valueOfPi * constants.valueOfPi
```

尽管在同一个文件中混用 import 和 require 调用的情况很少见，但这种 `module` 模式最能契合大多数现代打包工具以及 Bun 运行时的特性与能力。

> 为什么在使用打包工具或 Bun（此时很可能还设置了 `noEmit`）时还要关注 TypeScript 的 `module` 输出？TypeScript 的类型检查和模块解析行为会受到其“原本应该”生成的模块格式的影响。设置 `module` 能为 TypeScript 提供打包工具或运行时如何处理 import 和 export 的信息，从而确保导入值上展示的类型能够准确反映运行时或打包后的实际情况。

#### `None`

```ts twoslash
// @showEmit
// @module: none
// @noErrors
import { valueOfPi } from './constants'

export const twoPi = valueOfPi * 2
```
