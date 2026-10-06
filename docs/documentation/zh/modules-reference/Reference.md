---
title: 模块 - 参考
short: 参考
layout: docs
permalink: /zh/docs/handbook/modules/reference.html
oneline: 模块语法与编译器选项参考
translatable: true
---

## 模块语法

TypeScript 编译器在 TypeScript 和 JavaScript 文件中均能识别标准的 [ECMAScript 模块语法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)，并在 JavaScript 文件中支持多种形式的 [CommonJS 语法](https://www.typescriptlang.org/docs/handbook/type-checking-javascript-files.html#commonjs-modules-are-supported)。

此外，还有一些 TypeScript 专属的语法扩展，可用于 TypeScript 文件和/或 JSDoc 注释中。

### 导入和导出 TypeScript 专属声明

与任何标准的 JavaScript 声明一样，类型别名、接口、枚举和命名空间也可以使用 `export` 修饰符从模块中导出：

```ts
// Standard JavaScript syntax...
export function f() {}
// ...extended to type declarations
export type SomeType = /* ... */;
export interface SomeInterface { /* ... */ }
```

它们也可以在具名导出中被引用，甚至可以与标准 JavaScript 声明的引用并列：

```ts
export { f, SomeType, SomeInterface }
```

导出的类型（以及其他 TypeScript 专属声明）可以使用标准的 ECMAScript 导入语句导入：

```ts
import { f, SomeType, SomeInterface } from './module.js'
```

当使用命名空间导入或导出时，在类型位置引用命名空间时即可访问其导出的类型：

```ts
import * as mod from './module.js'
mod.f()
mod.SomeType // Property 'SomeType' does not exist on type 'typeof import("./module.js")'
let x: mod.SomeType // Ok
```

### 仅类型导入与导出

向 JavaScript 发射导入和导出语句时，TypeScript 默认会自动省略（不发射）仅在类型位置使用的导入以及仅引用类型的导出。仅类型导入与导出可用于强制执行此行为，并使省略行为显式化。使用 `import type` 编写的导入声明、使用 `export type { ... }` 编写的导出声明，以及带有 `type` 关键字前缀的导入或导出说明符，均保证会从输出的 JavaScript 中省略。

```ts
// @Filename: main.ts
import { f, type SomeInterface } from './module.js'
import type { SomeType } from './module.js'

class C implements SomeInterface {
  constructor(p: SomeType) {
    f()
  }
}

export type { C }

// @Filename: main.js
import { f } from './module.js'

class C {
  constructor(p) {
    f()
  }
}
```

即便值也可以使用 `import type` 导入，但由于它们不会存在于输出的 JavaScript 中，因此只能在非发射位置使用：

```ts
import type { f } from './module.js'
f() // 'f' cannot be used as a value because it was imported using 'import type'
let otherFunction: typeof f = () => {} // Ok
```

仅类型导入声明不能同时声明默认导入和具名绑定，因为这会导致无法明确 `type` 是仅修饰默认导入还是修饰整个导入声明。遇到这种情况，请将导入声明拆分为两条，或使用 `default` 作为具名绑定：

```ts
import type fs, { BigIntOptions } from "fs";
//          ^^^^^^^^^^^^^^^^^^^^^
// Error: A type-only import can specify a default import or named bindings, but not both.

import type { default as fs, BigIntOptions } from "fs"; // Ok
```

### `import()` 类型

TypeScript 提供了一种类似于 JavaScript 动态 `import` 的类型语法，用于在无需编写导入声明的情况下直接引用模块的类型：

```ts
// Access an exported type:
type WriteFileOptions = import('fs').WriteFileOptions
// Access the type of an exported value:
type WriteFileFunction = typeof import('fs').writeFile
```

这在 JavaScript 文件的 JSDoc 注释中尤为有用，因为在 JSDoc 中无法以其他方式导入类型：

```ts
/** @type {import("webpack").Configuration} */
module.exports = {
  // ...
}
```

### `export =` 与 `import = require()`

在发射 CommonJS 模块时，TypeScript 文件可以使用直接对应于 JavaScript 语法的 `module.exports = ...` 和 `const mod = require("...")`：

```ts
// @Filename: main.ts
import fs = require('fs')
export = fs.readFileSync('...')

// @Filename: main.js
;('use strict')
const fs = require('fs')
module.exports = fs.readFileSync('...')
```

之所以采用这种语法而非其 JavaScript 等价写法，是因为变量声明和属性赋值无法直接引用 TypeScript 类型，而专属的 TypeScript 语法可以做到：

```ts
// @Filename: a.ts
interface Options {
  /* ... */
}
module.exports = Options // Error: 'Options' only refers to a type, but is being used as a value here.
export = Options // Ok

// @Filename: b.ts
const Options = require('./a')
const options: Options = {/* ... */} // Error: 'Options' refers to a value, but is being used as a type here.

// @Filename: c.ts
import Options = require('./a')
const options: Options = {/* ... */} // Ok
```

### 环境模块

TypeScript 在脚本（非模块）文件中支持一种语法，用于声明在运行时存在但没有对应文件的模块。这些*环境模块*通常代表由运行时提供的模块，例如 Node.js 中的 `"fs"` 或 `"path"`：

```ts
declare module 'path' {
  export function normalize(p: string): string
  export function join(...paths: any[]): string
  export var sep: string
}
```

一旦环境模块被加载到 TypeScript 程序中，TypeScript 就能在其他文件中识别对该声明模块的导入：

```ts
// 👇 Ensure the ambient module is loaded -
//    may be unnecessary if path.d.ts is included
//    by the project tsconfig.json somehow.
/// <reference path="path.d.ts" />

import { normalize, join } from 'path'
```

环境模块声明很容易与[模块扩展](https://www.typescriptlang.org/docs/handbook/declaration-merging.html#module-augmentation)混淆，因为二者使用了完全相同的语法。当所在文件本身是一个模块（即包含顶层 `import` 或 `export` 语句，或受到 [`--moduleDetection force` 或 `auto`](https://www.typescriptlang.org/tsconfig#moduleDetection) 的影响）时，这种模块声明语法就会变为模块扩展：

```ts
// Not an ambient module declaration anymore!
export {}
declare module 'path' {
  export function normalize(p: string): string
  export function join(...paths: any[]): string
  export var sep: string
}
```

环境模块可以在模块声明体内使用导入来引用其他模块，而不会导致所在文件变为模块（否则会导致该环境模块声明变为模块扩展）：

```ts
declare module 'm' {
  // Moving this outside "m" would totally change the meaning of the file!
  import { SomeType } from 'other'
  export function f(): SomeType
}
```

*模式*环境模块在其名称中包含一个 `*` 通配符，用于匹配导入路径中的零个或多个字符。这在声明由自定义加载器提供的模块时非常有用：

```ts
declare module '*.html' {
  const content: string
  export default content
}
```

## `module` 编译器选项

本节将详细探讨 `module` 编译器选项的各个取值。关于该选项是什么以及它在整个编译流程中的定位，更多背景信息请参见[_模块输出格式_](/docs/handbook/modules/theory.html#the-module-output-format)理论章节。简而言之，`module` 编译器选项在历史上仅用于控制发射的 JavaScript 文件的输出模块格式。然而，较新的 `node16`、`node18` 和 `nodenext` 取值则描述了 Node.js 模块系统的全方位特性，包括支持哪些模块格式、每个文件的模块格式如何确定，以及不同模块格式之间如何相互操作。

### `node16`, `node18`, `node20`, `nodenext`

Node.js 同时支持 CommonJS 和 ECMAScript 模块，并对每个文件可以是哪种格式以及这两种格式之间如何互操作制定了具体规则。`node16`、`node18` 和 `nodenext` 描述了 Node.js 双格式模块系统的完整行为，并且**会将文件发射为 CommonJS 或 ESM 格式**。这与其他所有的 `module` 选项截然不同——其他选项与运行时无关，且会强制所有输出文件采用单一格式，需要由用户自行确保输出对其运行时有效。

> 一个常见的误解是：`node16`～`nodenext` 只会发射 ES 模块。实际上，这些模式描述的是*支持* ES 模块的 Node.js 版本，而不仅仅是*使用* ES 模块的项目。基于每个文件[检测到的模块格式](#module-format-detection)，ESM 和 CommonJS 发射均受支持。正因为它们是唯一能够反映 Node.js 双模块系统复杂性的 `module` 选项，所以对于所有旨在运行在 Node.js v12 或更高版本的应用程序和库（无论它们是否使用 ES 模块），它们都是**唯一正确的 `module` 选项**。

固定版本的 `node16` 和 `node18` 模式代表在其对应 Node.js 版本中趋于稳定的模块系统行为，而 `nodenext` 模式则随着 Node.js 最新的稳定版本持续更新。下表总结了这三种模式当前的差异：

|          | `target` | `moduleResolution` | import 断言 | import 属性 | JSON 导入          | require(esm) |
| -------- | -------- | ------------------ | ----------- | ----------- | ------------------ | ------------ |
| node16   | `es2022` | `node16`           | ❌          | ❌          | 无限制             | ❌           |
| node18   | `es2022` | `node16`           | ✅          | ✅          | 需要 `type "json"` | ❌           |
| nodenext | `esnext` | `nodenext`         | ❌          | ✅          | 需要 `type "json"` | ✅           |

#### 模块格式检测

- `.mts`/`.mjs`/`.d.mts` 文件始终是 ES 模块。
- `.cts`/`.cjs`/`.d.cts` 文件始终是 CommonJS 模块。
- 若最近的祖先 package.json 文件包含 `"type": "module"`，则 `.ts`/`.tsx`/`.js`/`.jsx`/`.d.ts` 文件为 ES 模块，否则为 CommonJS 模块。

输入的 `.ts`/`.tsx`/`.mts`/`.cts` 文件检测到的模块格式决定了发射的 JavaScript 文件的模块格式。例如，一个完全由 `.ts` 文件组成的项目在 `--module nodenext` 下默认会全部发射为 CommonJS 模块；而只需在项目 package.json 中添加 `"type": "module"`，即可使其全部发射为 ES 模块。

#### 互操作性规则

- **当 ES 模块引用 CommonJS 模块时：**
  - CommonJS 模块的 `module.exports` 可以作为默认导入供 ES 模块使用。
  - CommonJS 模块 `module.exports` 上的属性（除 `default` 外）可能可以、也可能无法作为具名导入供 ES 模块使用。Node.js 会尝试通过[静态分析](https://github.com/nodejs/cjs-module-lexer)来使它们可用。TypeScript 无法从声明文件中得知该静态分析是否会成功，因此会乐观地假设其能够成功。这限制了 TypeScript 捕获可能在运行时崩溃的具名导入的能力。更多详情请参见 [#54018](https://github.com/microsoft/TypeScript/issues/54018)。
- **当 CommonJS 模块引用 ES 模块时：**
  - 在 `node16` 和 `node18` 中，`require` 不能引用 ES 模块。对 TypeScript 而言，这也包括在[检测](#module-format-detection)为 CommonJS 模块的文件中的 `import` 语句，因为这些 `import` 语句在发射的 JavaScript 中会被转换为 `require` 调用。
  - 在 `nodenext` 中，为了反映 Node.js v22.12.0 及更高版本的行为，`require` 可以引用 ES 模块。在 Node.js 中，如果该 ES 模块或其导入的任何模块使用了顶层 `await`，则会抛出错误。TypeScript 不会尝试检测这种情况，也不会发出编译期错误。`require` 调用的结果是该模块的模块命名空间对象（Module Namespace Object），即与对同一模块执行 `await import()` 的结果相同（但无需 `await` 任何内容）。
  - 动态 `import()` 调用始终可用于导入 ES 模块。它返回一个解析为该模块的模块命名空间对象的 Promise（与在另一个 ES 模块中通过 `import * as ns from "./module.js"` 获取的对象相同）。

#### 发射（Emit）

每个文件的发射格式由其[检测到的模块格式](#module-format-detection)决定。ESM 发射与 [`--module esnext`](#es2015-es2020-es2022-esnext) 类似，但针对 `import x = require("...")` 进行了特殊转换（这在 `--module esnext` 中是不允许的）：

```ts
// @Filename: main.ts
import x = require('mod')
```

```js
// @Filename: main.js
import { createRequire as _createRequire } from 'module'
const __require = _createRequire(import.meta.url)
const x = __require('mod')
```

CommonJS 发射与 [`--module commonjs`](#commonjs) 类似，但不会转换动态 `import()` 调用。此处展示的发射代码启用了 `esModuleInterop`：

```ts
// @Filename: main.ts
import fs from 'fs' // transformed
const dynamic = import('mod') // not transformed
```

```js
// @Filename: main.js
'use strict'
var __importDefault =
  (this && this.__importDefault) ||
  function (mod) {
    return mod && mod.__esModule ? mod : { default: mod }
  }
Object.defineProperty(exports, '__esModule', { value: true })
const fs_1 = __importDefault(require('fs')) // transformed
const dynamic = import('mod') // not transformed
```

#### 隐式与强制选项

- `--module nodenext` 会隐式启用并强制使用 `--moduleResolution nodenext`。
- `--module node18` 或 `node16` 会隐式启用并强制使用 `--moduleResolution node16`。
- `--module nodenext` 隐式启用 `--target esnext`。
- `--module node18` 或 `node16` 隐式启用 `--target es2022`。
- `--module nodenext`、`node18` 或 `node16` 隐式启用 `--esModuleInterop`。

#### 小结

- 对于所有旨在运行于 Node.js v12 或更高版本的应用程序和库（无论它们是否使用 ES 模块），`node16`、`node18` 和 `nodenext` 都是唯一正确的 `module` 选项。
- `node16`、`node18` 和 `nodenext` 会基于每个文件[检测到的模块格式](#module-format-detection)，将文件发射为 CommonJS 或 ESM 格式。
- Node.js 在 ESM 与 CJS 之间的互操作规则会反映在类型检查中。
- ESM 发射会将 `import x = require("...")` 转换为通过导入的 `createRequire` 构造的 `require` 调用。
- CommonJS 发射会保留动态 `import()` 调用不予转换，因此 CommonJS 模块可以异步导入 ES 模块。

### `preserve`

在 `--module preserve`（在 TypeScript 5.4 中[引入](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-4.html#support-for-require-calls-in---moduleresolution-bundler-and---module-preserve)）下，输入文件中编写的 ECMAScript 导入和导出在输出中得以保留，而 CommonJS 风格的 `import x = require("...")` 和 `export = ...` 语句则发射为 CommonJS 的 `require` 和 `module.exports`。换句话说，每条单独的导入或导出语句的格式都被原样保留，而不是在整个编译（甚至整个文件）中被强行统一为单一格式。

虽然在同一个文件中混用导入和 require 调用的需求并不常见，但这种 `module` 模式最能契合大多数现代打包工具以及 Bun 运行时的能力。

> 既然在使用打包工具或 Bun 时通常也会设置 `noEmit`，那为什么还要关心 TypeScript 的 `module` 发射呢？这是因为 TypeScript 的类型检查和模块解析行为会受到它*假设*要发射的模块格式的影响。设置 `module` 可以向 TypeScript 提供有关你的打包工具或运行时将如何处理导入和导出的信息，从而确保你在导入值上看到的类型能够准确反映运行时或打包后的实际情况。更多讨论请参见 [`--moduleResolution bundler`](#bundler)。

#### 示例

```ts
// @Filename: main.ts
import x, { y, z } from 'mod'
import mod = require('mod')
const dynamic = import('mod')

export const e1 = 0
export default 'default export'
```

```js
// @Filename: main.js
import x, { y, z } from 'mod'
const mod = require('mod')
const dynamic = import('mod')

export const e1 = 0
export default 'default export'
```

#### 隐式与强制选项

- `--module preserve` 隐式启用 `--moduleResolution bundler`。
- `--module preserve` 隐式启用 `--esModuleInterop`。

> 在 `--module preserve` 中，`--esModuleInterop` 选项默认启用仅是为了其[类型检查](https://www.typescriptlang.org/docs/handbook/modules/appendices/esm-cjs-interop.html#allowsyntheticdefaultimports-and-esmoduleinterop)行为。由于在 `--module preserve` 中导入永远不会转换为 require 调用，因此 `--esModuleInterop` 不会影响发射的 JavaScript。

### `es2015`, `es2020`, `es2022`, `esnext`

#### 小结

- 对于打包工具、Bun 和 tsx，使用 `esnext` 搭配 `--moduleResolution bundler`。
- 请勿用于 Node.js。若要为 Node.js 发射 ES 模块，请使用 `node16`、`node18` 或 `nodenext` 并在 package.json 中设置 `"type": "module"`。
- 在非声明文件中不允许使用 `import mod = require("mod")`。
- `es2020` 增加了对 `import.meta` 属性的支持。
- `es2022` 增加了对顶层 `await` 的支持。
- `esnext` 是一个动态演进的目标，可能包含对 ECMAScript 模块 Stage 3 提案的支持。
- 发射的文件为 ES 模块，但依赖项可以是任何格式。

#### 示例

```ts
// @Filename: main.ts
import x, { y, z } from 'mod'
import * as mod from 'mod'
const dynamic = import('mod')
console.log(x, y, z, mod, dynamic)

export const e1 = 0
export default 'default export'
```

```js
// @Filename: main.js
import x, { y, z } from 'mod'
import * as mod from 'mod'
const dynamic = import('mod')
console.log(x, y, z, mod, dynamic)

export const e1 = 0
export default 'default export'
```

### `commonjs`

#### 小结

- 你大概率不应使用此选项。若要为 Node.js 发射 CommonJS 模块，请使用 `node16`、`node18` 或 `nodenext`。
- 发射的文件为 CommonJS 模块，但依赖项可以是任何格式。
- 动态 `import()` 会被转换为一个包装了 `require()` 调用的 Promise。
- `esModuleInterop` 会影响默认导入和命名空间导入的输出代码。

#### 示例

> 输出展示基于 `esModuleInterop: false`。

```ts
// @Filename: main.ts
import x, { y, z } from 'mod'
import * as mod from 'mod'
const dynamic = import('mod')
console.log(x, y, z, mod, dynamic)

export const e1 = 0
export default 'default export'
```

```js
// @Filename: main.js
'use strict'
Object.defineProperty(exports, '__esModule', { value: true })
exports.e1 = void 0
const mod_1 = require('mod')
const mod = require('mod')
const dynamic = Promise.resolve().then(() => require('mod'))

console.log(mod_1.default, mod_1.y, mod_1.z, mod)
exports.e1 = 0
exports.default = 'default export'
```

```ts
// @Filename: main.ts
import mod = require('mod')
console.log(mod)

export = {
  p1: true,
  p2: false,
}
```

```js
// @Filename: main.js
'use strict'
const mod = require('mod')
console.log(mod)

module.exports = {
  p1: true,
  p2: false,
}
```

### `system`

#### 小结

- 专为与 [SystemJS 模块加载器](https://github.com/systemjs/systemjs)搭配使用而设计。

#### 示例

```ts
// @Filename: main.ts
import x, { y, z } from 'mod'
import * as mod from 'mod'
const dynamic = import('mod')
console.log(x, y, z, mod, dynamic)

export const e1 = 0
export default 'default export'
```

```js
// @Filename: main.js
System.register(['mod'], function (exports_1, context_1) {
  'use strict'
  var mod_1, mod, dynamic, e1
  var __moduleName = context_1 && context_1.id
  return {
    setters: [
      function (mod_1_1) {
        mod_1 = mod_1_1
        mod = mod_1_1
      },
    ],
    execute: function () {
      dynamic = context_1.import('mod')
      console.log(mod_1.default, mod_1.y, mod_1.z, mod, dynamic)
      exports_1('e1', (e1 = 0))
      exports_1('default', 'default export')
    },
  }
})
```

### `amd`

#### 小结

- 专为诸如 RequireJS 等 AMD 加载器设计。
- 你大概率不应使用此选项。请改用打包工具。
- 发射的文件为 AMD 模块，但依赖项可以是任何格式。
- 支持 `outFile`。

#### 示例

```ts
// @Filename: main.ts
import x, { y, z } from 'mod'
import * as mod from 'mod'
const dynamic = import('mod')
console.log(x, y, z, mod, dynamic)

export const e1 = 0
export default 'default export'
```

```js
// @Filename: main.js
define(['require', 'exports', 'mod', 'mod'], function (
  require,
  exports,
  mod_1,
  mod,
) {
  'use strict'
  Object.defineProperty(exports, '__esModule', { value: true })
  exports.e1 = void 0
  const dynamic = new Promise((resolve_1, reject_1) => {
    require(['mod'], resolve_1, reject_1)
  })

  console.log(mod_1.default, mod_1.y, mod_1.z, mod, dynamic)
  exports.e1 = 0
  exports.default = 'default export'
})
```

### `umd`

#### 小结

- 专为 AMD 或 CommonJS 加载器设计。
- 不像大多数其他 UMD 包装器那样暴露全局变量。
- 你大概率不应使用此选项。请改用打包工具。
- 发射的文件为 UMD 模块，但依赖项可以是任何格式。

#### 示例

```ts
// @Filename: main.ts
import x, { y, z } from 'mod'
import * as mod from 'mod'
const dynamic = import('mod')
console.log(x, y, z, mod, dynamic)

export const e1 = 0
export default 'default export'
```

```js
// @Filename: main.js
;(function (factory) {
  if (typeof module === 'object' && typeof module.exports === 'object') {
    var v = factory(require, exports)
    if (v !== undefined) module.exports = v
  } else if (typeof define === 'function' && define.amd) {
    define(['require', 'exports', 'mod', 'mod'], factory)
  }
})(function (require, exports) {
  'use strict'
  var __syncRequire =
    typeof module === 'object' && typeof module.exports === 'object'
  Object.defineProperty(exports, '__esModule', { value: true })
  exports.e1 = void 0
  const mod_1 = require('mod')
  const mod = require('mod')
  const dynamic = __syncRequire
    ? Promise.resolve().then(() => require('mod'))
    : new Promise((resolve_1, reject_1) => {
        require(['mod'], resolve_1, reject_1)
      })

  console.log(mod_1.default, mod_1.y, mod_1.z, mod, dynamic)
  exports.e1 = 0
  exports.default = 'default export'
})
```

## `moduleResolution` 编译器选项

本节介绍多个 `moduleResolution` 模式共有的模块解析特性与流程，随后详细说明每个模式的具体细节。关于该选项是什么以及它在整个编译流程中的定位，更多背景信息请参见[_模块解析_](/docs/handbook/modules/theory.html#module-resolution)理论章节。简而言之，`moduleResolution` 控制 TypeScript 如何将*模块标识符*（`import`/`export`/`require` 语句中的字符串字面量）解析为磁盘上的文件，并且应当将其配置为与目标运行时或打包工具所使用的模块解析器相匹配。

### 通用特性与处理流程

#### 文件扩展名替换

TypeScript 始终希望在内部解析到一个能够提供类型信息的文件，同时确保运行时或打包工具可以使用相同的路径解析到提供 JavaScript 实现的文件。对于任何根据所指定的 `moduleResolution` 算法会在运行时或打包工具中触发查找 JavaScript 文件的模块标识符，TypeScript 都会首先尝试查找具有相同名称和相应文件扩展名的 TypeScript 实现文件或类型声明文件。

| 运行时查找 | TypeScript 查找 #1 | TypeScript 查找 #2 | TypeScript 查找 #3 | TypeScript 查找 #4 | TypeScript 查找 #5 |
| ---------- | ------------------ | ------------------ | ------------------ | ------------------ | ------------------ |
| `/mod.js`  | `/mod.ts`          | `/mod.tsx`         | `/mod.d.ts`        | `/mod.js`          | `./mod.jsx`        |
| `/mod.mjs` | `/mod.mts`         | `/mod.d.mts`       | `/mod.mjs`         |                    |                    |
| `/mod.cjs` | `/mod.cts`         | `/mod.d.cts`       | `/mod.cjs`         |                    |                    |

请注意，此行为与 import 中实际书写的模块标识符无关。这意味着即使模块标识符显式使用了 `.js` 文件扩展名，TypeScript 也能解析到 `.ts` 或 `.d.ts` 文件：

```ts
import x from './mod.js'
// Runtime lookup: "./mod.js"
// TypeScript lookup #1: "./mod.ts"
// TypeScript lookup #2: "./mod.d.ts"
// TypeScript lookup #3: "./mod.js"
```

关于 TypeScript 模块解析为何采用这种机制的解释，请参见[_TypeScript 模拟宿主的模块解析，但附加类型_](/docs/handbook/modules/theory.html#typescript-imitates-the-hosts-module-resolution-but-with-types)。

#### 相对文件路径解析

TypeScript 的所有 `moduleResolution` 算法都支持通过包含文件扩展名的相对路径来引用模块（该扩展名将根据[上述规则](#file-extension-substitution)进行替换）：

```ts
// @Filename: a.ts
export {}

// @Filename: b.ts
import {} from './a.js' // ✅ Works in every `moduleResolution`
```

#### 无扩展名相对路径

在某些情况下，运行时或打包工具允许在相对路径中省略 `.js` 文件扩展名。只要 `moduleResolution` 设置和上下文表明运行时或打包工具支持此行为，TypeScript 就会支持该用法：

```ts
// @Filename: a.ts
export {}

// @Filename: b.ts
import {} from './a'
```

如果 TypeScript 确定针对模块标识符 `"./a"` 运行时将执行对 `./a.js` 的查找，那么 `./a.js` 将经历[扩展名替换](#file-extension-substitution)，并在本例中解析为 `a.ts` 文件。

在 Node.js 中，`import` 路径不支持无扩展名相对路径，且在 package.json 文件中指定的文件路径也不一定始终支持。尽管某些运行时和打包工具支持省略 `.mjs`/`.mts` 或 `.cjs`/`.cts` 文件扩展名，但 TypeScript 目前从不支持省略这些扩展名。

#### 目录模块（index 文件解析）

在某些情况下，可以引用目录而非文件作为模块。在最简单且最常见的情况下，这涉及运行时或打包工具在目录中查找 `index.js` 文件。只要 `moduleResolution` 设置和上下文表明运行时或打包工具支持此行为，TypeScript 就会支持该用法：

```ts
// @Filename: dir/index.ts
export {}

// @Filename: b.ts
import {} from './dir'
```

如果 TypeScript 确定针对模块标识符 `"./dir"` 运行时将执行对 `./dir/index.js` 的查找，那么 `./dir/index.js` 将经历[扩展名替换](#file-extension-substitution)，并在本例中解析为 `dir/index.ts` 文件。

目录模块还可以包含一个 package.json 文件，其中支持对 [`"main"` 和 `"types"`](#packagejson-main-and-types) 字段的解析，且其优先级高于 `index.js` 查找。目录模块中也支持 [`"typesVersions"`](#packagejson-typesversions) 字段。

请注意，目录模块与 [`node_modules` 包](#node_modules-package-lookups)并不相同，它仅支持包可用特性的一个子集，且在某些上下文中完全不受支持。Node.js 将其视为一项[遗留特性](https://nodejs.org/dist/latest-v20.x/docs/api/modules.html#folders-as-modules)。

#### `paths`

##### 概述

TypeScript 提供了 `paths` 编译器选项，用于覆盖编译器针对裸标识符的模块解析。尽管该特性最初是为与 AMD 模块加载器搭配使用而设计的（在 ESM 出现或打包工具广泛使用之前在浏览器中运行模块的一种手段），但当运行时或打包工具支持 TypeScript 未建模的模块解析特性时，它在今天依然有其用途。例如，当使用 `--experimental-network-imports` 运行 Node.js 时，你可以为特定的 `https://` 导入手动指定本地类型定义文件：

```json
{
  "compilerOptions": {
    "module": "nodenext",
    "paths": {
      "https://esm.sh/lodash@4.17.21": [
        "./node_modules/@types/lodash/index.d.ts"
      ]
    }
  }
}
```

```ts
// Typed by ./node_modules/@types/lodash/index.d.ts due to `paths` entry
import { add } from 'https://esm.sh/lodash@4.17.21'
```

使用打包工具构建的应用程序通常也会在打包工具配置中定义便捷的路径别名，然后通过 `paths` 将这些别名告知 TypeScript：

```json
{
  "compilerOptions": {
    "module": "esnext",
    "moduleResolution": "bundler",
    "paths": {
      "@app/*": ["./src/*"]
    }
  }
}
```

##### `paths` 不影响发射

`paths` 选项*不会*改变 TypeScript 发射的代码中的导入路径。因此，非常容易创建出在 TypeScript 中看似有效但在运行时会崩溃的路径别名：

```json
{
  "compilerOptions": {
    "module": "nodenext",
    "paths": {
      "node-has-no-idea-what-this-is": ["./oops.ts"]
    }
  }
}
```

```ts
// TypeScript: ✅
// Node.js: 💥
import {} from 'node-has-no-idea-what-this-is'
```

虽然被打包的应用程序配置 `paths` 并没有问题，但发布的库*绝不应该*这样做，因为如果不要求库的使用者同时为 TypeScript 和其打包工具配置完全相同的别名，发射的 JavaScript 就无法正常工作。无论是库还是应用程序，都可以考虑使用 [package.json `"imports"`](#packagejson-imports-and-self-name-imports) 作为便捷 `paths` 别名的标准替代方案。

##### `paths` 不应指向 monorepo 包或 node_modules 包

尽管匹配 `paths` 别名的模块标识符是裸标识符，但一旦别名解析完毕，模块解析就会将解析后的路径作为相对路径继续进行。因此，针对 [`node_modules` 包查找](#node_modules-package-lookups)生效的解析特性（包括 package.json `"exports"` 字段支持）在匹配 `paths` 别名时都不会生效。如果使用 `paths` 指向 `node_modules` 包，可能会导致意料之外的行为：

```ts
{
  "compilerOptions": {
    "paths": {
      "pkg": ["./node_modules/pkg/dist/index.d.ts"],
      "pkg/*": ["./node_modules/pkg/*"]
    }
  }
}
```

尽管该配置可以模拟包解析的部分行为，但它会覆盖该包的 `package.json` 文件所定义的任何 `main`、`types`、`exports` 和 `typesVersions`，并且从该包导入的内容在运行时可能会失败。

同样的注意事项也适用于 monorepo 中相互引用的包。与其使用 `paths` 让 TypeScript 人为地将 `"@my-scope/lib"` 解析到同级包，不如通过 [npm](https://docs.npmjs.com/cli/v7/using-npm/workspaces)、[yarn](https://classic.yarnpkg.com/en/docs/workspaces/) 或 [pnpm](https://pnpm.io/workspaces) 使用工作区（workspaces）将你的包符号链接到 `node_modules` 中，这样 TypeScript 以及运行时或打包工具都能执行真正的 `node_modules` 包查找。如果要将 monorepo 中的包发布到 npm，这一点尤为重要——因为在被用户安装后，这些包将通过 `node_modules` 包查找相互引用，而使用工作区可以让你在本地开发期间就能对该行为进行测试。

##### 与 `baseUrl` 的关系

当提供了 [`baseUrl`](#baseurl) 时，每个 `paths` 数组中的值都会相对于 `baseUrl` 解析。否则，它们将相对于定义它们的 `tsconfig.json` 文件进行解析。

##### 通配符替换

`paths` 模式可以包含单个 `*` 通配符，用于匹配任意字符串。随后可以在文件路径值中使用 `*` 标记来替换匹配到的字符串：

```json
{
  "compilerOptions": {
    "paths": {
      "@app/*": ["./src/*"]
    }
  }
}
```

解析 `"@app/components/Button"` 的导入时，TypeScript 将匹配 `@app/*`，将 `*` 绑定到 `components/Button`，然后尝试相对于 `tsconfig.json` 路径解析 `./src/components/Button` 路径。该查找的其余部分将根据 `moduleResolution` 设置遵循与任何其他[相对路径查找](#relative-file-path-resolution)相同的规则。

当多个模式与同一个模块标识符匹配时，将使用在任何 `*` 标记之前具有最长匹配前缀的模式：

```json
{
  "compilerOptions": {
    "paths": {
      "*": ["./src/foo/one.ts"],
      "foo/*": ["./src/foo/two.ts"],
      "foo/bar": ["./src/foo/three.ts"]
    }
  }
}
```

在解析 `"foo/bar"` 的导入时，所有三个 `paths` 模式均能匹配，但最终会使用最后一个，因为 `"foo/bar"` 比 `"foo/"` 和 `""` 更长。

##### 回退（Fallbacks）

可以为一个路径映射提供多个文件路径。如果某个路径解析失败，则会依次尝试数组中的下一个路径，直到解析成功或到达数组末尾。

```json
{
  "compilerOptions": {
    "paths": {
      "*": ["./vendor/*", "./types/*"]
    }
  }
}
```

#### `baseUrl`

> `baseUrl` 最初是为与 AMD 模块加载器搭配使用而设计的。如果你没有使用 AMD 模块加载器，你大概率不应使用 `baseUrl`。自 TypeScript 4.1 起，使用 [`paths`](#paths) 已不再必须配置 `baseUrl`，因此不应仅仅为了设置 `paths` 值的基准解析目录而使用它。

`baseUrl` 编译器选项可以与任何 `moduleResolution` 模式结合使用，它指定了解析裸标识符（不以 `./`、`../` 或 `/` 开头的模块标识符）的基准目录。在支持 [`node_modules` 包查找](#node_modules-package-lookups)的 `moduleResolution` 模式中，`baseUrl` 的优先级高于该查找。

在执行 `baseUrl` 查找时，解析过程遵循与其他相对路径解析相同的规则。例如，在支持[无扩展名相对路径](#extensionless-relative-paths)的 `moduleResolution` 模式下，如果 `baseUrl` 设置为 `/src`，模块标识符 `"some-file"` 可能会解析为 `/src/some-file.ts`。

相对模块标识符的解析绝不受 `baseUrl` 选项的影响。

#### `node_modules` 包查找

Node.js 将不是相对路径、绝对路径或 URL 的模块标识符视为对包的引用，并在 `node_modules` 子目录中查找它们。打包工具很自然地沿用了这一行为，从而让其用户能够使用与 Node.js 相同的依赖管理机制，甚至往往使用完全相同的依赖。除 `classic` 外，TypeScript 的所有 `moduleResolution` 选项均支持 `node_modules` 查找。（在其他解析手段均告失败时，`classic` 支持在 `node_modules/@types` 中查找，但从不直接在 `node_modules` 中查找包。）每一次 `node_modules` 包查找都具有以下结构（在诸如 `paths`、`baseUrl`、自名称导入以及 package.json `"imports"` 等更高优先级的裸标识符规则用尽后开始）：

1. 对于导入文件的每个祖先目录，如果其内部存在 `node_modules` 目录：
   1. 如果在 `node_modules` 内存在与包名同名的目录：
      1. 尝试从该包目录解析类型。
      2. 如果找到结果，返回该结果并终止搜索。
   2. 如果在 `node_modules/@types` 内存在与包名同名的目录：
      1. 尝试从该 `@types` 包目录解析类型。
      2. 如果找到结果，返回该结果并终止搜索。
2. 再次遍历所有 `node_modules` 目录重复上述搜索，但这一次允许返回 JavaScript 文件作为结果，且不再在 `@types` 目录中搜索。

除 `classic` 外的所有 `moduleResolution` 模式均遵循此模式，而一旦定位到包目录后具体如何从中解析的细节则有所不同，下文将对此进行说明。

#### package.json `"exports"`

当 `moduleResolution` 设置为 `node16`、`nodenext` 或 `bundler` 且未禁用 `resolvePackageJsonExports` 时，在由[裸标识符 `node_modules` 包查找](#node_modules-package-lookups)触发并从包目录进行解析时，TypeScript 遵循 Node.js 的 [package.json `"exports"` 规范](https://nodejs.org/api/packages.html#packages_package_entry_points)。

TypeScript 通过 `"exports"` 将模块标识符解析为文件路径的实现与 Node.js 完全一致。然而，一旦文件路径解析完成，TypeScript 仍会[尝试多个文件扩展名](#file-extension-substitution)，以便优先找到类型。

通过[条件导出](https://nodejs.org/api/packages.html#conditional-exports)解析时，如果存在 `"types"` 和 `"default"` 条件，TypeScript 始终会匹配它们。此外，TypeScript 还会根据 [`"typesVersions"`](#packagejson-typesversions) 中实现的相同版本匹配规则，匹配形如 `"types@{selector}"`（其中 `{selector}` 是兼容 `"typesVersions"` 的版本选择器）的带版本类型条件。其他不可配置的条件取决于 `moduleResolution` 模式，并在后文中说明。额外的匹配条件可以通过 `customConditions` 编译器选项进行配置。

请注意，只要存在 `"exports"`，就会阻止解析任何未在 `"exports"` 中显式列出或未被模式匹配的子路径。

##### 示例：子路径、条件与扩展名替换

场景：在具有以下 package.json 的包目录中，请求 `"pkg/subpath"` 并附带条件 `["types", "node", "require"]`（由 `moduleResolution` 设置以及触发模块解析请求的上下文决定）：

```json
{
  "name": "pkg",
  "exports": {
    ".": {
      "import": "./index.mjs",
      "require": "./index.cjs"
    },
    "./subpath": {
      "import": "./subpath/index.mjs",
      "require": "./subpath/index.cjs"
    }
  }
}
```

包目录内的解析流程：

1. `"exports"` 是否存在？**是。**
2. `"exports"` 是否包含 `"./subpath"` 条目？**是。**
3. `exports["./subpath"]` 的值是一个对象——它必定是在指定条件。
4. 第一个条件 `"import"` 是否匹配此请求？**否。**
5. 第二个条件 `"require"` 是否匹配此请求？**是。**
6. 路径 `"./subpath/index.cjs"` 是否具有可识别的 TypeScript 文件扩展名？**否，因此使用扩展名替换。**
7. 通过[扩展名替换](#file-extension-substitution)，尝试以下路径，返回第一个存在的文件，否则返回 `undefined`：
   1. `./subpath/index.cts`
   2. `./subpath/index.d.cts`
   3. `./subpath/index.cjs`

如果 `./subpath/index.cts` 或 `./subpath.d.cts` 存在，则解析完成。否则，解析将根据 [`node_modules` 包查找](#node_modules-package-lookups)规则搜索 `node_modules/@types/pkg` 和其他 `node_modules` 目录以尝试解析类型。如果未找到任何类型，对所有 `node_modules` 进行的第二轮搜索将解析为 `./subpath/index.cjs`（假设它存在），这算作解析成功，但由于其不提供类型，因此会导致导入被推断为 `any` 类型，并在启用时触发 `noImplicitAny` 错误。

##### 示例：显式 `"types"` 条件

场景：在具有以下 package.json 的包目录中，请求 `"pkg/subpath"` 并附带条件 `["types", "node", "import"]`（由 `moduleResolution` 设置以及触发模块解析请求的上下文决定）：

```json
{
  "name": "pkg",
  "exports": {
    "./subpath": {
      "import": {
        "types": "./types/subpath/index.d.mts",
        "default": "./es/subpath/index.mjs"
      },
      "require": {
        "types": "./types/subpath/index.d.cts",
        "default": "./cjs/subpath/index.cjs"
      }
    }
  }
}
```

包目录内的解析流程：

1. `"exports"` 是否存在？**是。**
2. `"exports"` 是否包含 `"./subpath"` 条目？**是。**
3. `exports["./subpath"]` 的值是一个对象——它必定是在指定条件。
4. 第一个条件 `"import"` 是否匹配此请求？**是。**
5. `exports["./subpath"].import` 的值是一个对象——它必定是在指定条件。
6. 第一个条件 `"types"` 是否匹配此请求？**是。**
7. 路径 `"./types/subpath/index.d.mts"` 是否具有可识别的 TypeScript 文件扩展名？**是，因此不使用扩展名替换。**
8. 若该文件存在则返回路径 `"./types/subpath/index.d.mts"`，否则返回 `undefined`。

##### 示例：带版本的 `"types"` 条件

场景：使用 TypeScript 4.7.5，在具有以下 package.json 的包目录中请求 `"pkg/subpath"` 并附带条件 `["types", "node", "import"]`（由 `moduleResolution` 设置以及触发模块解析请求的上下文决定）：

```json
{
  "name": "pkg",
  "exports": {
    "./subpath": {
      "types@>=5.2": "./ts5.2/subpath/index.d.ts",
      "types@>=4.6": "./ts4.6/subpath/index.d.ts",
      "types": "./tsold/subpath/index.d.ts",
      "default": "./dist/subpath/index.js"
    }
  }
}
```

包目录内的解析流程：

1. `"exports"` 是否存在？**是。**
2. `"exports"` 是否包含 `"./subpath"` 条目？**是。**
3. `exports["./subpath"]` 的值是一个对象——它必定是在指定条件。
4. 第一个条件 `"types@>=5.2"` 是否匹配此请求？**否，4.7.5 不大于或等于 5.2。**
5. 第二个条件 `"types@>=4.6"` 是否匹配此请求？**是，4.7.5 大于或等于 4.6。**
6. 路径 `"./ts4.6/subpath/index.d.ts"` 是否具有可识别的 TypeScript 文件扩展名？**是，因此不使用扩展名替换。**
7. 若该文件存在则返回路径 `"./ts4.6/subpath/index.d.ts"`，否则返回 `undefined`。

##### 示例：子路径模式

场景：在具有以下 package.json 的包目录中，请求 `"pkg/wildcard.js"` 并附带条件 `["types", "node", "import"]`（由 `moduleResolution` 设置以及触发模块解析请求的上下文决定）：

```json
{
  "name": "pkg",
  "type": "module",
  "exports": {
    "./*.js": {
      "types": "./types/*.d.ts",
      "default": "./dist/*.js"
    }
  }
}
```

包目录内的解析流程：

1. `"exports"` 是否存在？**是。**
2. `"exports"` 是否包含 `"./wildcard.js"` 条目？**否。**
3. 是否存在包含 `*` 的键能够匹配 `"./wildcard.js"`？**是，`"./*.js"` 匹配成功，并将替换内容设为 `wildcard`。**
4. `exports["./*.js"]` 的值是一个对象——它必定是在指定条件。
5. 第一个条件 `"types"` 是否匹配此请求？**是。**
6. 在 `./types/*.d.ts` 中，将 `*` 替换为匹配内容 `wildcard`：**`./types/wildcard.d.ts`**
7. 路径 `"./types/wildcard.d.ts"` 是否具有可识别的 TypeScript 文件扩展名？**是，因此不使用扩展名替换。**
8. 若该文件存在则返回路径 `"./types/wildcard.d.ts"`，否则返回 `undefined`。

##### 示例：`"exports"` 阻止访问其他子路径

场景：在具有以下 package.json 的包目录中请求 `"pkg/dist/index.js"`：

```json
{
  "name": "pkg",
  "main": "./dist/index.js",
  "exports": "./dist/index.js"
}
```

包目录内的解析流程：

1. `"exports"` 是否存在？**是。**
2. `exports` 的值是一个字符串——它必定是包根路径（`"."`）的文件路径。
3. 请求 `"pkg/dist/index.js"` 是否针对包根路径？**否，它包含子路径 `dist/index.js`。**
4. 解析失败；返回 `undefined`。

若没有 `"exports"`，该请求本可以成功；但 `"exports"` 的存在会阻止解析任何无法通过 `"exports"` 匹配的子路径。

#### package.json `"typesVersions"`

[`node_modules` 包](#node_modules-package-lookups)或[目录模块](#directory-modules-index-file-resolution)可以在其 package.json 中指定 `"typesVersions"` 字段，以根据 TypeScript 编译器版本（对于 `node_modules` 包，还根据正在解析的子路径）重定向 TypeScript 的解析过程。这使得包作者可以在一套类型定义中包含较新的 TypeScript 语法，同时提供另一套定义（通过诸如 [downlevel-dts](https://github.com/sandersn/downlevel-dts) 之类的工具）以向后兼容较旧的 TypeScript 版本。`"typesVersions"` 在所有 `moduleResolution` 模式下均受支持；然而，在读取 [package.json `"exports"`](#packagejson-exports) 的情况下，不会读取该字段。

##### 示例：重定向所有请求到子目录

场景：某模块使用 TypeScript 5.2 导入 `"pkg"`，其中 `node_modules/pkg/package.json` 如下：

```json
{
  "name": "pkg",
  "version": "1.0.0",
  "types": "./index.d.ts",
  "typesVersions": {
    ">=3.1": {
      "*": ["ts3.1/*"]
    }
  }
}
```

解析流程：

1. （取决于编译器选项）`"exports"` 是否存在？**否。**
2. `"typesVersions"` 是否存在？**是。**
3. TypeScript 版本是否 `>=3.1`？**是。记录映射 `"*": ["ts3.1/*"]`。**
4. 我们是否在解析包名之后的子路径？**否，仅解析根路径 `"pkg"`。**
5. `"types"` 是否存在？**是。**
6. `"typesVersions"` 中是否有任何键匹配 `./index.d.ts`？**是，`"*"` 匹配成功，并将替换内容设为 `index.d.ts`。**
7. 在 `ts3.1/*` 中，将 `*` 替换为匹配内容 `./index.d.ts`：**`ts3.1/index.d.ts`**。
8. 路径 `./ts3.1/index.d.ts` 是否具有可识别的 TypeScript 文件扩展名？**是，因此不使用扩展名替换。**
9. 若该文件存在则返回路径 `./ts3.1/index.d.ts`，否则返回 `undefined`。

##### 示例：重定向对特定文件的请求

场景：某模块使用 TypeScript 3.9 导入 `"pkg"`，其中 `node_modules/pkg/package.json` 如下：

```json
{
  "name": "pkg",
  "version": "1.0.0",
  "types": "./index.d.ts",
  "typesVersions": {
    "<4.0": { "index.d.ts": ["index.v3.d.ts"] }
  }
}
```

解析流程：

1. （取决于编译器选项）`"exports"` 是否存在？**否。**
2. `"typesVersions"` 是否存在？**是。**
3. TypeScript 版本是否 `<4.0`？**是。记录映射 `"index.d.ts": ["index.v3.d.ts"]`。**
4. 我们是否在解析包名之后的子路径？**否，仅解析根路径 `"pkg"`。**
5. `"types"` 是否存在？**是。**
6. `"typesVersions"` 中是否有任何键匹配 `./index.d.ts`？**是，`"index.d.ts"` 匹配成功。**
7. 路径 `./index.v3.d.ts` 是否具有可识别的 TypeScript 文件扩展名？**是，因此不使用扩展名替换。**
8. 若该文件存在则返回路径 `./index.v3.d.ts`，否则返回 `undefined`。

#### package.json `"main"` 和 `"types"`

如果目录的 [package.json `"exports"`](#packagejson-exports) 字段未被读取（无论是由于编译器选项、该字段不存在，还是由于该目录被作为[目录模块](#directory-modules-index-file-resolution)而非 [`node_modules` 包](#node_modules-package-lookups)进行解析），且模块标识符在包名或包含 package.json 的目录后没有子路径，TypeScript 将尝试按顺序从以下 package.json 字段进行解析，以尝试找到包或目录的主模块：

- `"types"`
- `"typings"`（遗留）
- `"main"`

在 `"types"` 处找到的声明文件被假定为对在 `"main"` 处找到的实现文件的准确表述。如果 `"types"` 和 `"typings"` 不存在或无法解析，TypeScript 将读取 `"main"` 字段并执行[扩展名替换](#file-extension-substitution)以查找声明文件。

向 npm 发布带有类型的包时，即便[扩展名替换](#file-extension-substitution)或 [package.json `"exports"`](#packagejson-exports) 使得 `"types"` 并非必需，仍建议包含 `"types"` 字段，因为 npm 仅在 package.json 包含 `"types"` 字段时才会在包注册表列表中展示 TS 图标。

#### 包相对文件路径

如果 [package.json `"exports"`](#packagejson-exports) 和 [package.json `"typesVersions"`](#packagejson-typesversions) 都不适用，裸包标识符的子路径将根据适用的[相对路径](#relative-file-path-resolution)解析规则，相对于包目录进行解析。在遵循 package.json `"exports"` 的模式中，只要包的 package.json 中存在 `"exports"` 字段，该行为就会被阻止，即便导入无法通过 `"exports"` 解析也是如此，正如[上文示例](#example-exports-block-other-subpaths)所示。另一方面，如果导入无法通过 `"typesVersions"` 解析，则会尝试将包相对文件路径解析作为回退。

当支持包相对路径时，它们在考虑 `moduleResolution` 模式和上下文的前提下，按照与任何其他相对路径相同的规则进行解析。例如，在 [`--moduleResolution nodenext`](#node16-nodenext-1) 中，[目录模块](#directory-modules-index-file-resolution)和[无扩展名路径](#extensionless-relative-paths)仅在 `require` 调用中受支持，而在 `import` 中不受支持：

```ts
// @Filename: module.mts
import 'pkg/dist/foo' // ❌ import, needs `.js` extension
import 'pkg/dist/foo.js' // ✅
import foo = require('pkg/dist/foo') // ✅ require, no extension needed
```

#### package.json `"imports"` 与自名称导入

当 `moduleResolution` 设置为 `node16`、`nodenext` 或 `bundler` 且未禁用 `resolvePackageJsonImports` 时，TypeScript 将尝试通过导入文件最近的祖先 package.json 的 `"imports"` 字段解析以 `#` 开头的导入路径。类似地，当启用 [package.json `"exports"` 查找](#packagejson-exports)时，TypeScript 将尝试通过该 package.json 的 `"exports"` 字段解析以当前包名（即导入文件最近的祖先 package.json 的 `"name"` 字段值）开头的导入路径。这两个特性均允许包内的文件导入同一包内的其他文件，从而取代相对导入路径。

在解析出文件路径之前，TypeScript 对 [`"imports"`](https://nodejs.org/api/packages.html#subpath-imports) 和[自身引用](https://nodejs.org/api/packages.html#self-referencing-a-package-using-its-name)的处理与 Node.js 的解析算法完全一致。到达此步骤时，TypeScript 的解析算法会根据正在解析的包含 `"imports"` 或 `"exports"` 的 package.json 是属于 `node_modules` 依赖项还是属于正在编译的本地项目（即其所在目录包含含有导入文件的项目的 tsconfig.json 文件）产生分流：

- 如果 package.json 位于 `node_modules` 中，当文件路径尚未具有可识别的 TypeScript 文件扩展名时，TypeScript 将对该路径应用[扩展名替换](#file-extension-substitution)，并检查生成的文件路径是否存在。
- 如果 package.json 是本地项目的一部分，则会执行额外的重映射步骤，以找到最终将生成从 `"imports"` 解析出的输出 JavaScript 或声明文件路径的*输入* TypeScript 实现文件。若无此步骤，任何解析 `"imports"` 路径的编译都会引用*上一次编译*生成的输出文件，而非预期包含在当前编译中的其他输入文件。此重映射利用了 tsconfig.json 中的 `outDir`/`declarationDir` 和 `rootDir`，因此使用 `"imports"` 通常需要显式设置 `rootDir`。

这一机制让包作者能够在编写 `"imports"` 和 `"exports"` 字段时仅引用将被发布到 npm 的编译输出，同时在本地开发时仍能直接使用原始的 TypeScript 源文件。

##### 示例：带条件的本地项目

场景：在包含 tsconfig.json 和 package.json 的项目目录中，`"/src/main.mts"` 导入 `"#utils"` 并附带条件 `["types", "node", "import"]`（由 `moduleResolution` 设置以及触发模块解析请求的上下文决定）：

```json5
// tsconfig.json
{
  compilerOptions: {
    moduleResolution: 'node16',
    resolvePackageJsonImports: true,
    rootDir: './src',
    outDir: './dist',
  },
}
```

```json5
// package.json
{
  name: 'pkg',
  imports: {
    '#utils': {
      import: './dist/utils.d.mts',
      require: './dist/utils.d.cts',
    },
  },
}
```

解析流程：

1. 导入路径以 `#` 开头，尝试通过 `"imports"` 解析。
2. 最近的祖先 package.json 中是否存在 `"imports"`？**是。**
3. `"imports"` 对象中是否存在 `"#utils"`？**是。**
4. `imports["#utils"]` 的值是一个对象——它必定是在指定条件。
5. 第一个条件 `"import"` 是否匹配此请求？**是。**
6. 我们是否应尝试将输出路径映射到输入路径？**是，因为：**
   - package.json 是否位于 `node_modules` 中？**否，它位于本地项目中。**
   - tsconfig.json 是否位于 package.json 目录下？**是。**
7. 在 `./dist/utils.d.mts` 中，将 `outDir` 前缀替换为 `rootDir`：**`./src/utils.d.mts`**
8. 将输出扩展名 `.d.mts` 替换为对应的输入扩展名 `.mts`：**`./src/utils.mts`**
9. 若该文件存在，则返回路径 `"./src/utils.mts"`。
10. 否则，若该文件存在，则返回路径 `"./dist/utils.d.mts"`。

##### 示例：带子路径模式的 `node_modules` 依赖

场景：`"/node_modules/pkg/main.mts"` 导入 `"#internal/utils"` 并附带条件 `["types", "node", "import"]`（由 `moduleResolution` 设置以及触发模块解析请求的上下文决定），其 package.json 如下：

```json5
// /node_modules/pkg/package.json
{
  name: 'pkg',
  imports: {
    '#internal/*': {
      import: './dist/internal/*.mjs',
      require: './dist/internal/*.cjs',
    },
  },
}
```

解析流程：

1.  导入路径以 `#` 开头，尝试通过 `"imports"` 解析。
2.  最近的祖先 package.json 中是否存在 `"imports"`？**是。**
3.  `"imports"` 对象中是否存在 `"#internal/utils"`？**否，检查模式匹配。**
4.  是否有任何带 `*` 的键匹配 `"#internal/utils"`？**是，`"#internal/*"` 匹配成功，并将替换内容设为 `utils`。**
5.  `imports["#internal/*"]` 的值是一个对象——它必定是在指定条件。
6.  第一个条件 `"import"` 是否匹配此请求？**是。**
7.  我们是否应尝试将输出路径映射到输入路径？**否，因为 package.json 位于 `node_modules` 中。**
8.  在 `./dist/internal/*.mjs` 中，将 `*` 替换为匹配内容 `utils`：**`./dist/internal/utils.mjs`**
9.  路径 `./dist/internal/utils.mjs` 是否具有可识别的 TypeScript 文件扩展名？**否，尝试扩展名替换。**
10. 通过[扩展名替换](#file-extension-substitution)，尝试以下路径，返回第一个存在的文件，否则返回 `undefined`：
    1. `./dist/internal/utils.mts`
    2. `./dist/internal/utils.d.mts`
    3. `./dist/internal/utils.mjs`

### `node16`, `nodenext`

这些模式反映了 Node.js v12 及更高版本的模块解析行为。（`node16` 和 `nodenext` 目前是完全相同的，但如果 Node.js 将来对其模块系统做出重大变更，`node16` 将被冻结，而 `nodenext` 将会更新以反映新的行为。）在 Node.js 中，ECMAScript import 的解析算法与 CommonJS `require` 调用的算法有显著差异。对于正在解析的每个模块标识符，首先根据语法以及[导入文件的模块格式](#module-format-detection)来确定发射的 JavaScript 中该模块标识符将位于 `import` 还是 `require` 中。然后将该信息传递给模块解析器，以确定使用哪种解析算法（以及针对 package.json [`"exports"`](#packagejson-exports) 或 [`"imports"`](#packagejson-imports-and-self-name-imports) 使用 `"import"` 还是 `"require"` 条件）。

> 默认情况下，[确定为 CommonJS 格式](#module-format-detection)的 TypeScript 文件仍可使用 `import` 和 `export` 语法，但在发射的 JavaScript 中将改用 `require` 和 `module.exports`。这意味着常常会看到使用 `require` 算法进行解析的 `import` 语句。如果这造成了困惑，可以启用 `verbatimModuleSyntax` 编译器选项，该选项会禁止使用会被发射为 `require` 调用的 `import` 语句。

请注意，根据 Node.js 的行为，动态 `import()` 调用始终使用 `import` 算法进行解析。然而，`import()` 类型则是根据导入文件的格式进行解析的（为了向后兼容现有的 CommonJS 格式类型声明）：

```ts
// @Filename: module.mts
import x from './mod.js' // `import` algorithm due to file format (emitted as-written)
import('./mod.js') // `import` algorithm due to syntax (emitted as-written)
type Mod = typeof import('./mod.js') // `import` algorithm due to file format
import mod = require('./mod') // `require` algorithm due to syntax (emitted as `require`)

// @Filename: commonjs.cts
import x from './mod' // `require` algorithm due to file format (emitted as `require`)
import('./mod.js') // `import` algorithm due to syntax (emitted as-written)
type Mod = typeof import('./mod') // `require` algorithm due to file format
import mod = require('./mod') // `require` algorithm due to syntax (emitted as `require`)
```

#### 隐式与强制选项

- `--moduleResolution node16` 和 `nodenext` 必须与 [`--module node16`, `node18`, `node20`, or `nodenext`](#node16-node18-node20-nodenext) 搭配使用。

#### 支持的特性

特性按优先级顺序列出。

|                                                                                     | `import`                          | `require`                          |
| ----------------------------------------------------------------------------------- | --------------------------------- | ---------------------------------- |
| [`paths`](#paths)                                                                   | ✅                                | ✅                                 |
| [`baseUrl`](#baseurl)                                                               | ✅                                | ✅                                 |
| [`node_modules` 包查找](#node_modules-package-lookups)                              | ✅                                | ✅                                 |
| [package.json `"exports"`](#packagejson-exports)                                    | ✅ 匹配 `types`、`node`、`import` | ✅ 匹配 `types`、`node`、`require` |
| [package.json `"imports"` 与自名称导入](#packagejson-imports-and-self-name-imports) | ✅ 匹配 `types`、`node`、`import` | ✅ 匹配 `types`、`node`、`require` |
| [package.json `"typesVersions"`](#packagejson-typesversions)                        | ✅                                | ✅                                 |
| [包相对路径](#package-relative-file-paths)                                          | ✅ 当 `exports` 不存在时          | ✅ 当 `exports` 不存在时           |
| [完整相对路径](#relative-file-path-resolution)                                      | ✅                                | ✅                                 |
| [无扩展名相对路径](#extensionless-relative-paths)                                   | ❌                                | ✅                                 |
| [目录模块](#directory-modules-index-file-resolution)                                | ❌                                | ✅                                 |

### `bundler`

`--moduleResolution bundler` 试图对大多数 JavaScript 打包工具通用的模块解析行为进行建模。简而言之，这意味着支持传统上与 Node.js 的 CommonJS `require` 解析算法相关联的所有行为，如 [`node_modules` 查找](#node_modules-package-lookups)、[目录模块](#directory-modules-index-file-resolution)和[无扩展名路径](#extensionless-relative-paths)，同时也支持较新的 Node.js 解析特性，如 [package.json `"exports"`](#packagejson-exports) 和 [package.json `"imports"`](#packagejson-imports-and-self-name-imports)。

思考 `--moduleResolution bundler` 与 `--moduleResolution nodenext` 之间的异同是很有启发性的，尤其是在它们如何决定解析 package.json `"exports"` 或 `"imports"` 时使用哪些条件方面。考虑 `.ts` 文件中的一条导入语句：

```ts
// index.ts
import { foo } from 'pkg'
```

回顾一下，在 `--module nodenext --moduleResolution nodenext` 下，`--module` 设置首先[确定](#module-format-detection)该导入在发射到 `.js` 文件时是作为 `import` 还是 `require` 调用，然后将该信息传递给 TypeScript 的模块解析器，解析器据此决定在 `"pkg"` 的 package.json `"exports"` 中匹配 `"import"` 还是 `"require"` 条件。假设该文件所在作用域内没有 package.json。文件扩展名为 `.ts`，因此输出文件扩展名为 `.js`，Node.js 会将其解释为 CommonJS，所以 TypeScript 会将该 `import` 发射为 `require` 调用。因此，模块解析器在解析来自 `"pkg"` 的 `"exports"` 时将使用 `require` 条件。

在 `--moduleResolution bundler` 中也会发生相同的过程，但决定是否针对该导入语句发射 `import` 或 `require` 调用的规则将有所不同，因为 `--moduleResolution bundler` 必须使用 [`--module esnext`](#es2015-es2020-es2022-esnext) 或 [`--module preserve`](#preserve)。在这两种模式下，ESM `import` 声明始终发射为 ESM `import` 声明，因此 TypeScript 的模块解析器将接收到该信息，并在解析来自 `"pkg"` 的 `"exports"` 时使用 `"import"` 条件。

这种解释可能略显反直觉，因为 `--moduleResolution bundler` 通常与 `--noEmit` 结合使用——打包工具通常直接处理原始的 `.ts` 文件，并在未转换的 `import` 或 `require` 上执行模块解析。然而，为了保持一致性，TypeScript 仍会使用由 `module` 决定的假设发射来指导模块解析和类型检查。这使得每当运行时或打包工具直接操作原始 `.ts` 文件时，[`--module preserve`](#preserve) 都是最佳选择，因为这意味着不进行任何转换。在 `--module preserve --moduleResolution bundler` 下，你可以在同一个文件中编写导入和 require，它们将分别使用 `import` 和 `require` 条件进行解析：

```ts
// index.ts
import pkg1 from 'pkg' // Resolved with "import" condition
import pkg2 = require('pkg') // Resolved with "require" condition
```

#### 隐式与强制选项

- `--moduleResolution bundler` 必须与 `--module esnext` 或 `--module preserve` 搭配使用。
- `--moduleResolution bundler` 隐式启用 `--allowSyntheticDefaultImports`。

#### 支持的特性

- [`paths`](#paths) ✅
- [`baseUrl`](#baseurl) ✅
- [`node_modules` 包查找](#node_modules-package-lookups) ✅
- [package.json `"exports"`](#packagejson-exports) ✅ 匹配 `types`，根据语法匹配 `import`/`require`
- [package.json `"imports"` 与自名称导入](#packagejson-imports-and-self-name-imports) ✅ 匹配 `types`，根据语法匹配 `import`/`require`
- [package.json `"typesVersions"`](#packagejson-typesversions) ✅
- [包相对路径](#package-relative-file-paths) ✅ 当 `exports` 不存在时
- [完整相对路径](#relative-file-path-resolution) ✅
- [无扩展名相对路径](#extensionless-relative-paths) ✅
- [目录模块](#directory-modules-index-file-resolution) ✅

### `node10`（原名 `node`）

在 TypeScript 5.0 中，`--moduleResolution node` 被重命名为 `node10`（保留 `node` 作为别名以向后兼容）。它反映了早于 v12 的 Node.js 版本中存在的 CommonJS 模块解析算法。它不应再被使用。

#### 支持的特性

- [`paths`](#paths) ✅
- [`baseUrl`](#baseurl) ✅
- [`node_modules` 包查找](#node_modules-package-lookups) ✅
- [package.json `"exports"`](#packagejson-exports) ❌
- [package.json `"imports"` 与自名称导入](#packagejson-imports-and-self-name-imports) ❌
- [package.json `"typesVersions"`](#packagejson-typesversions) ✅
- [包相对路径](#package-relative-file-paths) ✅
- [完整相对路径](#relative-file-path-resolution) ✅
- [无扩展名相对路径](#extensionless-relative-paths) ✅
- [目录模块](#directory-modules-index-file-resolution) ✅

### `classic`

不要使用 `classic`。
