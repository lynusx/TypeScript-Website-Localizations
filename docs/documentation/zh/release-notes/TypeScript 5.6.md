---
title: TypeScript 5.6
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-5-6.html
oneline: TypeScript 5.6 发布说明
---

## 禁用无效的空值与真值检查

你可能曾写过一个正则表达式，却忘了调用其 `.test(...)` 方法：

```ts
if (/0x[0-9a-f]/) {
  // 糟糕！此代码块始终会执行。
  // ...
}
```

或者不小心将大于等于运算符 `>=` 误写成了箭头函数 `=>`：

```ts
if ((x) => 0) {
  // 糟糕！此代码块始终会执行。
  // ...
}
```

或者你想用 `??` 提供默认值，却混淆了 `??` 与诸如 `<` 这类比较运算符的优先级：

```ts
function isValid(
  value: string | number,
  options: any,
  strictness: 'strict' | 'loose',
) {
  if (strictness === 'loose') {
    value = +value
  }
  return value < options.max ?? 100
  // 糟糕！这会被解析为 (value < options.max) ?? 100
}
```

又或者在复杂的表达式中把括号放错了位置：

```ts
if (
  isValid(primaryValue, 'strict') ||
  isValid(secondaryValue, 'strict') ||
  isValid(primaryValue, 'loose' || isValid(secondaryValue, 'loose'))
) {
  //                           ^^^^ 👀 Did we forget a closing ')'?
}
```

上述示例无一符合开发者的初衷，但它们全都是合法的 JavaScript 代码。
在此之前，TypeScript 也会静默接受这些写法。

然而经过一些调研与尝试，我们发现标记出上述这类可疑代码可以捕获非常多的 Bug。
在 TypeScript 5.6 中，当编译器能在语法层面上判定真值（truthy）或空值（nullish）检查的结果始终固定时，就会直接报错。
因此，在上述示例中你将开始看到错误提示：

```ts
if (/0x[0-9a-f]/) {
  //  ~~~~~~~~~~~~
  // 错误：此类表达式始终为真值（truthy）。
}

if ((x) => 0) {
  //  ~~~~~~
  // 错误：此类表达式始终为真值（truthy）。
}

function isValid(
  value: string | number,
  options: any,
  strictness: 'strict' | 'loose',
) {
  if (strictness === 'loose') {
    value = +value
  }
  return value < options.max ?? 100
  //     ~~~~~~~~~~~~~~~~~~~
  // 错误：?? 的右操作数不可达，因为左操作数绝不为空值（nullish）。
}

if (
  isValid(primaryValue, 'strict') ||
  isValid(secondaryValue, 'strict') ||
  isValid(primaryValue, 'loose' || isValid(secondaryValue, 'loose'))
) {
  //                    ~~~~~~~
  // 错误：此类表达式始终为真值（truthy）。
}
```

