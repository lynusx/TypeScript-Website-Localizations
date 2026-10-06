---
title: TypeScript 5.7
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-5-7.html
oneline: TypeScript 5.7 发布说明
---

## 未初始化变量的检查

长期以来，当变量在所有先前的分支中尚未被完全初始化时，TypeScript 都能够捕获此类问题：

```ts
let result: number
if (someCondition()) {
  result = doSomeWork()
} else {
  let temporaryWork = doSomeWork()
  temporaryWork *= 2
  // forgot to assign to 'result'
}

console.log(result) // error: Variable 'result' is used before being assigned.
```

遗憾的是，在某些情况下控制流分析无法奏效。
例如，若变量在另一个独立的函数内部被访问，类型系统无法确切知晓该函数何时会被调用，因而采取了“乐观”假定，认为该变量已被初始化：

```ts
function foo() {
  let result: number
  if (someCondition()) {
    result = doSomeWork()
  } else {
    let temporaryWork = doSomeWork()
    temporaryWork *= 2
    // forgot to assign to 'result'
  }

  printResult()

  function printResult() {
    console.log(result) // no error here.
  }
}
```

尽管 TypeScript 5.7 对*可能*已被初始化的变量仍然保持宽松，但现在当变量*完全未被初始化*时，类型系统能够准确报告错误：

```ts
function foo() {
  let result: number

  // do work, but forget to assign to 'result'

  function printResult() {
    console.log(result) // error: Variable 'result' is used before being assigned.
  }
}
```

