---
title: TypeScript 5.5
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-5-5.html
oneline: TypeScript 5.5 发布说明
---

## 推断类型谓词

_本节由 [Dan Vanderkam](https://github.com/danvk) 撰写，他[在 TypeScript 5.5 中实现了该特性](https://github.com/microsoft/TypeScript/pull/57465)。感谢 Dan！_

TypeScript 的控制流分析能够出色地跟踪变量在代码执行流中的类型变化：

```tsx
interface Bird {
  commonName: string
  scientificName: string
  sing(): void
}

// 将国家名称映射到国鸟。
// 并非所有国家都有官方国鸟（说的就是你，加拿大！）
declare const nationalBirds: Map<string, Bird>

function makeNationalBirdCall(country: string) {
  const bird = nationalBirds.get(country) // bird 的声明类型为 Bird | undefined
  if (bird) {
    bird.sing() // 在 if 语句内部，bird 的类型为 Bird
  } else {
    // 在此处，bird 的类型为 undefined。
  }
}
```

通过强制开发者处理 `undefined` 的情况，TypeScript 促使你编写出更加健壮的代码。

在过去，这种类型细化（type refinement）很难直接应用于数组。在 TypeScript 5.5 之前的所有版本中，以下代码都会报错：

```tsx
function makeBirdCalls(countries: string[]) {
  // birds: (Bird | undefined)[]
  const birds = countries
    .map((country) => nationalBirds.get(country))
    .filter((bird) => bird !== undefined)

  for (const bird of birds) {
    bird.sing() // 错误：'bird' 可能为 'undefined'。
  }
}
```

这段代码本身没有任何问题：我们已经从列表中过滤掉了所有 `undefined` 值。
然而，先前的 TypeScript 并不能理解这种过滤操作。

在 TypeScript 5.5 中，类型检查器可以顺利通过这段代码：

```tsx
function makeBirdCalls(countries: string[]) {
  // birds: Bird[]
  const birds = countries
    .map((country) => nationalBirds.get(country))
    .filter((bird) => bird !== undefined)

  for (const bird of birds) {
    bird.sing() // 正常！
  }
}
```

请注意 `birds` 此时拥有了更加精确的类型。

这是因为 TypeScript 现在会为 `filter` 函数推断出[类型谓词（type predicate）](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates)。
将其提取为独立函数可以更清晰地看清背后的原理：

```tsx
// function isBirdReal(bird: Bird | undefined): bird is Bird
function isBirdReal(bird: Bird | undefined) {
  return bird !== undefined
}
```

`bird is Bird` 即为类型谓词。
它意味着：如果函数返回 `true`，则参数是 `Bird`（如果函数返回 `false`，则为 `undefined`）。
`Array.prototype.filter` 的类型声明能够识别类型谓词，因此最终能得到更精确的类型，代码也能顺利通过类型检查。

当满足以下条件时，TypeScript 会推断该函数返回类型谓词：

1. 函数没有显式的返回类型或类型谓词标注。
2. 函数只有一条 `return` 语句，且没有隐式返回。
3. 函数不会修改其参数。
4. 函数返回一个与参数细化相关的 `boolean` 表达式。

通常情况下，它的表现完全符合你的直觉。
以下是推断类型谓词的更多示例：

```tsx
// const isNumber: (x: unknown) => x is number
const isNumber = (x: unknown) => typeof x === 'number'

// const isNonNullish: <T>(x: T) => x is NonNullable<T>
const isNonNullish = <T,>(x: T) => x != null
```

在过去，TypeScript 只会将这些函数的返回类型推断为 `boolean`。
现在，它能够推断出包含类型谓词的签名，例如 `x is number` 或 `x is NonNullable<T>`。

类型谓词具有“当且仅当”（充要条件）语义。
如果一个函数返回 `x is T`，则意味着：

1. 如果函数返回 `true`，则 `x` 的类型为 `T`。
2. 如果函数返回 `false`，则 `x` 的类型*不是* `T`。

如果你期望推断出类型谓词却没有推断出来，可能是违反了第二条规则。这种情况常见于“真值（truthiness）”检查：

```tsx
function getClassroomAverage(
  students: string[],
  allScores: Map<string, number>,
) {
  const studentScores = students
    .map((student) => allScores.get(student))
    .filter((score) => !!score)

  return studentScores.reduce((a, b) => a + b) / studentScores.length
  //     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  // 错误：对象可能为 'undefined'。
}
```

TypeScript 没有为 `score => !!score` 推断类型谓词，这是完全正确的：如果该函数返回 `true`，`score` 确实是 `number`；
但如果返回 `false`，`score` 既可能是 `undefined`，也可能是 `number`（具体来说是 `0`）。
这会导致真正的 bug：如果有学生考了零分，过滤掉其成绩会导致平均分虚高。
“高于平均分的人更少，伤心的人更多”！

正如第一个示例一样，更好的做法是显式过滤掉 `undefined` 值：

```tsx
function getClassroomAverage(
  students: string[],
  allScores: Map<string, number>,
) {
  const studentScores = students
    .map((student) => allScores.get(student))
    .filter((score) => score !== undefined)

  return studentScores.reduce((a, b) => a + b) / studentScores.length // 正常！
}
```

对于不存在二义性的对象类型，真值检查*确实会*推断出类型谓词。
需要牢记的是，函数必须返回 `boolean` 才能成为推断类型谓词的候选者：`x => !!x` 可能会推断出类型谓词，但 `x => x` 绝对不会。

显式类型谓词的行为与以往完全一致。
TypeScript 不会检查自己推断出的谓词是否与显式标注一致。
显式类型谓词（"is"）在安全性上与类型断言（"as"）并无二致。

如果 TypeScript 推断出的类型比你预期的更精确，这项特性可能会对现有代码造成破坏性变更（breaking change）。例如：

```tsx
// 此前，nums: (number | null)[]
// 现在，nums: number[]
const nums = [1, 2, 3, null, 5].filter((x) => x !== null)

nums.push(null) // 在 TS 5.4 中正常，在 TS 5.5 中报错
```

修复方法是通过显式类型标注告诉 TypeScript 你期望的类型：

```tsx
const nums: (number | null)[] = [1, 2, 3, null, 5].filter((x) => x !== null)
nums.push(null) // 在所有版本中均正常
```

有关更多信息，请查阅[实现该特性的 Pull Request](https://github.com/microsoft/TypeScript/pull/57465) 以及 [Dan 关于实现该特性的博文](https://effectivetypescript.com/2024/04/16/inferring-a-type-predicate/)。

## 常量索引访问的控制流类型收窄

当 `obj` 与 `key` 均为事实上的常量时，TypeScript 现在能够对 `obj[key]` 形式的表达式进行类型收窄。

```ts
function f1(obj: Record<string, unknown>, key: string) {
  if (typeof obj[key] === 'string') {
    // 现在正常，此前报错
    obj[key].toUpperCase()
  }
}
```

在上面的代码中，`obj` 和 `key` 均未被重新赋值或修改，因此 TypeScript 可以在 `typeof` 检查之后将 `obj[key]` 的类型收窄为 `string`。
更多信息，[请参见实现该特性的 Pull Request](https://github.com/microsoft/TypeScript/pull/57847)。

## JSDoc `@import` 标签

如今，如果想在 JavaScript 文件中仅为了类型检查而导入某些内容，操作往往十分繁琐。
由于 `SomeType` 在运行时并不存在，JavaScript 开发者无法直接导入该类型：

```js
// ./some-module.d.ts
export interface SomeType {
    // ...
}

// ./index.js
import { SomeType } from "./some-module"; // ❌ 运行时错误！

/**
 * @param {SomeType} myValue
 */
function doSomething(myValue) {
    // ...
}
```

`SomeType` 在运行时不存在，因此该导入会报错。
开发者可以改用命名空间导入：

```js
import * as someModule from './some-module'

/**
 * @param {someModule.SomeType} myValue
 */
function doSomething(myValue) {
  // ...
}
```

但 `./some-module` 仍然会在运行时被导入——这可能同样不符合预期。

为了避免这种情况，开发者通常只能在 JSDoc 注释中使用 `import(...)` 类型：

```js
/**
 * @param {import("./some-module").SomeType} myValue
 */
function doSomething(myValue) {
  // ...
}
```

如果想在多处复用该类型，可以使用 `@typedef` 避免重复书写导入路径：

```js
/**
 * @typedef {import("./some-module").SomeType} SomeType
 */

/**
 * @param {SomeType} myValue
 */
function doSomething(myValue) {
  // ...
}
```

这有助于在当前作用域复用 `SomeType`，但面对大量导入时依然显得繁琐且冗长。

为此，TypeScript 现在支持了全新的 `@import` 注释标签，其语法与 ECMAScript 的导入语句完全一致：

```js
/** @import { SomeType } from "some-module" */

/**
 * @param {SomeType} myValue
 */
function doSomething(myValue) {
  // ...
}
```

这里使用了具名导入（named imports）。
我们也可以将其写为命名空间导入（namespace import）：

```js
/** @import * as someModule from "some-module" */

/**
 * @param {someModule.SomeType} myValue
 */
function doSomething(myValue) {
  // ...
}
```

由于这些仅是 JSDoc 注释，因此它们完全不会影响任何运行时行为。

在此衷心感谢 [Oleksandr Tarasiuk](https://github.com/a-tarasyuk)，感谢他贡献了[该变更](https://github.com/microsoft/TypeScript/pull/57207)！

## 正则表达式语法检查

在此之前，TypeScript 通常会跳过代码中的绝大多数正则表达式。
这是因为从技术上讲，正则表达式拥有可扩展的语法，且 TypeScript 从未尝试过将正则表达式降级编译为较早版本的 JavaScript。
然而，这也意味着正则表达式中的许多常见错误无法被及时发现，最终要么在运行时抛出错误，要么静默失败。

现在，TypeScript 可以对正则表达式进行基础的语法检查！

```ts
let myRegex = /@robot(\s+(please|immediately)))? do some task/
//                                            ~
// 错误！
// 意外的 ')'。你是想用反斜杠对其进行转义吗？
```

这是一个简单的示例，但这项检查能够捕获大量常见错误。
事实上，TypeScript 的检查不仅限于纯语法层面。
例如，TypeScript 现在可以捕获引用了不存在的分组的反向引用（backreference）：

```ts
let myRegex = /@typedef \{import\((.+)\)\.([a-zA-Z_]+)\} \3/u
//                                                        ~
// 错误！
// 此反向引用指向了一个不存在的分组。
// 此正则表达式中仅有 2 个捕获组。
```

具名捕获组（named capturing groups）同样如此：

```ts
let myRegex =
  /@typedef \{import\((?<importPath>.+)\)\.(?<importedEntity>[a-zA-Z_]+)\} \k<namedImport>/
//                                                                                        ~~~~~~~~~~~
// 错误！
// 此正则表达式中没有名为 'namedImport' 的捕获组。
```

TypeScript 的检查现在还能感知某些正则表达式特性是否高于你配置的目标 ECMAScript 版本。
例如，如果我们在目标版本为 ES5 的项目中像上面那样使用具名捕获组，就会收到报错：

```ts
let myRegex =
  /@typedef \{import\((?<importPath>.+)\)\.(?<importedEntity>[a-zA-Z_]+)\} \k<importedEntity>/
//                                  ~~~~~~~~~~~~         ~~~~~~~~~~~~~~~~
// 错误！
// 具名捕获组仅在目标设置为 'ES2018' 或更高版本时可用。
```

对于某些正则表达式标志（flags）也是如此。

需要注意的是，TypeScript 的正则表达式支持仅限于正则表达式*字面量*。
如果使用字符串字面量调用 `new RegExp`，TypeScript 不会检查传入的字符串。

我们非常感谢 [GitHub 用户 graphemecluster](https://github.com/graphemecluster/)，他与我们进行了大量迭代与合作，[促成了该特性在 TypeScript 中的落地](https://github.com/microsoft/TypeScript/pull/55600)。

## 支持崭新的 ECMAScript `Set` 方法

TypeScript 5.5 为 [ECMAScript `Set` 类型声明了新提案中的方法](https://github.com/tc39/proposal-set-methods)。

其中部分方法（如 `union`、`intersection`、`difference` 和 `symmetricDifference`）接受另一个 `Set` 并返回一个新的 `Set` 作为结果；
其他方法（如 `isSubsetOf`、`isSupersetOf` 和 `isDisjointFrom`）接受另一个 `Set` 并返回一个 `boolean`。
这些方法均不会修改原有的 `Set`。

以下是这些方法的使用示例及其行为表现：

```ts
let fruits = new Set(['apples', 'bananas', 'pears', 'oranges'])
let applesAndBananas = new Set(['apples', 'bananas'])
let applesAndOranges = new Set(['apples', 'oranges'])
let oranges = new Set(['oranges'])
let emptySet = new Set()

////
// union
////

// Set(4) {'apples', 'bananas', 'pears', 'oranges'}
console.log(fruits.union(oranges))

// Set(3) {'apples', 'bananas', 'oranges'}
console.log(applesAndBananas.union(oranges))

////
// intersection
////

// Set(2) {'apples', 'bananas'}
console.log(fruits.intersection(applesAndBananas))

// Set(0) {}
console.log(applesAndBananas.intersection(oranges))

// Set(1) {'apples'}
console.log(applesAndBananas.intersection(applesAndOranges))

////
// difference
////

// Set(3) {'apples', 'bananas', 'pears'}
console.log(fruits.difference(oranges))

// Set(2) {'pears', 'oranges'}
console.log(fruits.difference(applesAndBananas))

// Set(1) {'bananas'}
console.log(applesAndBananas.difference(applesAndOranges))

////
// symmetricDifference
////

// Set(2) {'bananas', 'oranges'}
console.log(applesAndBananas.symmetricDifference(applesAndOranges)) // 没有 apples

////
// isDisjointFrom
////

// true
console.log(applesAndBananas.isDisjointFrom(oranges))

// false
console.log(applesAndBananas.isDisjointFrom(applesAndOranges))

// true
console.log(fruits.isDisjointFrom(emptySet))

// true
console.log(emptySet.isDisjointFrom(emptySet))

////
// isSubsetOf
////

// true
console.log(applesAndBananas.isSubsetOf(fruits))

// false
console.log(fruits.isSubsetOf(applesAndBananas))

// false
console.log(applesAndBananas.isSubsetOf(oranges))

// true
console.log(fruits.isSubsetOf(fruits))

// true
console.log(emptySet.isSubsetOf(fruits))

////
// isSupersetOf
////

// true
console.log(fruits.isSupersetOf(applesAndBananas))

// false
console.log(applesAndBananas.isSupersetOf(fruits))

// false
console.log(applesAndBananas.isSupersetOf(oranges))

// true
console.log(fruits.isSupersetOf(fruits))

// false
console.log(emptySet.isSupersetOf(fruits))
```

我们要感谢 [Kevin Gibbons](https://github.com/bakkot)，他不仅是 ECMAScript 中该特性的共同推动者，[还在 TypeScript 中为 `Set`、`ReadonlySet` 和 `ReadonlySetLike` 提供了类型声明](https://github.com/microsoft/TypeScript/pull/57230)！

## 独立声明生成（Isolated Declarations）

_本节由支持了独立声明生成设计的 [Rob Palmer](https://github.com/robpalme) 共同撰写。_

声明文件（即 `.d.ts` 文件）用于向 TypeScript 描述现有库和模块的类型结构。
这种轻量级的描述仅包含库的类型签名，剥离了函数体等具体实现细节。
发布声明文件后，TypeScript 就能高效地检查你对该库的使用是否正确，而无需去分析库本身的实现代码。
虽然可以手动编写声明文件，但如果你编写的是带有类型标注的代码，使用 `--declaration` 让 TypeScript 直接根据源码自动生成声明文件会更加安全且省心。

一直以来，生成声明文件都是 TypeScript 编译器及其 API 的职责；
然而在某些场景下，你可能希望使用其他工具，或者传统的构建流程在大规模场景下遇到了扩展性瓶颈。

### 使用场景：更快速的声明生成工具

设想一下，如果你想打造一个更快的声明文件生成工具，例如将其作为发布服务或新型打包器（bundler）的一部分。
尽管社区中已有蓬勃发展、速度极快的工具生态可以将 TypeScript 转译为 JavaScript，但在将 TypeScript 转换为声明文件方面却非如此。
究其原因，是因为 TypeScript 强大的类型推断允许我们在不显式声明类型的情况下编写代码，这也意味着生成声明文件的逻辑极其复杂。

我们来看一个将两个导入变量相加的简单函数示例：

```ts
// util.ts
export let one = '1'
export let two = '2'

// add.ts
import { one, two } from './util'
export function add() {
  return one + two
}
```

即使我们仅仅想要生成 `add.d.ts`，TypeScript 也必须深入跟踪到另一个导入的文件（`util.ts`），推断出 `one` 和 `two` 的类型为字符串，然后再计算出两个字符串相加操作（`+`）会得到 `string` 类型的返回值：

```ts
// add.d.ts
export declare function add(): string
```

虽然这种推断对于开发体验至关重要，但它也意味着任何想要生成声明文件的第三方工具，都必须重新实现类型检查器的部分核心能力，包括类型推断以及解析模块说明符以跟踪导入关系的能力。

### 使用场景：并行声明生成与并行类型检查

设想你有一个包含众多项目的 Monorepo，以及一颗渴望帮你更快完成代码检查的多核 CPU。
如果我们能让每个项目在不同的 CPU 核心上同时运行，并行检查所有项目，岂不是很棒？

遗憾的是，我们并不能随心所欲地并行执行所有工作。
原因在于我们必须按照依赖顺序构建这些项目，因为每个项目都需要依赖其上游项目的声明文件来进行类型检查。
因此，必须先构建上游依赖以生成声明文件。
TypeScript 的项目引用（project references）特性也是以同样的方式工作，即按照“拓扑”依赖顺序依次构建项目集合。

例如，如果我们有两个项目 `backend` 和 `frontend`，它们都依赖名为 `core` 的项目，那么在 `core` 完成构建并生成声明文件之前，TypeScript 无法开始对 `frontend` 或 `backend` 进行类型检查。

![frontend 和 backend 指向 core，其他内容可能分别指向它们](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2024/04/5-5-beta-isolated-declarations-deps.png)

在上图中可以看到一个明显的瓶颈。
虽然我们可以并行构建 `frontend` 和 `backend`，但必须首先等待 `core` 构建完成，两者的构建才能启动。

我们该如何改善这一现状？
设想一下，如果有一个快速工具能够*并行*为 `core` 生成所有声明文件，那么 TypeScript 紧接着就可以*同时并行*对 `core`、`frontend` 和 `backend` 进行类型检查。

### 解决方案：显式类型标注！

在这两个使用场景中，共同的痛点在于生成声明文件都需要一个跨文件的类型检查器。
而这对工具链生态社区来说要求过高了。

再看一个更复杂的例子，如果我们想为以下代码生成声明文件……

```ts
import { add } from './add'

const x = add()

export function foo() {
  return x
}
```

……我们需要为 `foo` 生成签名。
这需要检查 `foo` 的实现；
而 `foo` 只是返回了 `x`，因此获取 `x` 的类型又需要检查 `add` 的实现；
这甚至可能进一步需要检查 `add` 的依赖项的实现，以此类推。
由此可见，生成声明文件需要大量的逻辑来推导不同位置的类型，而这些位置甚至可能不在当前文件内。

不过，对于追求极致迭代速度和完全并行构建的开发者而言，还有另一种思考该问题的角度：
声明文件仅仅需要模块公共 API 的类型——换句话说，只需导出内容的类型。
如果开发者愿意做出权衡，显式标注其导出的所有实体的类型，工具就能在无需分析模块实现代码、也无需重新实现完整类型检查器的情况下，直接生成声明文件。

这就是全新 `--isolatedDeclarations` 选项发挥作用的地方。
`--isolatedDeclarations` 会在模块在没有类型检查器就无法可靠转换时报错。
更直白地说，如果某个文件未对其导出内容提供充分的类型标注，TypeScript 就会报错。

这意味着在上面的例子中，我们会看到如下报错：

```ts
export function foo() {
  //              ~~~
  // 错误！在 --isolatedDeclarations 下，函数必须
  // 具有显式的返回类型注解。
  return x
}
```

### 为什么报错是件好事？

因为这意味着 TypeScript 能够：

1. 提前告知我们其他第三方工具在生成声明文件时是否会遇到问题；
2. 提供快速修复（Quick Fix）来帮助自动添加这些缺失的类型标注。

不过，该模式并不会要求*处处*都添加类型标注。
对于局部变量，由于它们不会影响公共 API，因此可以忽略。
例如，以下代码**不会**产生错误：

```ts
import { add } from './add'

const x = add('1', '2') // 'x' 没有报错，因为它未被导出。

export function foo(): string {
  return x
}
```

对于某些类型推导显而易见（"trivial"）的表达式，同样不需要显式标注：

```ts
// 'x' 没有报错。
// 很容易计算出其类型为 'number'
export let x = 10

// 'y' 没有报错。
// 我们可以从 return 表达式中获取其类型。
export function y() {
  return 20
}

// 'z' 没有报错。
// 类型断言清楚地表明了其类型是什么。
export function z() {
  return Math.max(x, y()) as number
}
```

### 使用 `isolatedDeclarations`

`isolatedDeclarations` 要求同时启用 `declaration` 或 `composite` 标志。

请注意，`isolatedDeclarations` 并不会改变 TypeScript 本身生成代码的方式，而只是改变了其报告错误的方式。
同样重要的是，类似于 `isolatedModules`，仅在 TypeScript 中启用该选项并不会立即带来本文讨论的潜在收益。
因此请保持耐心，期待该领域未来的工具生态演进。
考虑到工具作者的需求，我们也必须承认，目前并非所有 TypeScript 的声明生成逻辑都能被其他以此为参考的工具轻松复制。
这正是我们目前正在积极改进的方向。

此外，独立声明生成仍是一项新特性，我们正在积极优化其体验。
某些场景（例如在类和对象字面量中使用计算属性声明）在 `isolatedDeclarations` 下*暂未*获得支持。
敬请关注后续进展，并随时向我们提供反馈。

我们还认为有必要指出，是否采用 `isolatedDeclarations` 应根据实际情况具体权衡。
启用 `isolatedDeclarations` 会在一定程度上牺牲开发人体工程学体验（developer ergonomics），因此如果你的构建环境并未利用前文提到的两种场景，它可能并不是最佳选择。
而对于其他场景，围绕 `isolatedDeclarations` 的工作已经挖掘出许多优化点，并解锁了多种不同的并行构建策略。
与此同时，如果你愿意做出权衡，随着外部工具链生态的日益成熟，我们相信 `isolatedDeclarations` 将成为加速构建流程的强大工具。

欲了解更多信息，请查阅 TypeScript 问题追踪器上的 [Isolated Declarations: State of the Feature](https://github.com/microsoft/TypeScript/issues/58944) 讨论。

### 致谢

`isolatedDeclarations` 的工作是 TypeScript 团队与 Bloomberg 和 Google 内部基础设施及工具团队长期合作的成果。
来自 Google 的 Hana Joo 实现了[针对独立声明错误的快速修复](https://github.com/microsoft/TypeScript/pull/58260)（稍后详述），Ashley Claymore、Jan Kühle、Lisa Velden、Rob Palmer 和 Thomas Chetwin 也参与了数月以来的讨论、规范制定和实现工作。
而我们尤为想感谢来自 Bloomberg 的 [Titian Cernicova-Dragomir](https://github.com/dragomirtitian) 所付出的巨大努力。
Titian 在推动 `isolatedDeclarations` 的落地实现中发挥了举足轻重的作用，多年来也是 TypeScript 项目的核心贡献者。

尽管该特性涉及大量变动，你可以在此处查看[独立声明生成的核心工作](https://github.com/microsoft/TypeScript/pull/58201)。

## 配置文件中的 `${configDir}` 模板变量

在许多代码库中，通常会复用一个共享的 `tsconfig.json` 文件作为其他配置文件的“基础配置（base）”。
这可以通过在 `tsconfig.json` 文件中使用 `extends` 字段来实现：

```json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "./dist"
  }
}
```

但这种方式存在一个问题：`tsconfig.json` 文件中的所有路径都是相对于该文件自身所在位置解析的。
这意味着如果你有一个被多个项目共享的 `tsconfig.base.json` 文件，其中的相对路径在派生项目中往往无法按预期工作。
例如，设想有如下 `tsconfig.base.json`：

```json5
{
  compilerOptions: {
    typeRoots: ['./node_modules/@types', './custom-types'],
    outDir: 'dist',
  },
}
```

如果作者的本意是让每个继承此配置的 `tsconfig.json` 都能：

1. 输出到相对于派生 `tsconfig.json` 的 `dist` 目录；
1. 拥有相对于派生 `tsconfig.json` 的 `custom-types` 目录，

那么上述配置将无法满足需求。
`typeRoots` 的路径会相对于共享的 `tsconfig.base.json` 所在位置解析，而不是相对于继承它的派生项目。
每个继承该文件的项目都必须重复声明内容完全相同的 `outDir` 和 `typeRoots`。
这不仅令人繁琐，且难以在项目之间保持同步。虽然上例使用的是 `typeRoots`，但对于 `paths` 等其他选项也同样存在该问题。

为了解决该问题，TypeScript 5.5 引入了全新的模板变量 `${configDir}`。
当在 `tsconfig.json` 或 `jsconfig.json` 文件的特定路径字段中写入 `${configDir}` 时，该变量会在特定编译中被替换为当前配置文件的所在目录。
这意味着上述 `tsconfig.base.json` 可以重写为：

```json5
{
  compilerOptions: {
    typeRoots: [
      '${configDir}/node_modules/@types',
      '${configDir}/custom-types',
    ],
    outDir: '${configDir}/dist',
  },
}
```

现在，当项目继承此文件时，路径将相对于派生的 `tsconfig.json` 解析，而非共享的 `tsconfig.base.json` 文件。
这使得在跨项目共享配置文件时更加容易，并确保配置文件具备更高的可移植性。

如果你计划让某个 `tsconfig.json` 文件可被继承，请考虑是否应该将 `./` 改写为 `${configDir}`。

欲了解更多信息，请查阅[提案 Issue](https://github.com/microsoft/TypeScript/issues/57485) 以及[实现该特性的 Pull Request](https://github.com/microsoft/TypeScript/pull/58042)。

## 声明文件生成时参考 `package.json` 中的依赖

以往，TypeScript 经常会抛出类似如下的错误信息：

```
The inferred type of "X" cannot be named without a reference to "Y". This is likely not portable. A type annotation is necessary.
```

这通常是因为 TypeScript 生成声明文件时触及了程序中从未显式导入的文件内容。
如果最终生成的路径是相对路径，为此类文件生成导入可能会存在风险。
然而，对于在 `package.json` 的 `dependencies`（或 `peerDependencies`、`optionalDependencies`）中显式列出依赖的代码库，在特定的解析模式下生成此类导入应该是安全的。
因此在 TypeScript 5.5 中，对此类情况的处理更加宽容，该错误在很多场景下将不复存在。

更多关于该变动的详情，[请参见此 Pull Request](https://github.com/microsoft/TypeScript/issues/42873)。

## 编辑器与监视模式可靠性改进

TypeScript 添加了部分新功能并修复了现有逻辑，使 `--watch` 模式和 TypeScript 的编辑器集成更加稳定可靠。
这有望显著减少 TSServer 和编辑器的重启次数。

### 正确刷新配置文件中的编辑器错误

TypeScript 会针对 `tsconfig.json` 文件生成错误提示；
然而，这些错误实际上是在加载项目时生成的，且编辑器通常不会直接针对 `tsconfig.json` 文件请求这些错误信息。
虽然这听起来像是一个技术细节，但它导致的结果是：当 `tsconfig.json` 中的所有错误都被修复后，TypeScript 并未发出全新的空错误集合，导致用户除非重载编辑器，否则仍会看到过期的陈旧错误提示。

TypeScript 5.5 现在会主动发送事件来清除这些陈旧错误。
[详情请见此处](https://github.com/microsoft/TypeScript/pull/58120)。

### 更好地处理“删除后立即写入”的操作

某些工具在更新文件时不会选择直接覆盖，而是先删除文件，然后再从头创建新文件。
例如运行 `npm ci` 时就是如此。

虽然这对这些工具而言很高效，但在 TypeScript 的编辑器场景下却可能引发问题：删除被监视的文件可能会销毁该文件及其所有传递依赖项。
文件在短时间内被连续删除并重新创建，可能会导致 TypeScript 销毁整个项目然后从头重建。

TypeScript 5.5 采取了更加精细的处理策略：保留被删除项目的部分内容，直到捕获到新的文件创建事件。
这应该会使 `npm ci` 等操作在配合 TypeScript 使用时顺畅得多。
关于该策略的更多信息，[请参见此处](https://github.com/microsoft/TypeScript/pull/57492)。

### 在模块解析失败时跟踪符号链接（Symlinks）

当 TypeScript 解析模块失败时，仍需监视所有查找失败的路径，以防后续添加了该模块。
过去对于符号链接目录并没有执行此操作，在 Monorepo 类场景中，某一个项目完成构建后另一个项目可能无法感知到变动，从而引发稳定性问题。
该问题在 TypeScript 5.5 中已得到修复，这意味着你不再需要频繁重启编辑器。

[更多信息请参见此处](https://github.com/microsoft/TypeScript/pull/58139)。

### 项目引用参与自动导入补全

在配置了项目引用的工程中，自动导入不再需要先对依赖项目至少显式导入一次。
现在，只要在 `tsconfig.json` 的 `references` 字段中列出了该项目，自动导入补全就能直接跨项目生效。

[详情请见实现该特性的 Pull Request](https://github.com/microsoft/TypeScript/pull/55955)。

## 性能与体积优化

### 语言服务与公共 API 中的单态化对象

在 TypeScript 5.0 中，我们确保了 [`Node`](https://github.com/microsoft/TypeScript/pull/51682) 与 [`Symbol`](https://github.com/microsoft/TypeScript/pull/51880) 对象拥有一致的属性集以及一致的初始化顺序。
这样做有助于减少各项操作中的多态性（polymorphism），让运行时引擎能够更快地访问对象属性。

通过这一改进，编译器获得了令人瞩目的速度提升；
然而，当时的大多数优化都是针对数据结构的内部内存分配器进行的。
语言服务以及 TypeScript 的公共 API 对某些对象使用了另一套不同的分配器。
这曾让 TypeScript 编译器更加精简，因为仅用于语言服务的数据无需加载到编译器中。

在 TypeScript 5.5 中，语言服务与公共 API 也应用了相同的单态化（monomorphization）优化。
这意味着你的编辑器体验以及任何使用 TypeScript API 的构建工具都将获得可观的速度提升。
事实上，在我们的基准测试中，使用公共 TypeScript API 的分配器时**构建时间缩短了 5-8%**，且**语言服务操作速度提升了 10-20%**。
尽管这会带来一定的内存开销增加，但我们认为这一权衡是非常值得的，并希望后续能找到降低内存开销的方法。
现在的操作响应应该会流畅得多。

更多信息，[请参见此处变更](https://github.com/microsoft/TypeScript/pull/58045)。

### 控制流节点的单态化

在 TypeScript 5.5 中，控制流图（control flow graph）的节点已被单态化，以始终保持一致的结构形状。
通过这种方式，类型检查耗时通常能减少约 1%。

[更多信息请参见此处变更](https://github.com/microsoft/TypeScript/pull/57977)。

### 控制流图优化

在很多情况下，控制流分析会遍历一些无法提供任何新信息的节点。
我们观察到，如果某些节点的前驱节点（或“支配节点（dominators）”）中不存在任何提前终止或副作用，则这些节点完全可以直接跳过。
因此，TypeScript 现在利用这一特点来构建控制流图：直接链接到更早的*确实*能为控制流分析提供有用信息的节点。
这使得控制流图更加扁平，遍历效率更高。
该优化带来了可观的收益，在某些代码库中能减少高达 2% 的构建时间。

你可以[在此阅读更多内容](https://github.com/microsoft/TypeScript/pull/58013)。

### 在 `transpileModule` 与 `transpileDeclaration` 中跳过类型检查

TypeScript 的 `transpileModule` API 可用于将单个 TypeScript 文件的内容编译为 JavaScript。
类似地，`transpileDeclaration` API（见下文）可用于为单个 TypeScript 文件生成声明文件。
这些 API 存在的一个问题在于，TypeScript 内部在输出生成结果之前，会对整个文件内容执行一次完整的类型检查。
这是为了收集后续生成阶段所需的某些信息。

在 TypeScript 5.5 中，我们找到了一种避免进行完整检查的方法，仅在必要时按需延迟收集这些信息，且 `transpileModule` 和 `transpileDeclaration` 均已默认启用此项功能。
因此，集成了这些 API 的工具（例如配置了 `transpileOnly` 的 [ts-loader](https://www.npmjs.com/package/ts-loader) 以及 [ts-jest](https://www.npmjs.com/package/ts-jest)）应该能看到明显的速度提升。
在我们的测试中，[使用 `transpileModule` 通常能获得约 2 倍的构建速度提升](https://github.com/microsoft/TypeScript/pull/58364#issuecomment-2138580690)。

### 缩减 TypeScript 软件包体积

进一步借助 [5.0 中向模块化的迁移](https://devblogs.microsoft.com/typescript/typescripts-migration-to-modules/)，我们[让 `tsserver.js` 和 `typingsInstaller.js` 从公共 API 库导入，而不是各自生成独立的打包产物](https://github.com/microsoft/TypeScript/pull/55326)，从而大幅减小了 TypeScript 整体的软件包体积。

这使得 TypeScript 在磁盘上的体积从 30.2 MB 缩减至 20.4 MB，压缩打包后的体积也从 5.5 MB 缩减至 3.7 MB！

### 声明生成中的语法节点复用

作为支持 `isolatedDeclarations` 的一部分工作，我们大幅提升了 TypeScript 在生成声明文件时直接复用输入源代码的频率。

例如，假设你写了：

```ts
export const strBool: string | boolean = 'hello'
export const boolStr: boolean | string = 'world'
```

请注意，这两个联合类型是等价的，但联合成员的顺序不同。
在生成声明文件时，TypeScript 有两种等价的输出方案。

第一种是为每种类型使用一致的规范化表示形式：

```ts
export const strBool: string | boolean
export const boolStr: string | boolean
```

第二种是直接原样复用书写的类型标注：

```ts
export const strBool: string | boolean
export const boolStr: boolean | string
```

第二种方法通常更受青睐，原因如下：

- 许多等价的类型表示仍蕴含着某种代码意图，在声明文件中保留这些意图往往更好
- 重新生成类型的规范化表示开销较大，能省则省
- 用户手写的类型通常比自动生成的类型表示更简短

在 5.5 中，TypeScript 能够准确识别更多场景，安全且正确地将输入文件中原样书写的类型直接输出。
其中许多场景带来了无形的性能提升——以往 TypeScript 会生成全新的语法节点并将其序列化为字符串；
而现在，TypeScript 可以直接对原始语法节点进行操作，成本更低、速度更快。

### 缓存可辨识联合的上下文类型

当 TypeScript 查询对象字面量等表达式的上下文类型（contextual type）时，经常会遇到联合类型。
在这些情况下，TypeScript 会尝试根据具有确定值的已知属性（即可辨识属性 / discriminant properties）来过滤联合类型的成员。
这项工作可能相当耗时，尤其是当对象包含大量属性时。
在 TypeScript 5.5 中，[大部分计算结果会被缓存一次，这样 TypeScript 就无需为对象字面量中的每个属性都重复计算](https://github.com/microsoft/TypeScript/pull/58372)。
这项优化让编译 TypeScript 编译器本身的时间缩短了 250ms。

## 在 ECMAScript 模块中更轻松地调用 API

此前，如果你在 Node.js 中编写 ECMAScript 模块，是无法从 `typescript` 软件包中使用具名导入（named imports）的：

```ts
import { createSourceFile } from 'typescript' // ❌ 错误

import * as ts from 'typescript'
ts.createSourceFile // ❌ undefined???

ts.default.createSourceFile // ✅ 可以运行 —— 但很别扭！
```

这是因为 [cjs-module-lexer](https://github.com/nodejs/cjs-module-lexer) 无法识别 TypeScript 生成的 CommonJS 代码模式。
该问题现已修复，用户现在可以在 Node.js 的 ECMAScript 模块中直接从 TypeScript npm 包中使用具名导入：

```ts
import { createSourceFile } from 'typescript' // ✅ 现在可以正常运行！

import * as ts from 'typescript'
ts.createSourceFile // ✅ 现在可以正常运行！
```

更多信息，[请参见此处变更](https://github.com/microsoft/TypeScript/pull/57133)。

## `transpileDeclaration` API

TypeScript 的 API 提供了一个名为 `transpileModule` 的函数，旨在方便地编译单个 TypeScript 文件。
由于它无法访问整个*程序（Program）*上下文，其限制在于：如果代码违反了 `isolatedModules` 选项下的规则并报错，它可能无法生成正确的输出。

在 TypeScript 5.5 中，我们添加了一个类似的新 API：`transpileDeclaration`。
该 API 类似于 `transpileModule`，但专门用于根据输入的源代码文本生成单个*声明文件*。
与 `transpileModule` 一样，它同样无法访问完整的程序上下文，也有类似的限制：只有在输入代码符合全新的 `isolatedDeclarations` 选项且没有任何错误时，它才能生成准确的声明文件。

如有需要，可以使用该函数在 `isolatedDeclarations` 模式下跨所有文件并行生成声明文件。

更多信息，[请参见此处的实现](https://github.com/microsoft/TypeScript/pull/58261)。

## 显著的行为变更

本节列出了一系列值得关注的变更，在版本升级时应予以了解和注意。
这些内容可能包括废弃项、移除项以及新的语法限制，也可能包含功能改进类的 Bug 修复（但可能会引入新的错误并影响现有构建）。

### 禁用 TypeScript 5.0 中废弃的特性

TypeScript 5.0 废弃了以下编译选项和行为：

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
- 隐式与操作系统相关的 `newLine`

为了继续使用上述已废弃的选项，使用 TypeScript 5.0 及更高版本的开发者此前必须指定一个名为 `ignoreDeprecations` 且值为 `"5.0"` 的新选项。

在 TypeScript 5.5 中，这些选项不再产生任何效果。
为了保证平滑升级，你仍可以在 tsconfig 中指定它们，但在 TypeScript 6.0 中指定它们将会报错。
另请参阅概述了我们废弃策略的 [Flag Deprecation Plan](https://github.com/microsoft/TypeScript/issues/51000)。

[GitHub 上提供了关于这些废弃计划的更多信息](https://github.com/microsoft/TypeScript/issues/51909)，其中包含如何最好地调整代码库的建议。

### `lib.d.ts` 变更

为 DOM 生成的类型可能会对你代码库的类型检查产生影响。
欲了解更多信息，[请参见 TypeScript 5.5 的 DOM 更新](https://github.com/microsoft/TypeScript/pull/58211)。

### 更严格的装饰器语法解析

自 TypeScript 最初引入对装饰器的支持以来，该规范提案的语法规则已变得更加严格。
TypeScript 现在对其允许的形式进行了更严格的约束。
虽然这种情况较为少见，但部分既有的装饰器可能需要添加括号以避免报错。

```ts
class DecoratorProvider {
  decorate(...args: any[]) {}
}

class D extends DecoratorProvider {
  m() {
    class C {
      @(super.decorate) // ❌ 错误
      method1() {}

      @(super.decorate) // ✅ 正常
      method2() {}
    }
  }
}
```

更多关于该变更的信息，[请参见此处](https://github.com/microsoft/TypeScript/pull/57749)。

### `undefined` 不再可用作可定义的类型名称

TypeScript 一直以来都禁止类型别名与内置类型名称冲突：

```ts
// 非法
type null = any;
// 非法
type number = any;
// 非法
type object = any;
// 非法
type any = any;
```

由于一个 Bug，该逻辑此前并未应用到内置的 `undefined` 类型上。
在 5.5 中，这现在会被正确识别为错误：

```ts
// 现在同样非法
type undefined = any
```

事实上，对名为 `undefined` 的类型别名的裸引用从一开始就无法正常工作。
你可以定义它们，但无法将它们作为未限定作用域的类型名称来使用。

```ts
export type undefined = string
export const m: undefined = ''
//           ^
// 在 5.4 及更早版本中报错 —— 甚至根本没有参考 'undefined' 的局部定义。
```

更多信息，[请参见此处变更](https://github.com/microsoft/TypeScript/pull/57575)。

### 简化声明生成中的引用指令（Reference Directive）

在生成声明文件时，如果 TypeScript 认为需要引用指令（reference directive），便会自动合成一条。
例如，所有 Node.js 模块都是环境声明（ambiently declared），因此无法仅靠模块解析加载。
类似如下的文件：

```tsx
import path from 'path'
export const myPath = path.parse(__filename)
```

以往会生成如下声明文件：

```tsx
/// <reference types="node" />
import path from 'path'
export declare const myPath: path.ParsedPath
```

即使该引用指令从未在原始源码中出现过。

类似地，TypeScript 还会*移除*它认为不需要作为输出一部分的引用指令。
例如，假设我们有一条指向 `jest` 的引用指令，但在生成声明文件时并不需要它，TypeScript 就会直接将其丢弃。
因此在以下示例中：

```tsx
/// <reference types="jest" />
import path from 'path'
export const myPath = path.parse(__filename)
```

TypeScript 依然会生成：

```tsx
/// <reference types="node" />
import path from 'path'
export declare const myPath: path.ParsedPath
```

在开发 `isolatedDeclarations` 的过程中，我们意识到对于任何尝试在不进行类型检查、或不依赖单文件以外上下文的情况下实现声明生成器的工具而言，这种逻辑是站不住脚的。
从用户的角度来看，这种行为也很难理解：引用指令是否会出现在生成的文件中显得很不一致且难以预测，除非你完全清楚类型检查期间发生了什么。
为了防止启用 `isolatedDeclarations` 时声明生成的结果出现差异，我们必须改变现有的生成机制。

通过[实验探究](https://github.com/microsoft/TypeScript/pull/57569)，我们发现 TypeScript 自动合成引用指令的场景几乎全是为了引入 `node` 或 `react`。
在这些场景中，通常预期下游用户已经通过 tsconfig.json 的 `"types"` 或库导入引用了这些类型，因此不再自动合成这些引用指令基本不会对任何人造成破坏。
值得一提的是，`lib.d.ts` 本身就是这样工作的：当一个模块导出 `WeakMap` 时，TypeScript 并不会自动合成对 `lib="es2015"` 的引用，而是假定下游用户已经在其环境中包含了该库。

而对于库作者手写的引用指令（非自动合成），[进一步的实验](https://github.com/microsoft/TypeScript/pull/57656)表明它们几乎全被移除了，从未出现在输出中。
绝大多数被保留下来的引用指令反而存在问题，且很可能并非作者本意希望保留的。

基于这些实验结果，我们决定在 TypeScript 5.5 中大幅简化声明生成中的引用指令处理。
采用更一致的策略将有助于库作者和使用者更好地掌控其声明文件。

不再自动合成引用指令。
用户手写的引用指令也不会再被保留，除非添加了全新的 `preserve="true"` 属性进行标注。
具体而言，对于如下输入文件：

```tsx
/// <reference types="some-lib" preserve="true" />
/// <reference types="jest" />
import path from 'path'
export const myPath = path.parse(__filename)
```

将生成：

```tsx
/// <reference types="some-lib" preserve="true" />
import path from 'path'
export declare const myPath: path.ParsedPath
```

添加 `preserve="true"` 对旧版本的 TypeScript 是向后兼容的，因为未知的属性会被直接忽略。

此项变更还带来了性能提升：在我们的基准测试中，启用了声明生成的项目在生成阶段（emit stage）获得了 1-4% 的性能改进。