通过启用 ESLint 的 `no-constant-binary-expression` 规则也能达到类似效果，你可以[在他们的博文中查看其捕获到的问题示例](https://eslint.org/blog/2022/07/interesting-bugs-caught-by-no-constant-binary-expression/)；
不过 TypeScript 引入的新检查与 ESLint 规则并不完全重叠，而且我们相信将这些检查直接内置到 TypeScript 中具有巨大价值。

需要说明的是，某些特定表达式即使始终为真或为空值，依然是被允许的。
具体而言，`true`、`false`、`0` 和 `1` 尽管恒为真值或假值，但仍旧被允许使用，因为类似下面的代码：

```ts
while (true) {
  doStuff()

  if (something()) {
    break
  }

  doOtherStuff()
}
```

依旧是符合习惯且非常实用的，而像下面这样的代码：

```ts
if (true || inDebuggingOrDevelopmentEnvironment()) {
  // ...
}
```

在代码迭代与调试过程中也同样很有用。

如果你对该功能的具体实现或它能捕获的 Bug 类型感兴趣，可以查看[实现该特性的 Pull Request](https://github.com/microsoft/TypeScript/pull/59217)。

## 迭代器帮助方法

JavaScript 中有*可迭代对象*（iterable，即可以通过调用 `[Symbol.iterator]()` 获取迭代器的对象）和*迭代器*（iterator，即包含 `next()` 方法、在迭代过程中可调用其获取下一个值的对象）的概念。
大体上，当你把它们直接传入 `for`/`of` 循环，或通过 `[...spread]` 展开到新数组中时，通常不需要去思考这些细节。
但 TypeScript 通过 `Iterable` 和 `Iterator` 类型（甚至是兼具二者特性的 `IterableIterator`）对它们进行了建模，这些类型描述了让 `for`/`of` 等语法结构正常工作所需的最少成员集合。

`Iterable`（以及 `IterableIterator`）非常灵活，可以用于 JavaScript 的各种场景——但许多开发者发现它们缺少 `Array` 上那些便捷的方法，如 `map`、`filter`，甚至 `reduce`。
正因如此，[ECMAScript 最近提出了一项新提案](https://github.com/tc39/proposal-iterator-helpers)，将 `Array` 的众多方法（以及更多新增方法）引入到 JavaScript 生成的大多数 `IterableIterator` 上。

例如，现在每个生成器（generator）生成的对象都拥有了 `map` 和 `take` 方法：

```ts
function* positiveIntegers() {
  let i = 1
  while (true) {
    yield i
    i++
  }
}

const evenNumbers = positiveIntegers().map((x) => x * 2)

// 输出：
//    2
//    4
//    6
//    8
//   10
for (const value of evenNumbers.take(5)) {
  console.log(value)
}
```

`Map` 和 `Set` 上的 `keys()`、`values()` 和 `entries()` 等方法所返回的对象同样如此：

```ts
function invertKeysAndValues<K, V>(map: Map<K, V>): Map<V, K> {
  return new Map(map.entries().map(([k, v]) => [v, k]))
}
```

你还可以继承新的 `Iterator` 对象：

```ts
/**
 * Provides an endless stream of `0`s.
 */
class Zeroes extends Iterator<number> {
  next() {
    return { value: 0, done: false } as const
  }
}

const zeroes = new Zeroes()

// 转换为无限连续的 `1` 流。
const ones = zeroes.map((x) => x + 1)
```

并且你可以通过 `Iterator.from` 将现有的任何 `Iterable` 或 `Iterator` 包装为这种新类型：

```ts
Iterator.from(...).filter(someFunction);
```

现在，我们必须讨论一下命名问题。

前面我们提到过，TypeScript 本身就拥有 `Iterable` 和 `Iterator` 类型；
但正如前文所述，它们更像是某种“协议”，用于确保特定操作能够顺利执行。
_这意味着，在 TypeScript 中被声明为 `Iterable` 或 `Iterator` 的值，并不一定都拥有上面提到的那些辅助方法。_

然而，运行时现在确实存在一个名为 `Iterator` 的**运行时实体**。
你可以在 JavaScript 中将 `Iterator` 以及 `Iterator.prototype` 作为实际的值来引用。
这带来了一丝尴尬，因为 TypeScript 此前已经为纯类型检查定义了名为 `Iterator` 的类型。
由于这个不巧的名称冲突，TypeScript 需要引入一个独立的类型来描述这些原生的/内置的可迭代迭代器（iterable iterator）。

TypeScript 5.6 引入了一个名为 `IteratorObject` 的新类型，其定义如下：

```ts
interface IteratorObject<
  T,
  TReturn = unknown,
  TNext = unknown,
> extends Iterator<T, TReturn, TNext> {
  [Symbol.iterator](): IteratorObject<T, TReturn, TNext>
}
```

许多内置集合与方法都会生成 `IteratorObject` 的子类型（例如 `ArrayIterator`、`SetIterator`、`MapIterator` 等），并且 `lib.d.ts` 中的核心 JavaScript 与 DOM 类型，以及 `@types/node`，均已更新以使用该新类型。

类似地，为了保持对称，还提供了 `AsyncIteratorObject` 类型。
虽然在 JavaScript 中 `AsyncIterator` 作为一个为 `AsyncIterable` 提供同等方法的运行时值尚未正式落地，[但这属于正在推进中的提案](https://github.com/tc39/proposal-async-iterator-helpers)，新类型已经提前为其做好了准备。

感谢 [Kevin Gibbons](https://github.com/bakkot) 贡献了[这些类型的改动](https://github.com/microsoft/TypeScript/pull/58222)，他也是[该提案](https://github.com/tc39/proposal-iterator-helpers)的共同作者之一。

## 严格的内置迭代器检查（与 `--strictBuiltinIteratorReturn`）

当你在 `Iterator<T, TReturn>` 上调用 `next()` 方法时，它会返回一个包含 `value` 和 `done` 属性的对象。
这在类型系统中通过 `IteratorResult` 进行建模：

```ts
type IteratorResult<T, TReturn = any> =
  IteratorYieldResult<T> | IteratorReturnResult<TReturn>

interface IteratorYieldResult<TYield> {
  done?: false
  value: TYield
}

interface IteratorReturnResult<TReturn> {
  done: true
  value: TReturn
}
```

此处的命名灵感来源于生成器函数的工作方式。
生成器函数可以 `yield` 产出值，并在最后 `return` 返回一个最终值——但二者的类型可以毫无关联：

```ts
function abc123() {
  yield 'a'
  yield 'b'
  yield 'c'
  return 123
}

const iter = abc123()

iter.next() // { value: "a", done: false }
iter.next() // { value: "b", done: false }
iter.next() // { value: "c", done: false }
iter.next() // { value: 123, done: true }
```

随着新的 `IteratorObject` 类型的引入，我们发现在安全实现 `IteratorObject` 方面存在一些困难。
与此同时，在 `TReturn` 为 `any`（默认值！）的情况下，`IteratorResult` 一直存在长期以来的类型不安全性。
例如，假设我们有一个 `IteratorResult<string, any>`。
如果我们直接访问该类型的 `value`，最终得到的类型将是 `string | any`，也就是 `any`：

```ts
function* uppercase(iter: Iterator<string, any>) {
  while (true) {
    const { value, done } = iter.next()
    yield value.toUppercase() // 糟糕！忘记先检查 `done`，且拼错了 `toUpperCase`

    if (done) {
      return
    }
  }
}
```

要在如今现有的每一个 `Iterator` 上修正该问题而不引发大量的破坏性变更，几乎是不可能的；但我们至少可以在创建大多数 `IteratorObject` 时解决它。

TypeScript 5.6 引入了一个名为 `BuiltinIteratorReturn` 的全新内置固有类型，以及一个新的 `--strict` 模式标志 `--strictBuiltinIteratorReturn`。
无论在 `lib.d.ts` 等何处使用 `IteratorObject`，它们的 `TReturn` 都会标注为 `BuiltinIteratorReturn` 类型（尽管你更常看到的会是更具体的 `MapIterator`、`ArrayIterator`、`SetIterator` 等）：

```ts
interface MapIterator<T> extends IteratorObject<
  T,
  BuiltinIteratorReturn,
  unknown
> {
  [Symbol.iterator](): MapIterator<T>
}

// ...

interface Map<K, V> {
  // ...

  /**
   * Returns an iterable of key, value pairs for every entry in the map.
   */
  entries(): MapIterator<[K, V]>

  /**
   * Returns an iterable of keys in the map
   */
  keys(): MapIterator<K>

  /**
   * Returns an iterable of values in the map
   */
  values(): MapIterator<V>
}
```

默认情况下，`BuiltinIteratorReturn` 为 `any`；但当启用了 `--strictBuiltinIteratorReturn`（包括通过 `--strict` 间接启用）时，它将变为 `undefined`。
在此新模式下，若使用 `BuiltinIteratorReturn`，我们前面的示例现在就会正确地报错：

```ts
function* uppercase(iter: Iterator<string, BuiltinIteratorReturn>) {
  while (true) {
    const { value, done } = iter.next()
    yield value.toUppercase()
    //    ~~~~~ ~~~~~~~~~~~
    // 错误！ ┃      ┃
    //        ┃      ┗━ 属性 'toUppercase' 不存在于类型 'string' 上。你是想写 'toUpperCase' 吗？
    //        ┃
    //        ┗━ 'value' 可能为 'undefined'.

    if (done) {
      return
    }
  }
}
```

在整个 `lib.d.ts` 中，你通常会看到 `BuiltinIteratorReturn` 与 `IteratorObject` 搭配使用。
一般而言，我们建议在编写自己的代码时尽可能明确指定 `TReturn` 类型。

有关更多信息，你可以[在此阅读该特性的详细说明](https://github.com/microsoft/TypeScript/pull/58243)。

## 支持任意模块标识符

JavaScript 允许模块以字符串字面量的形式导出非法标识符名称的绑定：

```ts
const banana = '🍌'

export { banana as '🍌' }
```

同样，它也允许模块导入这些任意名称并将它们绑定到合法的标识符上：

```ts
import { '🍌' as banana } from './foo'

/**
 * om nom nom
 */
function eat(food: string) {
  console.log('Eating', food)
}

eat(banana)
```

这看起来可能只是个有趣的小把戏，但在与其他语言进行互操作时（通常跨越 JavaScript/WebAssembly 边界）它非常有用，因为其他语言对于合法标识符的判定规则可能有所不同。
它对于诸如 esbuild [的 `inject` 特性](https://esbuild.github.io/api/#inject) 等代码生成工具也同样有用。

TypeScript 5.6 现在允许你在代码中使用这些任意模块标识符！
感谢 [Evan Wallace](https://github.com/evanw) 为 TypeScript [贡献了此改动](https://github.com/microsoft/TypeScript/pull/58640)！

## `--noUncheckedSideEffectImports` 选项

在 JavaScript 中，可以通过 `import` 导入一个模块而无需从其实际导入任何值：

```ts
import 'some-module'
```

这类导入通常被称为*副作用导入*（side effect import），因为它们唯一能提供的有效行为就是执行某些副作用（例如注册全局变量，或向原型链添加 polyfill）。

在 TypeScript 中，该语法曾有一个相当奇怪的怪癖：如果该 `import` 能解析到有效的源文件，TypeScript 就会加载并检查该文件；反之，若找不到对应的源文件，TypeScript 却会静默忽略该 `import`！

这种行为令人意外，但它在一定程度上源于对 JavaScript 生态中常见模式的适配。
例如，在打包工具中配合特定的 loader 加载 CSS 或其他资源时常常会使用这种语法。
你的打包工具可能进行了相应配置，允许通过如下代码引入特定的 `.css` 文件：

```tsx
import './button-component.css'

export function Button() {
  // ...
}
```

然而，这也掩盖了副作用导入中潜在的拼写错误。
正因如此，TypeScript 5.6 引入了全新的编译器选项 `--noUncheckedSideEffectImports`，专门用来捕获此类问题。
启用 `--noUncheckedSideEffectImports` 后，如果 TypeScript 找不到副作用导入对应的源文件，就会直接报错：

```ts
import 'oops-this-module-does-not-exist'
//     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
// 错误：找不到模块 'oops-this-module-does-not-exist' 或其对应的类型声明。
```

启用该选项后，某些原本能够正常工作的代码可能会出现错误，例如上面提到的 CSS 示例。
为了解决这一问题，如果开发者仅希望对静态资源使用副作用 `import`，可以通过编写包含通配符标识符的*环境模块声明*（ambient module declaration）来处理。
该声明可以放在全局文件中，形如：

```ts
// ./src/globals.d.ts

// 将所有 CSS 文件识别为模块导入。
declare module '*.css' {}
```

实际上，你的项目中可能已经存在类似的文件了！
例如运行 `vite init` 等脚手架可能会自动生成类似的 `vite-env.d.ts`。

虽然该选项目前默认处于关闭状态，但我们非常鼓励大家尝试使用！

欲了解更多信息，请[查看此处的具体实现](https://github.com/microsoft/TypeScript/pull/58941)。

## `--noCheck` 选项

TypeScript 5.6 引入了一个新的编译器选项 `--noCheck`，允许你跳过对所有输入文件的类型检查。
这可以避免在进行生成输出文件所需的任何语义分析时，执行不必要的类型检查。

应用场景之一是将 JavaScript 文件的生成与类型检查分离，使两者作为独立的阶段运行。
例如，你可以在迭代时运行 `tsc --noCheck`，而在需要彻底检查时运行 `tsc --noEmit`。
你也可以并行运行这两个任务，甚至在 `--watch` 模式下运行，不过请注意，如果确实同时运行它们，可能需要指定独立的 `--tsBuildInfoFile` 路径。

`--noCheck` 在类似模式下对于生成声明文件同样非常有用。
在符合 `--isolatedDeclarations` 规范的项目中指定 `--noCheck`，TypeScript 可以在无需经历类型检查阶段的情况下快速生成声明文件。
生成的声明文件将纯粹依赖快速的句法转换。

需要注意的是，在指定了 `--noCheck` 但项目*未*使用 `--isolatedDeclarations` 的情况下，TypeScript 仍可能会执行生成 `.d.ts` 文件所必需的类型检查。
从这个角度来看，`--noCheck` 的名称有些不尽如人意；然而，该过程比完整的类型检查要更加惰性，它仅计算未显式标注类型的声明的类型。
这理应比完整的类型检查快得多。

`noCheck` 也可以通过 TypeScript API 作为标准选项使用。
在内部，`transpileModule` 和 `transpileDeclaration` 已经通过使用 `noCheck` 来加速处理（至少自 TypeScript 5.5 起）。
现在任何构建工具都能够利用该标志，采用各种自定义策略来协调并加速构建。

欲了解更多信息，请参阅[在 TypeScript 5.5 中从内部增强 `noCheck` 的相关工作](https://github.com/microsoft/TypeScript/pull/58364)，以及使其在[命令行中](https://github.com/microsoft/TypeScript/pull/58839)和

## 允许 `--build` 在存在中间错误时继续构建

TypeScript 的*项目引用*（project references）概念允许你将代码库组织为多个项目并在它们之间建立依赖关系。
以 `--build` 模式运行 TypeScript 编译器（简写为 `tsc -b`）是跨项目执行构建并计算出哪些项目与文件需要编译的内置方式。

此前，使用 `--build` 模式会默认假设 `--noEmitOnError`，并在遇到任何错误时立即终止构建。
这意味着如果任何“上游”依赖存在构建错误，“下游”项目就完全无法进行检查和构建。
在理论上，这种方式无可挑剔——毕竟如果一个项目存在错误，它对于下游依赖而言未必处于自洽状态。

但在实际开发中，这种僵化性给升级等工作带来了极大痛苦。
例如，若 `projectB` 依赖于 `projectA`，即便对 `projectB` 更熟悉的开发者想要主动升级自身代码，也必须等到依赖升级完成才行，他们会直接受阻于升级 `projectA` 的进度。

自 TypeScript 5.6 起，即使依赖项中存在中间错误，`--build` 模式仍将继续构建后续项目。
遇到中间错误时，这些错误会被如实报告，编译器会尽力生成输出文件，并且构建流程会一直推进，直至指定项目完成。

如果你希望在第一个出现错误的项目处就中止构建，可以使用新增的标志 `--stopOnBuildErrors`。
这在 CI 环境中，或者在对被大量其他项目深度依赖的底层项目进行迭代时非常有用。

请注意，为了实现此功能，现在只要在 `--build` 调用中涉及的任何项目，TypeScript 都会始终生成 `.tsbuildinfo` 文件（即使未显式指定 `--incremental` 或 `--composite`）。
这是为了记录 `--build` 的调用状态以及后续需要执行的工作。

你可以在[具体的实现 PR 中阅读有关此改动的更多信息](https://github.com/microsoft/TypeScript/pull/58838)。

## 编辑器中的区域优先诊断

当 TypeScript 语言服务被请求获取某个文件的*诊断信息*（diagnostics，如错误、建议与弃用提示）时，通常需要检查*整个文件*。
大部分情况下这没什么问题，但在超大文件中可能会引发明显的延迟。
这往往令人沮丧，因为修改一个拼写错误本应是一个快速操作，但在足够大的文件中却可能要等待数秒之久。

为了解决这个问题，TypeScript 5.6 引入了一项名为*区域优先诊断*（region-prioritized diagnostics）或*区域优先检查*（region-prioritized checking）的新特性。
编辑器现在不仅可以请求一组文件的诊断信息，还可以提供指定文件的相关区域——通常是用户当前在屏幕上可见的文件区域。
TypeScript 语言服务器随后可以选择返回两组诊断信息：一组针对该局部区域，另一组针对整个完整文件。
这样一来，在超大文件中进行编辑时，响应速度会有显著提升，你不再需要苦苦等待那些红色波浪线消失。

来看一些具体数据：在针对 [TypeScript 自身的 `checker.ts`](https://github.com/microsoft/TypeScript/blob/7319968e90600102892a79142fb804bcbe384160/src/compiler/checker.ts) 进行测试时，完整的语义诊断响应耗时 3330 毫秒；
相比之下，首次基于区域的诊断响应仅耗时 143 毫秒！
虽然剩余的全文件诊断仍需约 3200 毫秒，但这对于快速编辑体验而言带来了质的飞跃。

这项特性还涵盖了大量工作，以确保在整个使用体验中诊断信息的报告更加一致。
由于类型检查器利用缓存来避免重复工作，相同类型之间的后续检查往往会产生不同（通常更简短）的错误信息。
从技术上讲，即使在此特性出现之前，惰性、非顺序的检查就可能导致编辑器中不同位置报告的诊断信息不一致——但我们不希望加剧这个问题。
通过最近的改进，我们已经消除了许多这类错误信息不一致的情况。

目前，Visual Studio Code 针对 TypeScript 5.6 及更高版本已提供该功能。

欲了解更多详细信息，请[查看此处的实现与说明](https://github.com/microsoft/TypeScript/pull/57842)。

## 细粒度的补全提交字符

TypeScript 语言服务现在为每个自动补全项提供了独立的*提交字符*（commit characters）。
提交字符指的是特定字符：当键入该字符时，编辑器会自动确认（提交）当前建议的补全项。

这意味着，当你在键入特定字符时，编辑器将能够更智能、更频繁地采纳当前建议的补全项。
以如下代码为例：

```ts
declare let food: {
    eat(): any;
}

let f = (foo/**/
```

当光标位于 `/**/` 处时，尚无法确定我们要编写的代码是像 `let f = (food.eat())` 这样，还是像 `let f = (foo, bar) => foo + bar` 这样。
你可以想象，根据接下来键入的字符不同，编辑器本应能够给出不同的自动补全行为。
例如，如果我们键入点号（`.`），我们大概率希望编辑器补全变量 `food`；
但如果键入逗号（`,`），我们可能是在为箭头函数声明形参。

遗憾的是，此前 TypeScript 仅仅向编辑器提示当前文本可能正在定义一个新的参数名，导致*没有任何*提交字符是安全的。
因此，即便编辑器“显而易见”应该自动补全为单词 `food`，按下 `.` 也不会产生任何效果。

现在，TypeScript 会针对每个补全项显式列出哪些字符可以安全用于提交。
虽然这不会*立刻*颠覆你的日常开发体验，但支持这些提交字符的编辑器体验会随着时间推移不断改善。
如果想立刻体验这些改进，你可以配合 [Visual Studio Code Insiders](https://code.visualstudio.com/insiders/) 使用 [TypeScript Nightly 扩展](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-typescript-next)。
在上述代码中按下 `.` 将正确自动补全为 `food`。

欲了解更多信息，请参阅[添加提交字符的 Pull Request](https://github.com/microsoft/TypeScript/pull/59339) 以及[根据上下文调整提交字符的相关改动](https://github.com/microsoft/TypeScript/pull/59523)。

## 自动导入的排除模式

TypeScript 语言服务现在允许你指定一组正则表达式模式，用于过滤掉来自某些模块标识符的自动导入建议。
例如，若要排除像 `lodash` 这类包的所有“深层”导入，可以在 Visual Studio Code 中配置如下首选项：

```json5
{
  'typescript.preferences.autoImportSpecifierExcludeRegexes': ['^lodash/.*$'],
}
```

反过来，如果你希望禁止直接从包的入口点导入：

```json5
{
  'typescript.preferences.autoImportSpecifierExcludeRegexes': ['^lodash$'],
}
```

甚至可以通过如下设置避免 `node:` 协议导入：

```json5
{
  'typescript.preferences.autoImportSpecifierExcludeRegexes': ['^node:'],
}
```

注意，如果你需要指定正则表达式标志（如 `i` 或 `u`），需要用斜杠包裹正则表达式。
使用斜杠包裹时，需要对内部的其他斜杠进行转义：

```json5
{
  'typescript.preferences.autoImportSpecifierExcludeRegexes': [
    '^./lib/internal', // 无需转义
    '/^.\\/lib\\/internal/', // 需要转义 —— 注意开头和结尾的斜杠
    '/^.\\/lib\\/internal/i', // 需要转义 —— 我们需要斜杠来提供 'i' 正则修饰符
  ],
}
```

在 Visual Studio Code 中，通过 `javascript.preferences.autoImportSpecifierExcludeRegexes` 也可以将相同的设置应用于 JavaScript。

欲了解更多信息，请[查看此处的具体实现](https://github.com/microsoft/TypeScript/pull/59543)。

## 显著的行为变更

本节重点介绍在任何版本升级过程中都应知晓并理解的重要变更。
内容可能涵盖弃用项、移除项和新增的限制规则，也包括在功能上属于修复与改进、但可能因引入新错误而影响现有构建的改动。

### `lib.d.ts`

针对 DOM 生成的类型可能会对代码库的类型检查产生影响。
欲了解更多信息，请[查看与本版本 TypeScript 的 DOM 及 `lib.d.ts` 更新相关的议题](https://github.com/microsoft/TypeScript/issues/58764)。

### `.tsbuildinfo` 总是会被写入

为了使 `--build` 能够在依赖存在中间错误时仍继续构建项目，并支持在命令行中使用 `--noCheck`，TypeScript 现在在每次 `--build` 调用中都会始终为所有项目生成 `.tsbuildinfo` 文件。
无论实际上是否启用了 `--incremental`，都会生成该文件。
[在此查看更多信息](https://github.com/microsoft/TypeScript/pull/58626)。

### 遵循 `node_modules` 内部的文件后缀与 `package.json`

在 Node.js v12 实现对 ECMAScript 模块的支持之前，TypeScript 始终无法准确判断在 `node_modules` 中发现的 `.d.ts` 文件究竟代表用 CommonJS 还是 ECMAScript 编写的 JavaScript 文件。
当 npm 上绝大多数模块都只支持 CommonJS 时，这并没有引起太多问题——遇到存疑情况时，TypeScript 直接假定所有代码都遵循 CommonJS 规范即可。
不幸的是，如果这一假定不成立，就可能导致不安全的导入：

```ts
// node_modules/dep/index.d.ts
export declare function doSomething(): void

// index.ts
// 如果 "dep" 是 CommonJS 模块则正常，但如果
// 它是 ECMAScript 模块则会失败 —— 即使在打包器中也是如此！
import dep from 'dep'
dep.doSomething()
```

在实际开发中这种情况并不常出现。
但自从 Node.js 开始支持 ECMAScript 模块以来的几年里，npm 上 ESM 的占比不断增加。
幸运的是，Node.js 还引入了一种机制来帮助 TypeScript 判断文件是 ECMAScript 模块还是 CommonJS 模块：`.mjs` 和 `.cjs` 文件后缀以及 `package.json` 中的 `"type"` 字段。
TypeScript 4.7 添加了对这些标识的识别支持，以及编写 `.mts` 和 `.cts` 文件的能力；
然而，TypeScript *仅*在 `--module node16` 和 `--module nodenext` 下才会读取这些标识，因此上述不安全的导入对于使用 `--module esnext` 和 `--moduleResolution bundler` 的用户来说依然是个隐患。

为了解决这个问题，TypeScript 5.6 会收集模块格式信息，并在*所有* `module` 模式（`amd`、`umd` 和 `system` 除外）下利用这些信息来解决类似上例中的歧义。
无论出现在何处，特定格式的文件后缀（`.mts` 和 `.cts`）都会受到遵循，并且无论 `module` 设置为何，都会参考 `node_modules` 依赖内部的 `package.json` `"type"` 字段。
此前，在技术上是有可能将 CommonJS 输出写入 `.mjs` 文件的，反之亦然：

```ts
// main.mts
export default 'oops'

// $ tsc --module commonjs main.mts
// main.mjs
Object.defineProperty(exports, '__esModule', { value: true })
exports.default = 'oops'
```

现在，`.mts` 文件绝不会输出 CommonJS 代码，而 `.cts` 文件也绝不会输出 ESM 代码。

请注意，该行为的大部分在 TypeScript 5.5 预览版本中便已提供（[实现细节参见此处](https://github.com/microsoft/TypeScript/pull/57896)），而在 5.6 中，该行为仅被扩展至 `node_modules` 内部的文件。

有关此改动的更多细节可[在此查看](https://github.com/microsoft/TypeScript/pull/58825)。

### 计算属性上正确的 `override` 检查

此前，带有 `override` 修饰符的计算属性未能正确检查基类成员是否存在。
同样地，如果你使用了 `noImplicitOverride`，若*遗漏*为计算属性添加 `override` 修饰符，也不会收到任何报错。

TypeScript 5.6 现在对这两种情况下的计算属性均进行正确检查：

```ts
const foo = Symbol('foo')
const bar = Symbol('bar')

class Base {
  [bar]() {}
}

class Derived extends Base {
  override [foo]() {}
  //           ~~~~~
  // 错误：此成员不能具有 'override' 修饰符，因为它未在基类 'Base' 中声明。

  [bar]() {}
  //  ~~~~~
  // 在 noImplicitOverride 下报错：此成员必须具有 'override' 修饰符，因为它重写了基类 'Base' 中的成员。
}
```

感谢 [Oleksandr Tarasiuk](https://github.com/a-tarasyuk) 在[此 Pull Request](https://github.com/microsoft/TypeScript/pull/57146) 中贡献了该修复。
