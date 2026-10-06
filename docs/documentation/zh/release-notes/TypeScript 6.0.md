---
title: TypeScript 6.0
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-6-0.html
oneline: TypeScript 6.0 发布说明
---

## 针对无 `this` 函数的更少上下文敏感度

当参数未显式标注类型时，TypeScript 通常可以根据期望的类型、甚至是同一个函数调用中的其他参数来推断它们。

```ts
declare function callIt<T>(obj: {
  produce: (x: number) => T
  consume: (y: T) => void
}): void

// 正常运行，无问题。
callIt({
  produce: (x: number) => x * 2,
  consume: (y) => y.toFixed(),
})

// 正常运行，即使属性顺序颠倒也没有问题。
callIt({
  consume: (y) => y.toFixed(),
  produce: (x: number) => x * 2,
})
```

在此处，无论属性的顺序如何，TypeScript 都能根据从 `produce` 函数推断出的 `T` 来推断 `consume` 函数中 `y` 的类型。
但如果这些函数使用*方法语法*而非箭头函数语法来书写，情况又会如何？

```ts
declare function callIt<T>(obj: {
  produce: (x: number) => T
  consume: (y: T) => void
}): void

// 运行正常，`x` 被推断为 number。
callIt({
  produce(x: number) {
    return x * 2
  },
  consume(y) {
    return y.toFixed()
  },
})

callIt({
  consume(y) {
    return y.toFixed()
  },
  //                  ~
  // 错误：'y' 的类型为 'unknown'。

  produce(x: number) {
    return x * 2
  },
})
```

奇怪的是，对 `callIt` 的第二次调用会导致错误，因为 TypeScript 无法推断 `consume` 方法中 `y` 的类型。
之所以出现这种情况，是因为当 TypeScript 试图寻找 `T` 的候选类型时，首先会跳过那些参数没有显式类型的函数。
之所以这样做，是因为某些函数可能需要 `T` 的推断类型才能被正确检查——在我们的示例中，我们需要知道 `T` 的类型才能分析 `consume` 函数。

这些函数被称为*上下文敏感函数*——本质上就是参数没有显式类型的函数。
类型系统最终需要确定这些参数的类型，但这与泛型函数中的类型推断机制存在冲突，因为两者在推断类型时是在朝不同方向“拉扯”。

```ts
function callFunc<T>(callback: (x: T) => void, value: T) {
  return callback(value)
}

callFunc((x) => x.toFixed(), 42)
//       ^
// 我们需要在此推导 `x` 的类型，
// 但我们还需要推导 `T` 的类型以检查回调。
```

为了解决这个问题，TypeScript 在类型实参推断期间会跳过上下文敏感函数，而是优先检查其他参数并从中进行推断。
如果跳过上下文敏感函数依然无法完成推断，类型推断就会继续按参数列表从左到右检查所有尚未检查的参数。
在紧邻的上述示例中，TypeScript 在推断 `T` 时会先跳过回调函数，转而查看第二个参数 `42`，并推断出 `T` 为 `number`。
随后，当回过头来检查回调函数时，回调函数便获得了 `(x: number) => void` 的上下文类型，从而也能推断出 `x` 同样是 `number`。

那么我们之前的例子中究竟发生了什么？

```ts
// 箭头函数语法 —— 无错误。
callIt({
  consume: (y) => y.toFixed(),
  produce: (x: number) => x * 2,
})

// 方法语法 —— 报错！
callIt({
  consume(y) {
    return y.toFixed()
  },
  //                  ~
  // 错误：'y' 的类型为 'unknown'。

  produce(x: number) {
    return x * 2
  },
})
```

在这两个示例中，`produce` 都被赋予了一个带有显式类型参数 `x` 的函数。
难道它们不应该以相同的方式被检查吗？

问题非常微妙：大多数函数（例如使用方法语法的函数）都具有隐式的 `this` 参数，但箭头函数则没有。
任何对 `this` 的使用都可能需要反向“拉扯” `T` 的类型——例如，了解外层对象字面量的类型可能会进而需要 `consume` 的类型，而后者又依赖于 `T`。

但我们的代码中根本没有使用 `this`！
诚然，该函数在运行时可能拥有一个 `this` 值，但它从未使用过！

TypeScript 6.0 在判断函数是否为上下文敏感函数时考虑到了这一点。
如果在函数中从未实际*使用* `this`，那么该函数就不会被视作上下文敏感函数。
这意味着在类型推断中，方法语法与箭头函数具有一致的推断顺序，这些无 `this` 函数（`this`-less functions）在类型推断时将被赋予更高优先级，统一了推断流与参数关系，实现了更强健的相互推断，我们上面所有的示例现在都能正常工作了！

