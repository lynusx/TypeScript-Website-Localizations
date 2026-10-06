---
title: TypeScript 4.9
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-4-9.html
oneline: TypeScript 4.9 发布说明
---

## `satisfies` 运算符

TypeScript 开发者经常面临两难境地：我们既希望确保某个表达式*匹配*某种类型，又希望保留该表达式*最具体*的类型用于类型推断。

例如：

```ts
// 每个属性可以是字符串或 RGB 元组。
const palette = {
  red: [255, 0, 0],
  green: '#00ff00',
  bleu: [0, 0, 255],
  //  ^^^^ sacrebleu - we've made a typo!
}

// 我们希望能够对 'green' 使用字符串方法...
const greenNormalized = palette.green.toUpperCase()
```

注意，我们把属性名误写成了 `bleu`，而原本应该写成 `blue`。我们可以尝试在 `palette` 上添加类型标注来捕获 `bleu` 拼写错误，但这样会丢失各个属性的具体类型信息：

```ts
type Colors = 'red' | 'green' | 'blue'

type RGB = [red: number, green: number, blue: number]

const palette: Record<Colors, string | RGB> = {
  red: [255, 0, 0],
  green: '#00ff00',
  bleu: [0, 0, 255],
  //  ~~~~ 现在正确检测到了该拼写错误
}

// 但我们现在在这里遇到了一个不希望看到的错误 —— 'palette.green' “可能”是 RGB 类型，
// 且类型 'string | RGB' 上不存在属性 'toUpperCase'。
const greenNormalized = palette.green.toUpperCase()
```

全新的 `satisfies` 运算符允许我们验证表达式的类型是否匹配某种类型，同时不改变该表达式本身的推断类型结果。例如，我们可以使用 `satisfies` 来验证 `palette` 的所有属性是否与 `string | number[]` 兼容：

```ts
type Colors = 'red' | 'green' | 'blue'

type RGB = [red: number, green: number, blue: number]

const palette = {
  red: [255, 0, 0],
  green: '#00ff00',
  bleu: [0, 0, 255],
  //  ~~~~ 该拼写错误现已被捕获！
} satisfies Record<Colors, string | RGB>

// toUpperCase() 方法仍然可以访问！
const greenNormalized = palette.green.toUpperCase()
```

`satisfies` 可用于捕获许多潜在错误。例如，我们可以确保对象具备某种类型的*所有*键，且不多不少：

```ts
type Colors = 'red' | 'green' | 'blue'

// 确保我们拥有的键与 'Colors' 完全一致。
const favoriteColors = {
  red: 'yes',
  green: false,
  blue: 'kinda',
  platypus: false,
  //  ~~~~~~~~~~ 错误 - "platypus" 从未列在 'Colors' 中。
} satisfies Record<Colors, unknown>

// 关于 'red'、'green' 和 'blue' 属性的所有信息均被保留。
const g: boolean = favoriteColors.green
```

或者，我们可能不关心属性名称是否匹配，但关心各个属性的类型。在这种情况下，我们也可以确保对象的所有属性值都符合某种类型：

```ts
type RGB = [red: number, green: number, blue: number]

const palette = {
  red: [255, 0, 0],
  green: '#00ff00',
  blue: [0, 0],
  //    ~~~~~~ 错误！
} satisfies Record<string, string | RGB>

// 每个属性的信息仍被保留。
const redComponent = palette.red.at(0)
const greenNormalized = palette.green.toUpperCase()
```