感谢 GitHub 用户 [Zzzen](https://github.com/Zzzen) 贡献了[此项改动](https://github.com/microsoft/TypeScript/pull/55887)！

## 相对路径的路径重写

现有若干工具和运行时允许“就地”（in-place）直接运行 TypeScript 代码，这意味着它们不需要生成输出 JavaScript 文件的构建步骤。
例如，ts-node、tsx、Deno 和 Bun 都支持直接运行 `.ts` 文件。
最近，Node.js 也在通过 `--experimental-strip-types`（即将无需实验性标志默认启用！）以及 `--experimental-transform-types` 探索此类支持。
这带来了极大的便利，因为它让我们能够更快地迭代，而无需操心重复执行构建任务。

不过在使用这些模式时，需要注意一些复杂细节。
为了与所有这些工具保持最大兼容性，就地导入的 TypeScript 文件在运行时**必须**携带对应的 TypeScript 后缀进行导入。
例如，要导入一个名为 `foo.ts` 的文件，在 Node 的全新实验性支持中必须写成：

```ts
// main.ts

import * as foo from './foo.ts' // <- we need foo.ts here, not foo.js
```

通常情况下，如果我们这样做，TypeScript 会报错，因为它期望我们导入*输出文件*。
由于某些工具确实允许 `.ts` 后缀导入，TypeScript 此前通过 `--allowImportingTsExtensions` 选项对这种导入风格提供了支持。
这本身运行良好，但如果我们最终确实需要从这些 `.ts` 文件中生成 `.js` 文件呢？
对于需要仅分发 `.js` 文件的库作者来说，这是一个刚需，但在此之前 TypeScript 一直避免重写任何路径。

为了支持这种场景，我们新增了一个编译器选项 `--rewriteRelativeImportExtensions`。
当导入路径为*相对路径*（以 `./` 或 `../` 开头）、以 TypeScript 后缀（`.ts`、`.tsx`、`.mts`、`.cts`）结尾，且是非声明文件时，编译器会将该路径重写为对应的 JavaScript 后缀（`.js`、`.jsx`、`.mjs`、`.cjs`）：

```ts
// Under --rewriteRelativeImportExtensions...

// these will be rewritten.
import * as foo from './foo.ts'
import * as bar from '../someFolder/bar.mts'

// these will NOT be rewritten in any way.
import * as a from './foo'
import * as b from 'some-package/file.ts'
import * as c from '@some-scope/some-package/file.ts'
import * as d from '#/file.ts'
import * as e from './file.js'
```

这使我们能够编写既可就地直接运行、又可在准备就绪后编译为 JavaScript 的 TypeScript 代码。

前面提到，TypeScript 过去通常避免重写路径。
这背后有几个原因，其中最明显的一个是动态导入。
如果开发者写出如下代码，要处理 `import` 接收的路径绝非易事。
事实上，根本不可能在任何依赖项内部覆盖 `import` 的行为：

```ts
function getPath() {
  if (Math.random() < 0.5) {
    return './foo.ts'
  } else {
    return './foo.js'
  }
}

let myImport = await import(getPath())
```

另一个问题是（正如我们在上面所见），只有*相对路径*会被重写，而且重写是“朴素”（naive）进行的。
这意味着任何依赖于 TypeScript `baseUrl` 和 `paths` 的路径都不会被重写：

```json5
// tsconfig.json

{
  compilerOptions: {
    module: 'nodenext',
    // ...
    paths: {
      '@/*': ['./src/*'],
    },
  },
}
```

```ts
// Won't be transformed, won't work.
import * as utilities from '@/utilities.ts'
```

任何可能通过 `package.json` 的 [`exports`](https://nodejs.org/api/packages.html#exports) 和 [`imports`](https://nodejs.org/api/packages.html#imports) 字段解析的路径也同样不会被重写：

```json5
// package.json
{
  name: 'my-package',
  imports: {
    '#root/*': './dist/*',
  },
}
```

```ts
// Won't be transformed, won't work.
import * as utilities from '#root/utilities.ts'
```

因此，如果你一直采用多包相互引用的 monorepo 工作区架构，可能需要结合[带作用域的自定义条件](https://nodejs.org/api/packages.html#resolving-user-conditions)使用[条件导出](https://nodejs.org/api/packages.html#conditional-exports)来实现这一目标：

```json5
// my-package/package.json

{
  name: 'my-package',
  exports: {
    '.': {
      '@my-package/development': './src/index.ts',
      import: './lib/index.js',
    },
    './*': {
      '@my-package/development': './src/*.ts',
      import: './lib/*.js',
    },
  },
}
```

任何时候想要导入 `.ts` 文件，都可以通过 `node --conditions=@my-package/development` 运行。

请注意我们在条件中使用的“命名空间”或“作用域”`@my-package/development`。
这是一种权宜之计，用于避免与其他可能同样使用 `development` 条件的依赖项发生冲突。
如果所有包都在自身发布物中包含 `development` 条件，解析可能会尝试解析到一个未编译的 `.ts` 文件，而这未必能正常工作。
这一思路类似于 Colin McDonnell 在文章《[TypeScript Monorepo 中的实时类型](https://colinhacks.com/essays/live-types-typescript-monorepo#:~:text=custom%20conditions)》（_Live types in a TypeScript monorepo_）中所描述的方法，以及 [tshy 关于从源码加载的指南](https://github.com/isaacs/tshy#loading-from-source)。

欲了解此特性工作原理的更多细节，请[在此阅读该改动的详细说明](https://github.com/microsoft/TypeScript/pull/59767)。

## 支持 `--target es2024` 与 `--lib es2024`

TypeScript 5.7 现在支持 `--target es2024`，允许用户面向 ECMAScript 2024 运行时环境。
该目标最主要的作用是支持指定全新的 `--lib es2024`，其中包含了针对 `SharedArrayBuffer` 与 `ArrayBuffer` 的众多特性、`Object.groupBy`、`Map.groupBy`、`Promise.withResolvers` 等。
它还将 `Atomics.waitAsync` 从 `--lib es2022` 移动到了 `--lib es2024`。

请注意，作为针对 `SharedArrayBuffer` 和 `ArrayBuffer` 改动的一部分，二者现在产生了一些分歧。
为了消除差异并保留底层的缓冲区类型，所有 `TypedArray`（如 `Uint8Array` 等）[现在也都是泛型类型](https://github.com/microsoft/TypeScript/pull/59417)：

```ts
interface Uint8Array<TArrayBuffer extends ArrayBufferLike = ArrayBufferLike> {
  // ...
}
```

每个 `TypedArray` 现在都包含一个名为 `TArrayBuffer` 的类型参数，不过该类型参数带有默认类型实参，因此我们可以继续直接使用 `Int32Array`，而无需显式写成 `Int32Array<ArrayBufferLike>`。

如果你在此次更新中遇到任何问题，可能需要更新 `@types/node`。

[此项工作](https://github.com/microsoft/TypeScript/pull/58573)主要由 [Kenta Moriuchi](https://github.com/petamoriken) 提供，特此致谢！

## 向上搜索祖先配置文件以确定项目归属

当 TypeScript 文件在使用 TSServer 的编辑器（如 Visual Studio 或 VS Code）中打开时，编辑器会尝试查找“拥有”该文件的相关 `tsconfig.json` 配置文件。
为此，它会从正在编辑的文件所在的目录沿目录树向上遍历，寻找名为 `tsconfig.json` 的文件。

此前，该搜索在找到第一个 `tsconfig.json` 文件时就会停止；
然而，设想如下的项目结构：

```
project/
├── src/
│   ├── foo.ts
│   ├── foo-test.ts
│   ├── tsconfig.json
│   └── tsconfig.test.json
└── tsconfig.json
```

在此结构中，设计的设想是 `src/tsconfig.json` 为项目的“主”配置文件，而 `src/tsconfig.test.json` 则是用于运行测试的配置文件：

```json5
// src/tsconfig.json
{
  compilerOptions: {
    outDir: '../dist',
  },
  exclude: ['**/*.test.ts'],
}
```

```json5
// src/tsconfig.test.json
{
  compilerOptions: {
    outDir: '../dist/test',
  },
  include: ['**/*.test.ts'],
  references: [{ path: './tsconfig.json' }],
}
```

```json5
// tsconfig.json
{
  // This is a "workspace-style" or "solution-style" tsconfig.
  // Instead of specifying any files, it just references all the actual projects.
  files: [],
  references: [
    { path: './src/tsconfig.json' },
    { path: './src/tsconfig.test.json' },
  ],
}
```

这里的问题在于，当编辑 `foo-test.ts` 时，编辑器会把 `project/src/tsconfig.json` 判定为“归属”它的配置文件——但这并不是我们想要的！
如果遍历在此处直接停止，就无法满足需求。
此前规避此问题的唯一方法是将 `src/tsconfig.json` 重命名为类似 `src/tsconfig.src.json` 的名称，这样所有文件都会匹配到引用了所有子项目的顶层 `tsconfig.json`：

```
project/
├── src/
│   ├── foo.ts
│   ├── foo-test.ts
│   ├── tsconfig.src.json
│   └── tsconfig.test.json
└── tsconfig.json
```

TypeScript 5.7 不再强制开发者采用这种迂回做法，而是在编辑器场景下继续向上遍历目录树，以寻找其他更匹配的 `tsconfig.json` 文件。
这为项目的组织方式和配置文件的架构设计提供了更大的灵活性。

关于该实现的更多细节，可以在 GitHub 上的[此处](https://github.com/microsoft/TypeScript/pull/57196)和[此处](https://github.com/microsoft/TypeScript/pull/59688)查看。

## 编辑器中复合项目（Composite Projects）归属检查的速度提升

设想一个拥有如下结构的大型代码库：

```
packages
├── graphics/
│   ├── tsconfig.json
│   └── src/
│       └── ...
├── sound/
│   ├── tsconfig.json
│   └── src/
│       └── ...
├── networking/
│   ├── tsconfig.json
│   └── src/
│       └── ...
├── input/
│   ├── tsconfig.json
│   └── src/
│       └── ...
└── app/
    ├── tsconfig.json
    ├── some-script.js
    └── src/
        └── ...
```

`packages` 下的每个目录都是一个独立的 TypeScript 项目，而 `app` 目录是依赖于其他所有项目的主项目：

```json5
// app/tsconfig.json
{
  compilerOptions: {
    // ...
  },
  include: ['src'],
  references: [
    { path: '../graphics/tsconfig.json' },
    { path: '../sound/tsconfig.json' },
    { path: '../networking/tsconfig.json' },
    { path: '../input/tsconfig.json' },
  ],
}
```

现在请注意，我们在 `app` 目录中有一个文件 `some-script.js`。
当我们在编辑器中打开 `some-script.js` 时，TypeScript 语言服务（它也负责 JavaScript 文件的编辑器体验！）必须弄清楚该文件属于哪个项目，以便应用正确的配置。

在这种情况下，距离最近的 `tsconfig.json` 并*未*包含 `some-script.js`，但 TypeScript 会继续探寻：“被 `app/tsconfig.json` *所引用*的各个项目中，是否有一个包含 `some-script.js` 呢？”。
为此，TypeScript 此前会逐个加载每个项目，并在找到包含 `some-script.js` 的项目时立即停止。
即使 `some-script.js` 不在该项目的根文件集合中，TypeScript 仍会解析该项目内的所有文件，因为根文件集合中的某些文件可能会*传递性地*引用 `some-script.js`。

随着时间的推移，我们发现这种行为在大型代码库中会导致极其严重且不可预测的性能问题。
开发者打开一个零散的脚本文件，却发现整个代码库都被迫加载并解析了一遍。

幸运的是，每个能被另一个（非工作区）项目所引用的项目都必须启用名为 `composite` 的标志，该标志强制要求所有输入源文件必须预先明确可知。
因此，在探测 `composite` 项目时，TypeScript 5.7 将仅检查该文件是否属于该项目的*根文件集合*（root set of files）。
这应能有效避免此类常见的最坏情况。

欲了解更多信息，请[查看此处的改动](https://github.com/microsoft/TypeScript/pull/59688)。

### `--module nodenext` 下校验 JSON 导入

在 `--module nodenext` 下从 `.json` 文件导入时，TypeScript 现在会强制执行某些规则以防止运行时错误。

首先，任何 JSON 文件导入都必须包含带有 `type: "json"` 的导入属性（import attribute）：

```ts
import myConfig from './myConfig.json'
//                   ~~~~~~~~~~~~~~~~~
// ❌ error: Importing a JSON file into an ECMAScript module requires a 'type: "json"' import attribute when 'module' is set to 'NodeNext'.

import myConfig from './myConfig.json' with { type: 'json' }
//                                          ^^^^^^^^^^^^^^^^
// ✅ This is fine because we provided `type: "json"`
```

在此验证的基础上，TypeScript 将不会生成“命名”导出，并且导入的 JSON 内容将只能通过默认导出访问：

```ts
// ✅ This is okay:
import myConfigA from './myConfig.json' with { type: 'json' }
let version = myConfigA.version

///////////

import * as myConfigB from './myConfig.json' with { type: 'json' }

// ❌ This is not:
let version = myConfig.version

// ✅ This is okay:
let version = myConfig.default.version
```

关于该改动的更多信息，请[参见此处](https://github.com/microsoft/TypeScript/pull/60019)。

## 支持 Node.js 中的 V8 编译缓存

Node.js 22 支持了[名为 `module.enableCompileCache()` 的新 API](https://github.com/nodejs/node/pull/54501)。
该 API 允许运行时在工具初次运行之后，复用部分解析和编译成果。

TypeScript 5.7 现在利用了该 API，使其能够更快进入核心工作状态。
在我们自己的部分测试中，执行 `tsc --version` 获得了约 2.5 倍的速度提升：

```
Benchmark 1: node ./built/local/_tsc.js --version (*without* caching)
  Time (mean ± σ):     122.2 ms ±   1.5 ms    [User: 101.7 ms, System: 13.0 ms]
  Range (min … max):   119.3 ms … 132.3 ms    200 runs

Benchmark 2: node ./built/local/tsc.js --version  (*with* caching)
  Time (mean ± σ):      48.4 ms ±   1.0 ms    [User: 34.0 ms, System: 11.1 ms]
  Range (min … max):    45.7 ms …  52.8 ms    200 runs

Summary
  node ./built/local/tsc.js --version ran
    2.52 ± 0.06 times faster than node ./built/local/_tsc.js --version
```

欲了解更多信息，请[查看此处的 Pull Request](https://github.com/microsoft/TypeScript/pull/59720)。

## 显著的行为变更

本节重点介绍在任何版本升级过程中都应知晓并理解的重要变更。
内容可能涵盖弃用项、移除项和新增的限制规则，也包括在功能上属于修复与改进、但可能因引入新错误而影响现有构建的改动。

### `lib.d.ts`

针对 DOM 生成的类型可能会对代码库的类型检查产生影响。
欲了解更多信息，请[查看与本版本 TypeScript 的 DOM 及 `lib.d.ts` 更新相关的议题](https://github.com/microsoft/TypeScript/pull/60061)。

### `TypedArray` 现已基于 `ArrayBufferLike` 泛型化

在 ECMAScript 2024 中，`SharedArrayBuffer` 与 `ArrayBuffer` 的类型产生了一些分歧。
为了消除差异并保留底层的缓冲区类型，所有 `TypedArray`（如 `Uint8Array` 等）[现在也都是泛型类型](https://github.com/microsoft/TypeScript/pull/59417)：

```ts
interface Uint8Array<TArrayBuffer extends ArrayBufferLike = ArrayBufferLike> {
  // ...
}
```

每个 `TypedArray` 现在都包含一个名为 `TArrayBuffer` 的类型参数，不过该类型参数带有默认类型实参，因此用户可以继续直接使用 `Int32Array`，而无需显式写成 `Int32Array<ArrayBufferLike>`。

如果你在此次更新中遇到任何问题，例如：

```
error TS2322: Type 'Buffer' is not assignable to type 'Uint8Array<ArrayBufferLike>'.
error TS2345: Argument of type 'Buffer' is not assignable to parameter of type 'Uint8Array<ArrayBufferLike>'.
error TS2345: Argument of type 'ArrayBufferLike' is not assignable to parameter of type 'ArrayBuffer'.
error TS2345: Argument of type 'Buffer' is not assignable to parameter of type 'string | ArrayBufferView | Stream | Iterable<string | ArrayBufferView> | AsyncIterable<string | ArrayBufferView>'.
```

那么你可能需要更新 `@types/node`。

你可以在 GitHub 上[阅读有关此改动的具体细节](https://github.com/microsoft/TypeScript/pull/59417)。

### 从类中的非字面量方法名创建索引签名

对于使用非字面量计算属性名声明的类方法，TypeScript 现在具备了更加一致的行为。
例如在以下代码中：

```ts
declare const symbolMethodName: symbol

export class A {
  [symbolMethodName]() {
    return 1
  }
}
```

此前，TypeScript 仅将该类视为形如：

```ts
export class A {}
```

换言之，从类型系统的角度来看，`[symbolMethodName]` 对 `A` 的类型没有任何贡献。

TypeScript 5.7 现在对 `[symbolMethodName]() {}` 方法的处理更加合理，并为其生成了索引签名。
因此，上述代码会被解释为类似于如下代码：

```ts
export class A {
  [x: symbol]: () => number
}
```

这提供了与对象字面量中的属性和方法保持一致的行为。

[在此阅读有关此改动的更多信息](https://github.com/microsoft/TypeScript/pull/59860)。

### 在未开启 `strictNullChecks` 时返回 `null` 和 `undefined` 的函数会出现更多隐式 `any` 错误

当一个函数表达式被返回泛型类型的签名上下文类型化时，在开启 `noImplicitAny` 但未开启 `strictNullChecks` 的情况下，TypeScript 现在会适当地报告隐式 `any` 错误：

```ts
declare var p: Promise<number>
const p2 = p.catch(() => null)
//                 ~~~~~~~~~~
// error TS7011: Function expression, which lacks return-type annotation, implicitly has an 'any' return type.
```

[更多细节参见此改动](https://github.com/microsoft/TypeScript/pull/59661)。