[该改动由](https://github.com/microsoft/TypeScript/pull/62243) [Mateusz Burzyński](https://github.com/Andarist) 贡献完成。

## 以 `#/` 开头的子路径导入

当 Node.js 增加对模块的支持时，它添加了一项名为[“子路径导入”](https://nodejs.org/api/packages.html#subpath-imports)的功能。
这本质上是[一个名为 `imports` 的字段](https://nodejs.org/api/packages.html#imports)，允许 npm 包为其包内模块创建内部别名。

```json
{
  "name": "my-package",
  "type": "module",
  "imports": {
    "#root/*": "./dist/*"
  }
}
```

这使得 `my-package` 中的模块可以通过以 `#root/` 开头的路径进行导入：

```js
import * as utils from '#root/utils.js'
```

而无需使用如下所示的相对路径：

```js
import * as utils from '../../utils.js'
```

该特性的一个小缺憾是，开发者在指定子路径导入时，总是必须在 `#` 后面写上*某些字符*。
在上述示例中我们使用了 `root`，但这显得有些多余，因为除了 `./dist/` 之外我们并没有映射其他目录。

使用过打包工具的开发者也习惯于使用路径映射来避免冗长的相对路径。
打包工具中一个常见的约定是使用简单的 `@/` 作为前缀。
遗憾的是，以往子路径导入完全不能以 `#/` 开头，这给尝试在项目中采用该功能的开发者带来了不少困惑。

不过最近，[Node.js 增加了对以 `#/` 开头的子路径导入的支持](https://github.com/nodejs/node/pull/60864)。
这允许包在子路径导入中直接使用简洁的 `#/` 前缀，而无需添加多余的路径段。

```json
{
  "name": "my-package",
  "type": "module",
  "imports": {
    "#/*": "./dist/*"
  }
}
```

较新的 Node.js 20 版本已支持此语法，因此 TypeScript 现在也在 `--moduleResolution` 的 `nodenext` 和 `bundler` 选项下提供了对它的支持。

感谢 [magic-akari](https://github.com/magic-akari) 完成了这项工作，[实现的 Pull Request 请参见此处](https://github.com/microsoft/TypeScript/pull/62844)。

## 组合使用 `--moduleResolution bundler` 与 `--module commonjs`

TypeScript 的 `--moduleResolution bundler` 选项此前仅允许与 `--module esnext` 或 `--module preserve` 搭配使用；
然而，随着 `--moduleResolution node`（又称 `--moduleResolution node10`）被废弃，对于许多项目而言，这一新的组合往往是最适合的升级过渡路径。

项目通常需要根据项目类型（例如打包的 Web 应用、Bun 应用或 Node.js 应用），规划迁移到以下方案之一：

- `--module preserve` 搭配 `--moduleResolution bundler`
- `--module nodenext`

更多信息请参阅[实现该功能的 Pull Request](https://github.com/microsoft/TypeScript/pull/62320)。

## `--stableTypeOrdering` 标志

作为我们在 [TypeScript 原生重写](https://devblogs.microsoft.com/typescript/typescript-native-port/)方面持续推进工作的一部分，我们引入了一个名为 `--stableTypeOrdering` 的新标志，旨在协助从 6.0 迁移到 7.0。

目前，TypeScript 按照遇到类型的顺序为类型分配类型 ID（内部跟踪编号），并使用这些 ID 以一致的方式对联合类型进行排序。
属性的处理过程也类似。
因此，程序中声明各元素的顺序可能会对诸如声明文件生成等操作产生出人意料的影响。

例如，考虑此文件的声明文件输出：

```ts
// 输入：some-file.ts
export function foo(condition: boolean) {
  return condition ? 100 : 500
}

// 输出：some-file.d.ts
export declare function foo(condition: boolean): 100 | 500
//                                               ^^^^^^^^^
//             注意此联合类型的顺序：先 100，后 500。
```

如果在 `foo` _上方_ 添加一个无关的 `const`，生成的声明文件就会发生改变：

```ts
// 输入：some-file.ts
const x = 500
export function foo(condition: boolean) {
  return condition ? 100 : 500
}

// 输出：some-file.d.ts
export declare function foo(condition: boolean): 500 | 100
//                                               ^^^^^^^^^
//                           注意此处顺序的变化。
```

之所以会发生这种情况，是因为在分析 `const x` 声明时字面量类型 `500` 先被处理，从而获得了比 `100` 更小的类型 ID。
在极少数情况下，这种顺序变化甚至可能导致某些类型错误根据程序处理顺序的不同而出现或消失；但通常而言，你主要会在生成的声明文件或编辑器中展示类型的方式中注意到这种顺序差异。

TypeScript 7 的重大架构改进之一是并行类型检查，这极大地缩短了整体检查时间。
然而，并行化带来了一个挑战：当不同的类型检查器以不同的顺序访问节点、类型和符号时，分配给这些结构的内部 ID 将变得非确定性。
这进而会导致令人困惑的非确定性输出：同一个程序中内容完全相同的两个文件可能会生成不同的声明文件，甚至在分析同一个文件时计算出不同的错误。
为了解决这个问题，TypeScript 7.0 根据对象内容采用确定性算法对其内部对象（例如类型和符号）进行排序。
这确保了所有检查器无论在何时以及以何种方式创建对象，遇到的对象顺序都是完全相同的。
因此，在上述示例中，TypeScript 7 将_始终_输出 `100 | 500`，彻底消除了顺序的不稳定性。

这意味着 TypeScript 6 和 7 有时确实会显示不同的顺序。
尽管这些顺序变化绝大多数情况下是良性的，但如果你要在不同运行之间对比编译器输出（例如检查 6.0 与 7.0 生成的声明文件），这些不同的顺序会产生大量噪音，从而难以评估输出的正确性。
偶尔你还可能遇到因顺序变更而导致类型错误出现或消失的情况，这可能会更加令人困惑。

为了应对这一情况，在 6.0 中你可以指定新的 `--stableTypeOrdering` 标志。
该标志使 6.0 的类型排序行为与 7.0 保持一致，从而减少两个代码库之间的差异。
请注意，我们并不建议一直使用此标志，因为它会显著降低类型检查速度（视代码库而定，降幅最高可达 25%）。

如果你在使用 `--stableTypeOrdering` 时遇到了类型错误，这通常是由于推断差异导致的。
先前在未启用 `--stableTypeOrdering` 时的推断只是“恰好”基于程序中现有的类型顺序正常工作。
为了解决这一问题，在某些地方提供显式类型通常会很有帮助。
这通常可以是一个类型实参：

```diff
- someFunctionCall(/*...*/);
+ someFunctionCall<SomeExplicitType>(/*...*/);
```

或者是对准备传入调用的实参进行变量类型注解：

```diff
- const someVariable = { /*... some complex object ...*/ };
+ const someVariable: SomeExplicitType = { /*... some complex object ...*/ };

someFunctionCall(someVariable);
```

**请注意，该标志仅用于协助诊断 6.0 与 7.0 之间的差异——它并非设计为长期使用的功能特性。**

[更多详情请参阅此 Pull Request](https://github.com/microsoft/TypeScript/pull/63084)。

## `target` 和 `lib` 新增 `es2025` 选项

TypeScript 6.0 为 `target` 和 `lib` 配置项新增了对 `es2025` 选项的支持。
虽然 ES2025 中没有新的 JavaScript 语言语法特性，但该新 target 为内置 API 添加了新类型（例如 `RegExp.escape`），并将一些声明从 `esnext` 移动到了 `es2025` 中（例如 `Promise.try`、`Iterator` 方法以及 `Set` 方法）。
开启[该新 target](https://github.com/microsoft/TypeScript/pull/63046) 的工作由 [Kenta Moriuchi](https://github.com/petamoriken) 贡献。

## `Temporal` 的全新类型定义

万众期待的 [Temporal 提案](https://github.com/tc39/proposal-temporal) 已进入 Stage 4，并将成为未来 ECMAScript 标准的一部分。
TypeScript 6.0 现已内置了 Temporal API 的类型定义，因此你今天就可以通过 `--target esnext` 或 `"lib": ["esnext"]`（或更细粒度的 `esnext.temporal`）在 TypeScript 代码中使用它：

```ts
let yesterday = Temporal.Now.instant().subtract({
  hours: 24,
})

let tomorrow = Temporal.Now.instant().add({
  hours: 24,
})

console.log(`Yesterday: ${yesterday}`)
console.log(`Tomorrow: ${tomorrow}`)
```

Temporal 已经在多个运行时中可用，并且随着进入 Stage 4，它已正式成为 JavaScript 语言的一部分。
[关于 Temporal API 的文档可在 MDN 上查阅](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Temporal)。

[该项工作](https://github.com/microsoft/TypeScript/pull/62628) 由 GitHub 用户 [Renegade334](https://github.com/Renegade334) 贡献完成。

## “upsert” 方法（即 `getOrInsert`）的全新类型定义

在操作 `Map` 时，一种常见的模式是检查键是否存在，如果不存在，则设置并获取一个默认值：

```ts
function processOptions(compilerOptions: Map<string, unknown>) {
  let strictValue: unknown
  if (compilerOptions.has('strict')) {
    strictValue = compilerOptions.get('strict')
  } else {
    strictValue = true
    compilerOptions.set('strict', strictValue)
  }
  // ...
}
```

这种模式写起来往往较为繁琐。
[ECMAScript 的“upsert”提案](https://github.com/tc39/proposal-upsert)最近已进入 Stage 4，并在 `Map` 和 `WeakMap` 上引入了两个新方法：

- `getOrInsert`
- `getOrInsertComputed`

这些方法已被添加到 `esnext` lib 中，以便你可以在 TypeScript 6.0 中立即使用它们。

借助 `getOrInsert`，我们可以将上述代码替换为：

```ts
function processOptions(compilerOptions: Map<string, unknown>) {
  let strictValue = compilerOptions.getOrInsert('strict', true)
  // ...
}
```

`getOrInsertComputed` 的工作原理类似，但适用于计算默认值开销较大（例如需要大量计算、内存分配或执行耗时的同步 I/O）的情况。
相反，它接受一个回调函数，该回调仅在键尚不存在时才会被调用：

```ts
someMap.getOrInsertComputed('someKey', () => {
  return computeSomeExpensiveValue(/*...*/)
})
```

该回调还会接收键作为参数，这在默认值基于键生成时非常有用：

```ts
someMap.getOrInsertComputed(someKey, computeSomeExpensiveDefaultValue)

function computeSomeExpensiveValue(key: string) {
  // ...
}
```

[该项更新](https://github.com/microsoft/TypeScript/pull/62612) 由 GitHub 用户 [Renegade334](https://github.com/Renegade334) 贡献。

## `RegExp.escape`

当构造要在正则表达式中匹配的字面量字符串时，转义正则表达式的特殊字符（如 `*`、`+`、`?`、`(`、`)` 等）至关重要。
[ECMAScript 的 RegExp Escaping 提案](https://github.com/tc39/proposal-regex-escaping) 已达到 Stage 4，并引入了一个全新的 `RegExp.escape` 函数来替你处理此转义工作：

```ts
function matchWholeWord(word: string, text: string) {
  const escapedWord = RegExp.escape(word)
  const regex = new RegExp(`\\b${escapedWord}\\b`, 'g')
  return text.match(regex)
}
```

`RegExp.escape` 在 `es2025` lib 中提供，因此你现在就可以在 TypeScript 6.0 中使用它。

[该项工作](https://github.com/microsoft/TypeScript/pull/63046) 由 [Kenta Moriuchi](https://github.com/petamoriken) 贡献。

## `dom` 库现已包含 `dom.iterable` 与 `dom.asynciterable`

TypeScript 的 `lib` 选项允许你指定目标运行时所具备的全局声明。
其中一个选项是 `dom`，用于表示 Web 环境（即实现了 [DOM API](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model) 的浏览器）。
此前，为了照顾不支持 `Iterable` 和 `AsyncIterable` 的环境，DOM API 的部分内容被单独拆分到了 `dom.iterable` 和 `dom.asynciterable` 中。
这意味着你必须显式添加 `dom.iterable`，才能在 `NodeList` 或 `HTMLCollection` 等 DOM 集合上使用迭代方法。

在 TypeScript 6.0 中，`lib.dom.iterable.d.ts` 和 `lib.dom.asynciterable.d.ts` 的内容已全部合并到 `lib.dom.d.ts` 中。
你仍然可以在配置文件的 `"lib"` 数组中引用 `dom.iterable` 和 `dom.asynciterable`，但它们现在只是空文件。

```ts
// 在 TypeScript 6.0 之前，这需要 "lib": ["dom", "dom.iterable"]
// 现在仅需 "lib": ["dom"] 即可工作
for (const element of document.querySelectorAll('div')) {
  console.log(element.textContent)
}
```

这是一项体验上的改进，消除了一个常见的困惑点，因为目前所有主流现代浏览器都支持这些能力。
如果你之前同时配置了 `dom` 和 `dom.iterable`，现在可以简化为仅保留 `dom`。

更多内容请参阅[此议题](https://github.com/microsoft/TypeScript/issues/60959)及其[对应的 Pull Request](https://github.com/microsoft/TypeScript/pull/62111)。

## TypeScript 6.0 中的破坏性变更与弃用

TypeScript 6.0 是一个重要的过渡版本，旨在帮助开发者为即将推出的 TypeScript 编译器原生重写版本——TypeScript 7.0 做好准备。
尽管 TypeScript 6.0 保持了与现有 TypeScript 知识的完全一致，并继续与 TypeScript 5.9 保持 API 兼容，但该版本引入了多项破坏性变更和弃用，以反映 JavaScript 生态系统的演进，并为 TypeScript 7.0 铺平道路。

在 TypeScript 5.0 发布的两年间，我们见证了开发者编写和分发 JavaScript 方式的持续转变：

- 几乎所有的运行时环境现已成为“常青”环境。真正的传统老旧环境（ES5）已极其罕见。
- 打包工具和 ESM 已成为新项目中最主流的模块目标，尽管 CommonJS 仍是一个重要的目标。相比 2012 年，AMD 和其他浏览器端用户态模块系统已鲜为人用。
- 几乎所有的 npm 包都可以通过某种模块系统来加载。UMD 包依然存在，但几乎没有新代码会*仅*作为全局变量提供。
- `tsconfig.json` 作为配置机制已近乎普及。
- 对“更严格”类型检查的需求持续增长。
- TypeScript 构建性能备受关注。尽管 TypeScript 7 带来了巨大的性能提升，但性能始终是一个核心目标，无法以高性能方式支持的选项需要有更充分的存在理由。

因此，TypeScript 6.0 和 7.0 的设计充分考虑了这些现实情况。
对于 TypeScript 6.0，可以在 tsconfig 中设置 `"ignoreDeprecations": "6.0"` 来忽略这些弃用提示；然而请注意，TypeScript 7.0 _将不再支持_任何这些已弃用的选项。

某些必要的代码调整可以通过 codemod 或工具自动完成。
例如，[实验性工具 `ts5to6`](https://github.com/andrewbranch/ts5to6) 可以自动在你的代码库中调整 `baseUrl` 和 `rootDir`。

### 前期调整要点

我们将在下文中介绍各项具体的调整，但必须指出的是，某些弃用和行为变更并不一定会产生直接指向根本原因的错误信息。
因此，我们在此预先强调，**许多项目将需要至少执行以下操作之一**：

- 在 tsconfig 中显式设置 `"types"` 数组，通常为 `"types": ["node"]`。

  `"types": ["*"]` 可以恢复 5.9 的行为，但我们建议使用显式数组以提高构建性能和可预测性。

  如果你看到大量与缺失标识符或未解析内置模块相关的类型错误，通常表明这就是问题所在。

- 如果你先前依赖自动推断，请显式设置 `"rootDir": "./src"`。

  如果你发现输出文件被写入到 `./dist/src/index.js` 而非 `./dist/index.js`，通常就是这个原因所致。

### 简单的默认值变更

若干编译器选项的默认值现已更新，以更好地契合现代开发实践：

- **`strict` 现在默认设为 `true`**：
  对更严格类型检查的需求持续增长，我们发现大多数新项目都希望启用 `strict` 模式。
  如果你已经在使用 `"strict": true`，则不受任何影响。
  如果你之前依赖默认的 `false`，则需要在 `tsconfig.json` 中显式设置 `"strict": false`。

- **`module` 默认值变更为 `esnext`**：
  同样，新的默认 `module` 为 `esnext`，这是对 ESM 已成为主导模块格式的认可。

- **`target` 默认值变更为当年 ES 版本**：
  新的默认 `target` 是当前支持的最新 ECMAScript 规范版本（实际上是一个浮动目标）。
  目前，该目标为 `es2025`。
  这反映了绝大多数开发者都在面向常青运行时开发，无需编译降级到旧版 ECMAScript 的现实。

- **`noUncheckedSideEffectImports` 现在默认设为 `true`**：
  这有助于捕获仅产生副作用的导入中的拼写错误。

- **`libReplacement` 现在默认设为 `false`**：
  该标志此前在每次运行时都会导致大量的模块解析失败尝试，进而增加了在 `--watch` 和编辑器场景下需要监听的路径数量。
  在一个新项目中，除非进行了其他显式配置，否则 `libReplacement` 不会起任何作用，因此为了默认获得更佳的性能，将其默认关闭是合情合理的。

如果这些新的默认值破坏了你的项目，你可以在 `tsconfig.json` 中显式指定之前的旧值。

### `rootDir` 现在默认设为 `.`

`rootDir` 用于控制输出文件相对于输出目录的目录结构。
此前，如果未指定 `rootDir`，它会根据所有非声明输入文件的共同公共目录自动推断。
但这往往意味着：如果不尝试加载并解析整个项目，就无法得知某个文件是否属于该项目。
这也意味着 TypeScript 必须分析程序中的每个文件路径，花费更多时间来推断该公共源码目录。

在 TypeScript 6.0 中，默认的 `rootDir` 将始终是包含 `tsconfig.json` 文件的目录。
只有在没有 `tsconfig.json` 文件的情况下直接从命令行运行 `tsc` 时，才会对 `rootDir` 进行推断。

如果你的源码文件位于比 `tsconfig.json` 目录更深的层级，并且之前依赖 TypeScript 自动推断源码的公共根目录，则需要显式设置 `rootDir`：

```diff
  {
      "compilerOptions": {
          // ...
+         "rootDir": "./src"
      },
      "include": ["./src"]
  }
```

同样，如果你的 `tsconfig.json` 引用了该配置文件所在目录之外的文件，你也需要调整 `rootDir` 以包含这些文件：

```diff
  {
      "compilerOptions": {
          // ...
+         "rootDir": "../src"
      },
      "include": ["../src/**/*.tests.ts"]
  }
```

更多详情请参阅[此处的讨论](https://github.com/microsoft/TypeScript/issues/62194)以及[此处的具体实现](https://github.com/microsoft/TypeScript/pull/62418)。

### `types` 现在默认设为 `[]`

在 `tsconfig.json` 中，`compilerOptions` 的 `types` 字段用于指定在编译期间包含在全局作用域中的包名称列表。
通常，`node_modules` 中的包会通过源代码中的 `import` 语句自动引入；
但为了方便起见，TypeScript 以前默认还会自动包含 `node_modules/@types` 下的所有包，以便你无需显式导入即可使用来自 `@types/node` 的 `process` 全局变量或 `"fs"` 模块，或者来自 `@types/jest` 的 `describe` 和 `it`。

从某种意义上说，以往 `types` 的默认行为就是“遍历并包含 `node_modules/@types` 下的一切内容”。
这可能带来*极其高昂*的性能开销，因为如今一般的代码仓库可能会间接引入数百个 `@types` 包，尤其是在具有拍平 `node_modules` 的多项目工作区中。
现代项目几乎通常只需要 `@types/node`、`@types/jest` 或少数其他影响全局的常见包。

在 TypeScript 6.0 中，`types` 的默认值将为 `[]`（空数组）。
这一改变避免了项目在构建时不经意间引入数百甚至数千个不必要的声明文件。
我们调研过的许多项目仅通过合理配置 `types`，构建时间就缩短了 20% 到 50% 不等。

**这将对许多项目产生影响。** 你很可能需要添加 `"types": ["node"]` 或其他少数包：

```diff
  {
      "compilerOptions": {
          // 显式列出你需要的 @types 包
+         "types": ["node", "jest"]
      }
  }
```

你也可以指定一个 `*` 条目来重新启用以往的自动遍历行为：

```diff
  {
      "compilerOptions": {
          // 加载所有类型 —— TypeScript 5.9 及更早版本的默认行为。
+         "types": ["*"]
      }
  }
```

如果你遇到了如下所示的全新错误信息：

```
Cannot find module '...' or its corresponding type declarations.
Cannot find name 'fs'. Do you need to install type definitions for node? Try `npm i --save-dev @types/node` and then add 'node' to the types field in your tsconfig.
Cannot find name 'path'. Do you need to install type definitions for node? Try `npm i --save-dev @types/node` and then add 'node' to the types field in your tsconfig.
Cannot find name 'process'. Do you need to install type definitions for node? Try `npm i --save-dev @types/node` and then add 'node' to the types field in your tsconfig.
Cannot find name 'Bun'. Do you need to install type definitions for Bun? Try `npm i --save-dev @types/bun` and then add 'bun' to the types field in your tsconfig.
Cannot find name 'describe'. Do you need to install type definitions for a test runner? Try `npm i --save-dev @types/jest` or `npm i --save-dev @types/mocha` and then add 'jest' or 'mocha' to the types field in your tsconfig.
```

很可能是因为你需要在 `types` 字段中添加相应的条目。

更多详情请参阅[此提案](https://github.com/microsoft/TypeScript/issues/62195)以及[实现的 Pull Request](https://github.com/microsoft/TypeScript/pull/63054)。

### 已弃用：`target: es5`

ECMAScript 5 编译目标长期以来在支持老旧浏览器方面扮演着重要角色；但其后继者 ECMAScript 2015 (ES6) 发布已超过十年，所有现代浏览器也已全面支持多年。
随着 Internet Explorer 的退役以及常青浏览器的全面普及，如今输出 ES5 的使用场景已微乎其微。

TypeScript 支持的最低 target 现在将提升为 ES2015，且 `target: es5` 选项已被弃用。如果你之前使用的是 `target: es5`，你需要迁移到更高版本的 target 或使用外部编译器。
如果你仍然需要 ES5 输出，我们建议使用外部编译器直接编译 TypeScript 源码，或者对 TypeScript 的输出进行后处理转换。

[参见此处了解有关此项弃用的更多信息](https://github.com/microsoft/TypeScript/issues/62196)以及[其实现的 Pull Request](https://github.com/microsoft/TypeScript/pull/63067)。

### 已弃用：`--downlevelIteration`

`--downlevelIteration` 仅对 ES5 代码输出生效，而由于 `--target es5` 已被弃用，`--downlevelIteration` 不再具有任何实际作用。

微妙的是，在 TypeScript 5.9 及更早版本中，即使 `--downlevelIteration false` 与 `--target es2015` 一起使用没有任何效果，也不会报错。
而在 TypeScript 6.0 中，只要配置了 `--downlevelIteration` 就会引发弃用错误。

请参见[此处的具体实现](https://github.com/microsoft/TypeScript/pull/63071)。

### 已弃用：`--moduleResolution node`（即 `--moduleResolution node10`）

`--moduleResolution node` 固化了 Node.js 模块解析算法的一个特定版本，该算法最准确地反映的是 Node.js 10 的行为。
遗憾的是，该选项（及其名称）忽略了自那时起 Node.js 解析算法的大量更新，无法再很好地代表现代 Node.js 版本的行为。

在 TypeScript 6.0 中，`--moduleResolution node`（具体即 `--moduleResolution node10`）已被弃用。
之前使用 `--moduleResolution node` 的用户，如果打算直接面向 Node.js，通常应当迁移到 `--moduleResolution nodenext`；如果打算使用打包工具或 Bun，则应当迁移到 `--moduleResolution bundler`。

更多内容请参阅[此议题](https://github.com/microsoft/TypeScript/issues/62200)及其[对应的 Pull Request](https://github.com/microsoft/TypeScript/pull/62338)。

### 已弃用：`module` 的 `amd`、`umd` 和 `systemjs` 选项值

以下标志值已不再受支持：

- `--module amd`
- `--module umd`
- `--module systemjs`
- `--module none`

在浏览器缺乏原生模块支持的早期阶段，AMD、UMD 和 SystemJS 曾发挥了重要作用。
而 "none" 的语义从未明确界定，常常引起困惑。
如今，ESM 已在浏览器和 Node.js 中获得普遍支持，而 import maps 和打包工具也已成为填补空白的首选方式。
如果你仍以这些模块系统为目标，请考虑迁移到合适的 ECMAScript 模块输出目标、采用打包工具或其他编译器，或者在能够完成迁移前暂时停留在 TypeScript 5.x。

这也意味着废弃对 `amd-module` 指令的支持，该指令将不再产生任何效果。

更多内容请参阅[提案议题](https://github.com/microsoft/TypeScript/issues/62199)以及[实现的 Pull Request](https://github.com/microsoft/TypeScript/pull/62669)。

### 已弃用：`--baseUrl`

`baseUrl` 选项最常与 `paths` 配合使用，通常作为 `paths` 中各个路径值的前缀。
不幸的是，`baseUrl` 在模块解析时也会被当作查找根目录。

例如，在如下 `tsconfig.json` 中：

```json5
{
  compilerOptions: {
    // ...
    baseUrl: './src',
    paths: {
      '@app/*': ['app/*'],
      '@lib/*': ['lib/*'],
    },
  },
}
```

以及如下的导入语句：

```ts
import * as someModule from 'someModule.js'
```

TypeScript 可能会将其解析为 `src/someModule.js`，即使开发者的初衷只是为以 `@app/` 和 `@lib/` 开头的模块添加路径映射。

在最好的情况下，这往往会导致生成打包工具无法识别的“劣质”路径；
而在更糟糕的情况下，许多在运行时根本无法工作的导入路径却会被 TypeScript 误认为“完全有效”。

很长一段时间以来，`paths` 映射已经不再强制要求指定 `baseUrl`，在实践中，绝大多数使用 `baseUrl` 的项目只是将其作为 `paths` 各项的前缀。
在 TypeScript 6.0 中，`baseUrl` 已被弃用，并且将不再作为模块解析的查找根目录。

将 `baseUrl` 用作路径映射条目前缀的开发者，只需直接移除 `baseUrl` 并将该前缀添加到各自的 `paths` 条目中即可：

```diff json5
  {
    "compilerOptions": {
      // ...
-     "baseUrl": "./src",
      "paths": {
-       "@app/*": ["app/*"],
-       "@lib/*": ["lib/*"]
+       "@app/*": ["./src/app/*"],
+       "@lib/*": ["./src/lib/*"]
      }
    }
  }
```

而如果确实有开发者*曾经*将 `baseUrl` 作为查找根目录使用，也可以添加显式的路径映射来保留旧有行为：

```json5
{
  compilerOptions: {
    // ...
    paths: {
      // 替代 baseUrl 的新通用匹配：
      '*': ['./src/*'],

      // 其他所有路径现在都具有显式的通用前缀：
      '@app/*': ['./src/app/*'],
      '@lib/*': ['./src/lib/*'],
    },
  },
}
```

然而这种情况极为罕见。
我们建议绝大多数开发者直接移除 `baseUrl`，并在 `paths` 条目中加上相应的前缀。

更多内容请参阅[此议题](https://github.com/microsoft/TypeScript/issues/62207)以及[对应的 Pull Request](https://github.com/microsoft/TypeScript/pull/62509)。

### 已弃用：`--moduleResolution classic`

`moduleResolution: classic` 设置已被移除。
`classic` 解析策略是 TypeScript 最初的模块解析算法，其诞生甚至早于 Node.js 解析算法成为事实标准之前。
如今，所有的实际使用场景均由 `nodenext` 或 `bundler` 满足。
如果你还在使用 `classic`，请迁移到这些现代解析策略之一。

更多内容请参阅[此议题](https://github.com/microsoft/TypeScript/issues/62206)以及[实现的 Pull Request](https://github.com/microsoft/TypeScript/pull/62669)。

### 已弃用：`--esModuleInterop false` 与 `--allowSyntheticDefaultImports false`

以下配置项不能再设置为 `false`：

- `esModuleInterop`
- `allowSyntheticDefaultImports`

`esModuleInterop` 和 `allowSyntheticDefaultImports` 最初设计为可选开启，以避免破坏现有项目。
然而多年以来，它们所启用的行为一直是被推荐的默认做法。
将它们设置为 `false` 往往会导致在 ESM 中加载 CommonJS 模块时出现隐蔽的运行时问题。
在 TypeScript 6.0 中，更安全的互操作行为将始终处于启用状态。

如果你的代码中存在依赖旧行为的导入，可能需要进行调整：

```ts
// 之前（esModuleInterop: false）
import * as express from 'express'

// 之后（始终启用 esModuleInterop）
import express from 'express'
```

更多内容请参阅[此议题](https://github.com/microsoft/TypeScript/issues/62529)及其[实现的 Pull Request](https://github.com/microsoft/TypeScript/pull/62567)。

### 已弃用：`--alwaysStrict false`

`alwaysStrict` 标志用于控制推断并输出 `"use strict";` 指令。
在 TypeScript 6.0 中，所有代码都将被假定处于 [JavaScript 严格模式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode) 下，这套 JS 语义最明显的影响在于涉及保留字的语法边界情况。
如果你的代码处于“非严格模式”并将 `await`、`static`、`private` 或 `public` 等保留字用作常规标识符，你需要对其进行重命名。
如果你依赖非严格模式下关于 `this` 含义的微妙语义，你可能也需要调整代码。

更多内容请参阅[此议题](https://github.com/microsoft/TypeScript/issues/62213)及其[对应的 Pull Request](https://github.com/microsoft/TypeScript/pull/63089)。

### 已弃用：`outFile`

`--outFile` 选项已在 TypeScript 6.0 中被移除。该选项最初设计用于将多个输入文件合并为一个输出文件。然而，Webpack、Rollup、esbuild、Vite、Parcel 等外部打包工具现在能以更快、更好且配置更丰富的方式完成此任务。移除此选项简化了编译器内部实现，使我们能够专注于 TypeScript 最擅长的领域：类型检查和声明文件生成。如果你当前正在使用 `--outFile`，需要迁移到外部打包工具。大多数现代打包工具都提供了开箱即用的出色 TypeScript 支持。

### 已弃用：用于命名空间的旧版 `module` 语法

早期版本的 TypeScript 使用 `module` 关键字来声明命名空间：

```ts
// ❌ 已弃用语法 —— 现在报错
module Foo {
  export const bar = 10
}
```

该语法后来被更名为现代推荐的形式——使用 `namespace` 关键字：

```ts
// ✅ 正确语法
namespace Foo {
  export const bar = 10
}
```

当引入 `namespace` 时，`module` 语法仅被列为不推荐使用。
几年前，TypeScript 语言服务开始将该关键字标记为已弃用，并建议改用 `namespace`。

在 TypeScript 6.0 中，在期望使用 `namespace` 的地方使用 `module` 现已成为硬性弃用。
这项变更是必要的，因为 ECMAScript 存在一个潜在的 `module` 代码块提案，该提案会与旧版 TypeScript 语法产生冲突。

外部环境模块声明形式仍然得到完全支持：

```ts
// ✅ 仍然完全正常工作
declare module 'some-module' {
  export function doSomething(): void
}
```

有关更多详细信息，请参见[此议题](https://github.com/microsoft/TypeScript/issues/62211)及其[对应的 Pull Request](https://github.com/microsoft/TypeScript/pull/62876)。

### 已弃用：导入语句中的 `asserts` 关键字

`asserts` 关键字曾通过导入断言提案被提议加入 JavaScript 语言；
然而，该提案最终演变为[导入属性提案](https://github.com/tc39/proposal-import-attributes)，改用 `with` 关键字代替了 `asserts`。

因此，`asserts` 语法在 TypeScript 6.0 中现已被弃用，使用它将导致报错：

```ts
// ❌ 已弃用语法 —— 现在报错。
import blob from "./blahb.json" asserts { type: "json" }
//                              ~~~~~~~
// 错误：导入断言已被导入属性取代。请使用 'with' 代替 'asserts'。
```

相反，请改用导入属性的 `with` 语法：

```ts
// ✅ 适用于新的导入属性语法。
import blob from './blahb.json' with { type: 'json' }
```

更多内容请参阅[此议题](https://github.com/microsoft/TypeScript/issues/62210)及其[对应的 Pull Request](https://github.com/microsoft/TypeScript/pull/63077)。

### 已弃用：`no-default-lib` 指令

`/// <reference no-default-lib="true"/>` 指令在很大程度上被误解和误用。
在 TypeScript 6.0 中，该指令已不再受支持。
如果你之前正在使用它，请考虑改用 `--noLib` 或 `--libReplacement`。

[在此查看更多信息](https://github.com/microsoft/TypeScript/issues/62209)以及[对应的 Pull Request](https://github.com/microsoft/TypeScript/pull/62435)。

### 存在 `tsconfig.json` 时在命令行指定输入文件现在会报错

目前，如果你在存在 `tsconfig.json` 的目录中运行 `tsc foo.ts`，该配置文件将被完全忽略。
如果你期望类型检查和生成选项应用于该输入文件，这往往会让人感到非常困惑。

在 TypeScript 6.0 中，如果你在包含 `tsconfig.json` 的目录中附带文件参数运行 `tsc`，将会产生错误以明确提示该行为：

```
error TS5112: tsconfig.json is present but will not be loaded if files are specified on commandline. Use '--ignoreConfig' to skip this error.
```

如果你确实希望忽略 `tsconfig.json` 并使用 TypeScript 的默认配置直接编译 `foo.ts`，可以使用全新的 `--ignoreConfig` 标志：

```sh
tsc --ignoreConfig foo.ts
```

更多内容请参阅[此议题](https://github.com/microsoft/TypeScript/issues/62197)及其[对应的 Pull Request](https://github.com/microsoft/TypeScript/pull/62477)。

## 为 TypeScript 7.0 做好准备

TypeScript 6.0 被定位为一个过渡版本。
虽然在设置了 `"ignoreDeprecations": "6.0"` 的情况下，TypeScript 6.0 中弃用的选项仍可正常运行而不报错，但这些选项将在 **TypeScript 7.0（TypeScript 原生重写版本）中被彻底移除**。
如果你在升级到 TypeScript 6.0 后看到了弃用警告，我们强烈建议在项目采用 TypeScript 7.0（或尝试 [native 预览版](https://www.npmjs.com/package/@typescript/native-preview)）之前着手解决它们。
