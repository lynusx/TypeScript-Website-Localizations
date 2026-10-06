---
title: TypeScript 5.8
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-5-8.html
oneline: TypeScript 5.8 发布说明
---

## 返回表达式中分支的更细粒度检查

考虑如下代码：

```ts
declare const untypedCache: Map<any, any>

function getUrlObject(urlString: string): URL {
  return untypedCache.has(urlString) ? untypedCache.get(urlString) : urlString
}
```

这段代码的初衷是：如果缓存中存在 URL 对象就直接获取，不存在时则创建一个新的 URL 对象。
然而，代码中存在一个 bug：我们忘记使用传入的参数去真正构造一个新的 URL 对象。
遗憾的是，以往的 TypeScript 通常无法捕获此类 bug。

当 TypeScript 检查类似于 `cond ? trueBranch : falseBranch` 的条件表达式时，该表达式的类型会被视为两个分支类型的联合类型。
换句话说，它会分别获取 `trueBranch` 和 `falseBranch` 的类型，并将它们合并为一个联合类型。
在此示例中，`untypedCache.get(urlString)` 的类型为 `any`，而 `urlString` 的类型为 `string`。
问题正出在这里，因为 `any` 在与其他类型交互时具有极强的传染性。
联合类型 `any | string` 会被简化为 `any`，因此当 TypeScript 开始检查 `return` 语句中的表达式是否与预期的返回类型 `URL` 兼容时，类型系统中已经丢失了本可用于捕获该 bug 的类型信息。

在 TypeScript 5.8 中，类型系统对直接位于 `return` 语句内部的条件表达式进行了特殊处理。
条件表达式的每个分支都会独立与所在函数声明的返回类型（如果存在的话）进行检查，因此类型系统能够成功捕获上述示例中的 bug。

```ts
declare const untypedCache: Map<any, any>

function getUrlObject(urlString: string): URL {
  return untypedCache.has(urlString) ? untypedCache.get(urlString) : urlString
  //  ~~~~~~~~~
  // 错误！类型 'string' 不能赋值给类型 'URL'。
}
```