欲了解更多示例，你可以查看[提出该特性的 issue](https://github.com/microsoft/TypeScript/issues/47920) 和[实现该功能的 Pull Request](https://github.com/microsoft/TypeScript/pull/46827)。我们要向 [Oleksandr Tarasiuk](https://github.com/a-tarasyuk) 致谢，是他与我们一起实现并迭代了这一特性。

## 使用 `in` 运算符收窄未列出的属性

作为开发者，我们经常需要处理运行时无法完全确定的值。事实上，无论是在接收服务器响应还是读取配置文件时，我们通常都无法预先得知某个属性是否存在。JavaScript 的 `in` 运算符可以检查对象上是否存在某个属性。

此前，TypeScript 允许我们收窄并排除那些未显式列出该属性的类型：

```ts
interface RGB {
  red: number
  green: number
  blue: number
}

interface HSV {
  hue: number
  saturation: number
  value: number
}

function setColor(color: RGB | HSV) {
  if ('hue' in color) {
    // 'color' 现在具有类型 HSV
  }
  // ...
}
```

在这里，类型 `RGB` 没有列出 `hue`，因而被排除收窄，留给我们的类型就是 `HSV`。

但是，如果所有类型都没有列出给定属性呢？在那些情况下，语言并没有提供太多帮助。来看以下 JavaScript 示例：

```js
function tryGetPackageName(context) {
  const packageJSON = context.packageJSON
  // 检查是否为对象。
  if (packageJSON && typeof packageJSON === 'object') {
    // 检查它是否具有字符串类型的 name 属性。
    if ('name' in packageJSON && typeof packageJSON.name === 'string') {
      return packageJSON.name
    }
  }

  return undefined
}
```

将其重写为规范的 TypeScript 代码只需为 `context` 定义并使用一个类型；然而在旧版本的 TypeScript 中，为 `packageJSON` 属性选择一个像 `unknown` 这样的安全类型会导致问题：

```ts
interface Context {
  packageJSON: unknown
}

function tryGetPackageName(context: Context) {
  const packageJSON = context.packageJSON
  // 检查是否为对象。
  if (packageJSON && typeof packageJSON === 'object') {
    // 检查它是否具有字符串类型的 name 属性。
    if ('name' in packageJSON && typeof packageJSON.name === 'string') {
      //                                              ~~~~
      // 错误！类型 'object' 上不存在属性 'name'。
      return packageJSON.name
      //                     ~~~~
      // 错误！类型 'object' 上不存在属性 'name'。
    }
  }

  return undefined
}
```

这是因为虽然 `packageJSON` 的类型从 `unknown` 收窄为了 `object`，但 `in` 运算符此前只会严格收窄到实际定义了所检查属性的类型上。因此，`packageJSON` 的类型仍然是 `object`。

TypeScript 4.9 让 `in` 运算符在收窄根本*没有*列出该属性的类型时变得更加强大。语言不再保持它们的原样，而是会将它们的类型与 `Record<"被检查的属性名", unknown>` 进行交叉。

因此在我们的示例中，`packageJSON` 的类型将从 `unknown` 收窄为 `object`，再收窄为 `object & Record<"name", unknown>`。这使我们能够直接访问 `packageJSON.name` 并对其独立进行类型收窄：

```ts
interface Context {
  packageJSON: unknown
}

function tryGetPackageName(context: Context): string | undefined {
  const packageJSON = context.packageJSON
  // 检查是否为对象。
  if (packageJSON && typeof packageJSON === 'object') {
    // 检查它是否具有字符串类型的 name 属性。
    if ('name' in packageJSON && typeof packageJSON.name === 'string') {
      // 正常工作！
      return packageJSON.name
    }
  }

  return undefined
}
```

TypeScript 4.9 还收紧了围绕 `in` 运算符使用的若干检查，确保左操作数可赋值给 `string | number | symbol` 类型，而右操作数可赋值给 `object` 类型。这有助于确保我们使用的是有效的属性键，而不会意外去检查原始类型。

欲了解更多信息，请[阅读实现该功能的 Pull Request](https://github.com/microsoft/TypeScript/pull/50666)。

## 类中的自动访问器（Auto-Accessors）

TypeScript 4.9 支持 ECMAScript 中即将到来的“自动访问器（auto-accessors）”特性。自动访问器的声明方式与类属性相同，只是它们使用 `accessor` 关键字进行声明：

```ts
class Person {
  accessor name: string

  constructor(name: string) {
    this.name = name
  }
}
```

在底层，这些自动访问器会被“脱糖（de-sugar）”为一个 `get` 和 `set` 访问器以及一个外部无法访问的私有属性：

```ts
class Person {
  #__name: string

  get name() {
    return this.#__name
  }
  set name(value: string) {
    this.#__name = value
  }

  constructor(name: string) {
    this.name = name
  }
}
```

你可以在[原始 PR 中了解关于自动访问器的更多信息](https://github.com/microsoft/TypeScript/pull/49705)。

## 对 `NaN` 的相等性检查

对 JavaScript 开发者来说，使用内置相等运算符与 `NaN` 值进行比较是一个常见的陷阱。

背景知识是：`NaN` 是一个表示“非数字（Not a Number）”的特殊数值。没有任何值与 `NaN` 相等——即使是 `NaN` 本身也不相等！

```js
console.log(NaN == 0) // false
console.log(NaN === 0) // false

console.log(NaN == NaN) // false
console.log(NaN === NaN) // false
```

但对称地，*所有值*都始终不等于 `NaN`：

```js
console.log(NaN != 0) // true
console.log(NaN !== 0) // true

console.log(NaN != NaN) // true
console.log(NaN !== NaN) // true
```

从技术上讲，这并不是 JavaScript 独有的问题，因为任何采用 IEEE-754 浮点数的语言都具有相同的行为；但 JavaScript 的主要数值类型是浮点数，且 JavaScript 中的数字解析常常会产生 `NaN`。因此，检查 `NaN` 相当普遍，正确的方法是使用 [`Number.isNaN`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/isNaN)——*但是*正如我们提到的，许多人会不小心用 `someValue === NaN` 进行比较。

TypeScript 现在会对与 `NaN` 的直接比较报错，并建议使用 `Number.isNaN` 的某种变体来替代：

```ts
function validate(someValue: number) {
  return someValue !== NaN
  //     ~~~~~~~~~~~~~~~~~
  // 错误：此条件将始终返回 'true'。
  //        你的意思是 '!Number.isNaN(someValue)' 吗？
}
```

我们相信这项更改能切实帮助捕获初学者的错误，类似于目前 TypeScript 对对象和数组字面量比较所给出的错误提示。

感谢 [Oleksandr Tarasiuk](https://github.com/a-tarasyuk) [贡献了这项检查](https://github.com/microsoft/TypeScript/pull/50626)。

## 文件监视现采用文件系统事件

在早期版本中，TypeScript 在监视单个文件时严重依赖*轮询（polling）*。使用轮询策略意味着定期检查文件的状态是否有更新。在 Node.js 中，[`fs.watchFile`](https://nodejs.org/docs/latest-v18.x/api/fs.html#fswatchfilefilename-options-listener) 是获取轮询文件监视器的内置方式。虽然轮询在不同平台和文件系统之间往往更具可预测性，但这意味着即便没有任何变更，CPU 也必须定期被打断以检查文件更新。对于几十个文件，这可能并不明显；但对于包含大量文件的大型项目——或者在 `node_modules` 中有大量文件时——这会变成资源消耗大户。

一般而言，更好的方案是使用文件系统事件。我们可以声明自己对特定文件的更新感兴趣，并在这些文件*确实*发生变化时提供回调，而不是主动轮询。大多数主流现代平台都提供了类似 `CreateIoCompletionPort`、`kqueue`、`epoll` 和 `inotify` 的设施和 API。Node.js 主要通过提供 [`fs.watch`](https://nodejs.org/docs/latest-v18.x/api/fs.html#fswatchfilename-options-listener) 来将这些机制抽象化。文件系统事件通常工作良好，但在使用它们以及使用 `fs.watch` API 时有[许多需要注意的陷阱](https://nodejs.org/docs/latest-v18.x/api/fs.html#caveats)。监视器需要谨慎考虑 [inode 监视](https://nodejs.org/docs/latest-v18.x/api/fs.html#inodes)、[在某些文件系统上的不可用性](https://nodejs.org/docs/latest-v18.x/api/fs.html#availability)（例如网络文件系统）、是否支持递归文件监视、目录重命名是否会触发事件，甚至是文件监视器句柄耗尽问题！换言之，这并非没有代价，尤其是在寻求跨平台兼容方案时。

因此，我们此前默认选择了折中的最低标准：轮询。虽非全部情况，但大多数时候如此。

随着时间的推移，我们提供了[选择其他文件监视策略](https://www.typescriptlang.org/docs/handbook/configuring-watch.html)的能力。这使我们能够收集反馈并加强文件监视实现，以应对大多数平台特有的坑。随着 TypeScript 需要扩展到更大的代码库并在这一领域不断成熟，我们认为将默认策略切换为文件系统事件是一项值得的投入。

在 TypeScript 4.9 中，文件监视默认由文件系统事件驱动，仅在我们无法设置基于事件的监视器时才会回退到轮询。对于大多数开发者而言，无论是在 `--watch` 模式下运行，还是在使用 Visual Studio 或 VS Code 等基于 TypeScript 的编辑器时，这都应该带来占用资源大幅减少的体验。

[文件监视的工作机制仍可通过环境变量和 `watchOptions` 进行配置](https://www.typescriptlang.org/docs/handbook/configuring-watch.html)——且[某些编辑器（如 VS Code）可独立支持 `watchOptions`](https://code.visualstudio.com/docs/getstarted/settings#:~:text=typescript%2etsserver%2ewatchOptions)。使用源码位于网络文件系统（如 NFS 和 SMB）上的特殊配置的开发者可能需要选择恢复旧行为；不过，如果服务器具备合理的计算能力，启用 SSH 并远程运行 TypeScript 以直接访问本地文件可能会是更好的选择。VS Code 拥有丰富的[远程扩展](https://marketplace.visualstudio.com/search?term=remote&target=VSCode&category=All%20categories&sortBy=Relevance)以简化这一过程。

你可以在 GitHub 上[阅读有关此项变更的更多信息](https://github.com/microsoft/TypeScript/pull/50366)。

## 编辑器的“移除未使用的导入”与“对导入排序”命令

此前，TypeScript 仅支持两个用于管理导入的编辑器命令。看以下示例代码：

```ts
import { Zebra, Moose, HoneyBadger } from './zoo'
import { foo, bar } from './helper'

let x: Moose | HoneyBadger = foo()
```

第一个命令是“整理导入（Organize Imports）”，它会删除未使用的导入，然后对剩余的导入进行排序。它会将文件重写为如下所示：

```ts
import { foo } from './helper'
import { HoneyBadger, Moose } from './zoo'

let x: Moose | HoneyBadger = foo()
```

在 TypeScript 4.3 中，我们引入了一个名为“对导入排序（Sort Imports）”的命令，它*只*对文件中的导入进行排序，而不删除它们——它会将文件重写如下：

```ts
import { bar, foo } from './helper'
import { HoneyBadger, Moose, Zebra } from './zoo'

let x: Moose | HoneyBadger = foo()
```

“对导入排序”的局限在于，在 Visual Studio Code 中，该功能仅作为保存时触发的命令可用，而无法手动触发。

TypeScript 4.9 补全了另一半功能，现在提供了“移除未使用的导入（Remove Unused Imports）”。TypeScript 现在会移除未使用的导入名称和语句，但在其他方面保留其相对顺序不变：

```ts
import { Moose, HoneyBadger } from './zoo'
import { foo } from './helper'

let x: Moose | HoneyBadger = foo()
```

该特性适用于所有希望使用任一命令的编辑器；尤其值得一提的是，Visual Studio Code（1.73 及更高版本）已内置支持，*并且*会通过其命令面板（Command Palette）公开这些命令。更喜欢使用细粒度的“移除未使用的导入”或“对导入排序”命令的用户，可以根据需要将“整理导入”的快捷键重新绑定到它们上。

你可以[在此查看该特性的具体细节](https://github.com/microsoft/TypeScript/pull/50931)。

## 在 `return` 关键字上支持“跳转到定义”

在编辑器中，当在 `return` 关键字上执行“跳转到定义”时，TypeScript 现在会跳转到对应函数的顶部。这有助于快速了解某个 `return` 属于哪个函数。

我们预计 TypeScript 会将此功能扩展到更多关键字，[例如 `await` 和 `yield`](https://github.com/microsoft/TypeScript/issues/51223) 或 [`switch`、`case` 和 `default`](https://github.com/microsoft/TypeScript/issues/51225)。

[该特性的实现](https://github.com/microsoft/TypeScript/pull/51227)感谢 [Oleksandr Tarasiuk](https://github.com/a-tarasyuk)。

## 性能改进

TypeScript 带来了一些小巧但显著的性能改进。

首先，TypeScript 的 `forEachChild` 函数经过重写，采用了函数表查找而非对所有语法节点使用 `switch` 语句。`forEachChild` 是编译器中遍历语法节点的核心工作函数，在编译器的绑定阶段以及语言服务的部分功能中被大量使用。对 `forEachChild` 的重构使我们在绑定阶段和各项语言服务操作中耗费的时间减少了高达 20%。

当我们发现 `forEachChild` 的这项性能提升后，我们在 `visitEachChild` 上也进行了尝试（我们在编译器和语言服务中用该函数转换节点）。同样的重构使生成项目输出所花费的时间减少了高达 3%。

对 `forEachChild` 的最初探索是[受到一篇文章的启发](https://artemis.sh/2022/08/07/emulating-calculators-fast-in-js.html)，作者是 [Artemis Everfree](https://artemis.sh/)。虽然我们有理由相信性能提升的根本原因可能更多与函数体积/复杂度有关，而非博文中所描述的问题，但我们很庆幸能从这次经历中汲取经验，并尝试通过相对轻量的重构让 TypeScript 变得更快。

最后，TypeScript 优化了在条件类型的真分支中保留类型信息的方式。对于形如下述类型：

```ts
interface Zoo<T extends Animal> {
  // ...
}

type MakeZoo<A> = A extends Animal ? Zoo<A> : never
```

TypeScript 在检查 `Zoo<A>` 是否合法时必须“记住” `A` 也必须是一个 `Animal`。这基本上是通过创建一个特殊的类型来实现的，该类型此前用于保存 `A` 与 `Animal` 的交叉；然而，TypeScript 之前是急切（eagerly）创建该类型的，这并不总是必需的。此外，我们类型检查器中的某些缺陷代码阻碍了这些特殊类型的简化。TypeScript 现在会将这些类型的交叉推迟到必要时进行。对于大量使用条件类型的代码库，你可能会看到 TypeScript 显著加速；而在我们的性能测试套件中，我们观察到类型检查时间有了约 3% 的温和改善。

你可以在各自的 Pull Request 中了解关于这些优化的更多信息：

- [基于跳转表的 `forEachChild`](https://github.com/microsoft/TypeScript/pull/50225)
- [基于跳转表的 `visitEachChild`](https://github.com/microsoft/TypeScript/pull/50266)
- [优化替换类型（substitution types）](https://github.com/microsoft/TypeScript/pull/50397)

## 正确性修复与破坏性变更

### `lib.d.ts` 更新

虽然 TypeScript 竭力避免重大破坏性变更，但内置库中哪怕很小的改动也可能引发问题。我们预计 DOM 和 `lib.d.ts` 的更新不会带来重大破坏，但可能会有一些细微影响。

### `Promise.resolve` 的更优类型定义

`Promise.resolve` 现在使用 `Awaited` 类型来解包传入给它的 Promise-like 类型。这意味着它能更频繁地返回正确的 `Promise` 类型，但如果现有代码期望返回 `any` 或 `unknown` 而非 `Promise`，这种更精准的类型可能会破坏现有代码。欲了解更多信息，请[查看原始变更](https://github.com/microsoft/TypeScript/pull/33074)。

### JavaScript 代码生成不再省略导入语句

当 TypeScript 最初支持 JavaScript 的类型检查和编译时，它无意中支持了一项名为“导入省略（import elision）”的功能。简而言之，如果某个导入未作为值使用，或者编译器能够检测到该导入在运行时不指代任何值，编译器就会在生成代码时丢弃该导入。

这种行为值得商榷，尤其是检测导入是否不指代值这一项，因为这意味着 TypeScript 必须信任有时并不准确的声明文件。因此，TypeScript 现在会在 JavaScript 文件中保留导入语句。

```js
// 输入：
import { someValue, SomeClass } from 'some-module'

/** @type {SomeClass} */
let val = someValue

// 此前的输出：
import { someValue } from 'some-module'

/** @type {SomeClass} */
let val = someValue

// 当前的输出：
import { someValue, SomeClass } from 'some-module'

/** @type {SomeClass} */
let val = someValue
```

更多信息可在[实现该功能的变更](https://github.com/microsoft/TypeScript/pull/50404)中查看。

### `exports` 的优先级高于 `typesVersions`

此前，在 `--moduleResolution node16` 下通过 `package.json` 进行解析时，TypeScript 错误地将 `typesVersions` 字段的优先级排在 `exports` 字段之前。如果这一变更影响了你的库，你可能需要在 `package.json` 的 `exports` 字段中添加 `types@` 版本选择器：

```diff
  {
      "type": "module",
      "main": "./dist/main.js"
      "typesVersions": {
          "<4.8": { ".": ["4.8-types/main.d.ts"] },
          "*": { ".": ["modern-types/main.d.ts"] }
      },
      "exports": {
          ".": {
+             "types@<4.8": "./4.8-types/main.d.ts",
+             "types": "./modern-types/main.d.ts",
              "import": "./dist/main.js"
          }
      }
  }
```

欲了解更多信息，请[参阅此 Pull Request](https://github.com/microsoft/TypeScript/pull/50890)。

## `SubstitutionType` 上的 `substitute` 被替换为 `constraint`

作为替换类型优化的一部分，`SubstitutionType` 对象不再包含表示有效替换（通常是基础类型与隐式约束的交叉）的 `substitute` 属性——相反，它们仅包含 `constraint` 属性。

欲了解更多详情，请[在原始 Pull Request 中阅读更多内容](https://github.com/microsoft/TypeScript/pull/50397)。