该改动在[此 Pull Request](https://github.com/microsoft/TypeScript/pull/56941) 中实现，是 TypeScript 后续一系列更广泛改进计划的一部分。

## 在 `--module nodenext` 下支持 `require()` 加载 ECMAScript 模块

多年以来，Node.js 同时支持 ECMAScript 模块（ESM）与 CommonJS 模块。
然而，这两者之间的互操作性一直面临一些挑战：

- ESM 文件可以 `import` CommonJS 文件
- CommonJS 文件 _**无法**_ `require()` ESM 文件

换句话说，在 ESM 文件中引用 CommonJS 文件是可行的，反过来却不行。
这给希望提供 ESM 支持的库作者带来了诸多挑战。
库作者要么打破与 CommonJS 用户的兼容性，要么对其库进行“双重发布（dual-publish）”（为 ESM 和 CommonJS 提供独立的入口），要么就无限期停留在 CommonJS。
虽然双重发布听起来像是一个折中方案，但它流程复杂且容易出错，还会使 npm 包内的代码量几乎翻倍。

Node.js 22 放宽了部分限制，允许 CommonJS 模块通过 `require("esm")` 调用来加载 ECMAScript 模块。
Node.js 仍然不允许对包含顶层 `await` 的 ESM 文件使用 `require()`，但大多数其他 ESM 文件现在都可以在 CommonJS 文件中直接加载。
这为库作者提供了一个绝佳的机会，使他们无需双重发布即可提供 ESM 支持。

TypeScript 5.8 在 `--module nodenext` 标志下支持了这一行为。
当启用 `--module nodenext` 时，TypeScript 将不再对此类针对 ESM 文件的 `require()` 调用报错。

由于该特性可能会向后移植（back-port）到旧版本的 Node.js，因此目前尚无启用该行为的稳定版 `--module nodeXXXX` 选项；
不过我们预计 TypeScript 的后续版本可能会在 `node20` 下稳定支持该特性。
在此期间，我们建议 Node.js 22 及更高版本的用户使用 `--module nodenext`，而库作者以及旧版本 Node.js 的用户则应保留在 `--module node16`（或微调升级至 [`--module node18`](#--module-node18)）。

有关详细信息，请[参见支持 require("esm") 的 PR](https://github.com/microsoft/TypeScript/pull/60761)。

## `--module node18`

TypeScript 5.8 引入了稳定的 `--module node18` 标志。
对于固定使用 Node.js 18 的用户，该标志提供了一个稳定的参考基准，且不会包含 `--module nodenext` 中的某些特定行为。
具体而言：

- 在 `node18` 下禁止对 ECMAScript 模块使用 `require()`，但在 `nodenext` 下允许
- 在 `node18` 下允许使用导入断言（已被弃用并推荐使用导入属性），但在 `nodenext` 下禁止

更多内容请参阅 [`--module node18` Pull Request](https://github.com/microsoft/TypeScript/pull/60722) 以及[对 `--module nodenext` 所做的变更](https://github.com/microsoft/TypeScript/pull/60761)。

## `--erasableSyntaxOnly` 选项

最近，Node.js 23.6 默认开启了[直接运行 TypeScript 文件的实验性支持](https://nodejs.org/api/typescript.html#type-stripping)；
然而在此模式下仅支持特定语法结构。
Node.js 默认启用的 `--experimental-strip-types` 模式要求所有 TypeScript 专有语法都不能具备运行时语义。
换句话说，必须能够简单地将文件中的所有 TypeScript 专有语法直接擦除（erase）或“剥离（strip out）”，剥离后留下的必须是合法的 JavaScript 文件。

这意味着不支持以下语法结构：

- `enum` 声明
- 带有运行时代码的 `namespace` 与 `module`
- 类中的参数属性（parameter properties）
- 非 ECMAScript 的 `import =` 和 `export =` 赋值

以下是不受支持的语法示例：

```ts
// ❌ 错误：`import ... = require(...)` 别名
import foo = require('foo')

// ❌ 错误：包含运行时代码的 namespace。
namespace container {}

// ❌ 错误：`import =` 别名
import Bar = container.Bar

class Point {
  // ❌ 错误：参数属性
  constructor(
    public x: number,
    public y: number,
  ) {}
}

// ❌ 错误：`export =` 赋值。
export = Point

// ❌ 错误：enum 声明。
enum Direction {
  Up,
  Down,
  Left,
  Right,
}
```

类似的工具如 [ts-blank-space](https://github.com/bloomberg/ts-blank-space) 或 [Amaro](https://github.com/nodejs/amaro)（Node.js 类型剥离的底层库）也具有相同的限制。
当遇到不符合要求的代码时，这些工具会提供有用的错误信息，但直到真正尝试运行代码时，你才会发现代码无法工作。

因此，TypeScript 5.8 引入了 `--erasableSyntaxOnly` 标志。
启用此标志后，TypeScript 将对大多数具有运行时行为的 TypeScript 专有语法结构报错。

```ts
class C {
    constructor(public x: number) { }
    //          ~~~~~~~~~~~~~~~~
    // 错误！启用 'erasableSyntaxOnly' 时不允许使用该语法。
    }
}
```

通常，你会希望将此标志与 `--verbatimModuleSyntax` 结合使用，后者可确保模块包含适当的导入语法，且不会发生导入省略（import elision）。

有关更多信息，请[参见具体实现](https://github.com/microsoft/TypeScript/pull/61011)。

## `--libReplacement` 标志

在 TypeScript 4.5 中，我们引入了使用自定义文件替换默认 `lib` 文件的功能。
这是基于从名为 `@typescript/lib-*` 的 npm 包中解析库文件的机制实现的。
例如，你可以通过以下 `package.json` 将 `dom` 库锁定在 [`@types/web` 包](https://www.npmjs.com/package/@types/web?activeTab=readme)的特定版本上：

```json
{
  "devDependencies": {
    "@typescript/lib-dom": "npm:@types/web@0.0.199"
  }
}
```

安装完成后，会存在一个名为 `@typescript/lib-dom` 的包，当前只要你的配置中隐式包含了 `dom`，TypeScript 就始终会查找该包。

这是一项强大的特性，但也会带来一些额外开销。
即使你并没有使用该特性，TypeScript 依然会始终执行此项查找，并且必须监视 `node_modules` 的变化，以防某个 `lib` 替换包在后续被安装进来。

TypeScript 5.8 引入了 `--libReplacement` 标志，允许你禁用此行为。
如果你没有使用库替换功能，现在可以通过 `--libReplacement false` 将其关闭。
未来 `--libReplacement false` 可能会成为默认设置，因此如果你当前依赖此行为，建议显式配置 `--libReplacement true`。

有关更多信息，请[参阅此处的问题讨论与改动](https://github.com/microsoft/TypeScript/issues/61023)。

## 声明文件中保留计算属性名

为了让计算属性在声明文件生成时更加可预测，TypeScript 5.8 将始终在类的计算属性名中保留实体名称（例如 `bareVariables` 以及 `dotted.names.that.look.like.this` 这类名称）。

例如，考虑以下代码：

```ts
export let propName = 'theAnswer'

export class MyClass {
  [propName] = 42
  //  ~~~~~~~~~~
  // 错误！
  // 类属性声明中的计算属性名必须具有简单的字面量类型或 'unique symbol' 类型。
}
```

在此前版本的 TypeScript 中，为该模块生成声明文件时会报错，并且尽最大努力生成的声明文件会产生一个索引签名：

```ts
export declare let propName: string
export declare class MyClass {
  [x: string]: number
}
```

在 TypeScript 5.8 中，上述示例代码现在已完全允许，生成的声明文件将与你书写的代码保持一致：

```ts
export declare let propName: string
export declare class MyClass {
  [propName]: number
}
```

请注意，这并不会在类上创建静态命名的属性。
其最终效果仍然等同于 `[x: string]: number` 这样的索引签名；若要支持该用例，你仍需要使用 `unique symbol` 或字面量类型。

另请注意，在 `--isolatedDeclarations` 标志下，书写此类代码过去以及目前依然会报错；
但我们预计由于这项改进，计算属性名在声明文件生成中未来将得到更普遍的允许。

还需要注意，使用 TypeScript 5.8 编译的文件有可能（尽管概率较低）生成无法向后兼容 TypeScript 5.7 或更早版本的声明文件。

有关更多信息，请[参见实现该功能的 PR](https://github.com/microsoft/TypeScript/pull/60052)。

## 程序加载与更新的性能优化

TypeScript 5.8 引入了多项性能优化，不仅缩短了构建 Program 所需的时间，还提升了在 `--watch` 模式或编辑器场景下基于文件变更更新 Program 的速度。

首先，TypeScript 现在[避免了规范化路径时涉及的数组内存分配](https://github.com/microsoft/TypeScript/pull/60812)。
通常，路径规范化需要将路径的每个部分拆分为字符串数组，根据相对路径片段对结果路径进行规范化，然后使用标准分隔符重新拼接。
对于包含大量文件的项目，这是一项庞大且重复的工作。
TypeScript 现在避免了分配数组，而是更直接地在原始路径的索引上进行操作。

此外，当所做的编辑并未改变项目的基本结构时，[TypeScript 现在避免重新校验传给它的选项](https://github.com/microsoft/TypeScript/pull/60754)（例如 `tsconfig.json` 的内容）。
这意味着，例如一次简单的代码编辑可能不再需要重新检查项目的输出路径是否与输入路径发生冲突。
相反，它可以直接复用上一次检查的结果。
这将使大型项目中的代码编辑体验更加流畅敏捷。

## 显著的行为变更

本节列出了一系列值得关注的变更，作为任何版本升级的一部分，都应当对其予以了解和掌握。
其中有时会列出弃用、移除和新限制等；
也可能包含功能改进类的错误修复，但这些修复可能会因引入新的类型错误而影响现有项目的构建。

### `lib.d.ts`

为 DOM 生成的类型可能会对代码库的类型检查产生影响。
有关更多信息，请[参阅与此版本 TypeScript 中 DOM 和 `lib.d.ts` 更新相关的议题列表](https://github.com/microsoft/TypeScript/issues/60985)。

### `--module nodenext` 下对导入断言的限制

导入断言（Import assertions）曾是 ECMAScript 的一项提案，用于确保导入模块具有某些特定属性（例如“该模块是 JSON，不应作为可执行的 JavaScript 代码运行”）。
后来该提案被重新设计为名为[导入属性（import attributes）](https://github.com/tc39/proposal-import-attributes)的提案。
作为过渡的一部分，语法从使用 `assert` 关键字转变为使用 `with` 关键字。

```ts
// 导入断言 ❌ - 与大多数运行时的未来版本不兼容。
import data from './data.json' assert { type: 'json' }

// 导入属性 ✅ - 导入 JSON 文件的推荐方式。
import data from './data.json' with { type: 'json' }
```

Node.js 22 不再接受使用 `assert` 语法的导入断言。
相应地，在 TypeScript 5.8 中启用 `--module nodenext` 时，一旦遇到导入断言，TypeScript 就会报错。

```ts
import data from './data.json' assert { type: 'json' }
//                             ~~~~~~
// 错误！导入断言已被导入属性取代。请使用 'with' 代替 'assert'
```

有关更多信息，请[参阅此处的变更](https://github.com/microsoft/TypeScript/pull/60761)。
