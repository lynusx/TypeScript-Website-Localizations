---
title: TypeScript 4.6
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-4-6.html
oneline: TypeScript 4.6 发布说明
---

## 允许在构造函数中的 `super()` 之前执行代码

在 JavaScript 类中，在引用 `this` 之前必须先调用 `super()`。
TypeScript 同样强制要求这一点，但在*如何*保证这一规则方面此前过于严苛。
在 TypeScript 早期版本中，如果包含类具有任何属性初始值设定项，那么在构造函数开头包含*任何*代码都会报错：

```ts
class Base {
  // ...
}

class Derived extends Base {
  someProperty = true

  constructor() {
    // 错误！
    // 必须先调用 'super()'，因为它需要初始化 'someProperty'。
    doSomeStuff()
    super()
  }
}
```

这虽然能以低成本的方式检查 `super()` 是否在引用 `this` 之前被调用，但最终却拒绝了许多有效的代码。
TypeScript 4.6 现在对这一检查放宽了许多，允许在 `super()` 之前运行其他代码，同时仍然保证 `super()` 在顶层且在对 `this` 进行任何引用之前执行。

我们由衷感谢 [Joshua Goldberg](https://github.com/JoshuaKGoldberg) [耐心与我们合作落地了该改动](https://github.com/microsoft/TypeScript/pull/29374)！

## 可辨识联合在解构时的控制流分析改进

TypeScript 能够基于所谓的判别属性（discriminant property）进行类型窄化。
例如，在下面的代码片段中，每次我们对 `kind` 的值进行检查时，TypeScript 都能对 `action` 的类型进行窄化：

```ts
type Action =
  | { kind: 'NumberContents'; payload: number }
  | { kind: 'StringContents'; payload: string }

function processAction(action: Action) {
  if (action.kind === 'NumberContents') {
    // 此时 `action.payload` 是一个数字。
    let num = action.payload * 2
    // ...
  } else if (action.kind === 'StringContents') {
    // 此时 `action.payload` 是一个字符串。
    const str = action.payload.trim()
    // ...
  }
}
```

这使我们能够处理可容纳不同数据的对象，并通过一个公共字段获知这些对象具体持有*哪种*数据。

这在 TypeScript 中非常常见；然而，根据个人编码习惯，你可能更希望解构上述示例中的 `kind` 和 `payload`。
例如写成如下形式：

```ts
type Action =
  | { kind: 'NumberContents'; payload: number }
  | { kind: 'StringContents'; payload: string }

function processAction(action: Action) {
  const { kind, payload } = action
  if (kind === 'NumberContents') {
    let num = payload * 2
    // ...
  } else if (kind === 'StringContents') {
    const str = payload.trim()
    // ...
  }
}
```

此前 TypeScript 会在此处报错——一旦 `kind` 和 `payload` 从同一个对象解构到局部变量中，它们就会被视为完全独立的变量。

在 TypeScript 4.6 中，这可以直接正常工作！

当把单独的属性解构到 `const` 声明中，或者将参数解构为从未被重新赋值的变量时，TypeScript 会检查解构的类型是否为可辨识联合。
如果是，TypeScript 现在可以根据对其他变量的检查来窄化相关变量的类型。
因此在我们的示例中，对 `kind` 的检查会自动窄化 `payload` 的类型。

更多信息请参阅 [实现该分析的 Pull Request](https://github.com/microsoft/TypeScript/pull/46266)。

## 改进的递归深度检查

由于 TypeScript 构建在具有泛型能力的结构化类型系统之上，因此面临着一些有趣的挑战。

在结构化类型系统中，对象类型的兼容性取决于它们所拥有的成员：

```ts
interface Source {
  prop: string
}

interface Target {
  prop: number
}

function check(source: Source, target: Target) {
  target = source
  // 错误！
  // 类型 'Source' 不能分配给类型 'Target'。
  //   属性 'prop' 的类型不兼容。
  //     类型 'string' 不能分配给类型 'number'。
}
```

请注意，`Source` 是否与 `Target` 兼容取决于它们的*属性*是否可赋值。
在这个例子中，就是属性 `prop`。

当把泛型引入该机制时，就会引发一些更难回答的问题。
例如，在下述情况下，`Source<string>` 是否可以赋值给 `Target<number>` 呢？

```ts
interface Source<T> {
  prop: Source<Source<T>>
}

interface Target<T> {
  prop: Target<Target<T>>
}

function check(source: Source<string>, target: Target<number>) {
  target = source
}
```

为了回答这个问题，TypeScript 需要检查 `prop` 的类型是否兼容。
这又引出了另一个问题：`Source<Source<string>>` 是否可以赋值给 `Target<Target<number>>`？
为此，TypeScript 会检查*那些*类型的 `prop` 是否兼容，并最终检查 `Source<Source<Source<string>>>` 是否可以赋值给 `Target<Target<Target<number>>>`。
依此类推，你可能会注意到随着深入挖掘，类型会发生无限展开。

TypeScript 在此处采用了一些启发式规则——如果在达到某个深度检查后，类型*看似*处于无限展开状态，那么它就会认为这些类型*可能*是兼容的。
这通常足够应付大多数情况，但尴尬的是，这也留下了一些无法捕获的假阴性（false-negatives）问题：

```ts
interface Foo<T> {
  prop: T
}

declare let x: Foo<Foo<Foo<Foo<Foo<Foo<string>>>>>>
declare let y: Foo<Foo<Foo<Foo<Foo<string>>>>>

x = y
```

人类读者一眼就能看出上述示例中 `x` 和 `y` 应当是不兼容的。
尽管类型嵌套很深，但这仅仅是它们被声明出的形式而已。
启发式规则本意是捕获那些在类型推演过程中生成的深层嵌套类型，而不是开发者自己亲手写出的类型。

TypeScript 4.6 现在能够区分这些情况，并在上面的最后一个示例中正确报错。
此外，由于该语言不再担心显式手写类型所带来的假阳性，TypeScript 可以更早得出某个类型处于无限展开状态的结论，从而在类型兼容性检查中节省大量工作。
因此，DefinitelyTyped 上的 `redux-immutable`、`react-lazylog` 和 `yup` 等库的类型检查耗时减少了 50%。

你可能已经拥有此项改动，因为它已被挑选（cherry-pick）合入到了 TypeScript 4.5.3 中，但它依然是 TypeScript 4.6 的一项重要特性，你可以在 [此处](https://github.com/microsoft/TypeScript/pull/46599) 阅读更多内容。

## 索引访问推断改进

TypeScript 现在能够正确推断立即对映射对象类型进行索引的索引访问类型：

```ts
interface TypeMap {
  number: number
  string: string
  boolean: boolean
}

type UnionRecord<P extends keyof TypeMap> = {
  [K in P]: {
    kind: K
    v: TypeMap[K]
    f: (p: TypeMap[K]) => void
  }
}[P]

function processRecord<K extends keyof TypeMap>(record: UnionRecord<K>) {
  record.f(record.v)
}

// 该调用过去曾存在问题 —— 现在可以正常工作！
processRecord({
  kind: 'string',
  v: 'hello!',

  // 'val' 过去曾隐式具有类型 'string | number | boolean'，
  // 但现在被正确推断为仅 'string'。
  f: (val) => {
    console.log(val.toUpperCase())
  },
})
```

该模式此前虽然已经得到支持，并且允许 TypeScript 理解对 `record.f(record.v)` 的调用是合法的，但在之前，调用 `processRecord` 时会对 `val` 给出较差的推断结果。

TypeScript 4.6 对此进行了改进，因此在调用 `processRecord` 时不再需要任何类型断言。

更多详细信息，请参阅 [该 Pull Request](https://github.com/microsoft/TypeScript/pull/47109)。

## 针对相关参数的控制流分析

函数签名可以声明一个剩余参数，其类型为元组的可辨识联合：

```ts
function func(...args: ['str', string] | ['num', number]) {
  // ...
}
```

这意味着 `func` 的参数完全取决于第一个参数。
当第一个参数是字符串 `"str"` 时，其第二个参数必须是 `string`。
当第一个参数是字符串 `"num"` 时，其第二个参数必须是 `number`。

在 TypeScript 从此类签名推断函数类型的情况下，TypeScript 现在可以对彼此相互依赖的参数进行类型窄化：

```ts
type Func = (...args: ['a', number] | ['b', string]) => void

const f1: Func = (kind, payload) => {
  if (kind === 'a') {
    payload.toFixed() // 'payload' 被收窄为 'number'
  }
  if (kind === 'b') {
    payload.toUpperCase() // 'payload' 被收窄为 'string'
  }
}

f1('a', 42)
f1('b', 'hello')
```

更多信息请参阅 [GitHub 上的改动](https://github.com/microsoft/TypeScript/pull/47190)。

## `--target es2022`

TypeScript 的 `--target` 选项现已支持 `es2022`。
这意味着类字段（class fields）等特性现在拥有了一个可以保留它们的稳定输出目标。
这也意味着新的内置功能，如 [`Array` 上的 `at()` 方法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/at)、[`Object.hasOwn`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwn) 或 [`new Error` 上的 `cause` 选项](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error/Error#rethrowing_an_error_with_a_cause)，既可以在使用新的 `--target` 设置时使用，也可以在搭配 `--lib es2022` 时使用。

该功能由 [Kagami Sascha Rosylight (saschanaz)](https://github.com/saschanaz) 通过多个 PR [实现](https://github.com/microsoft/TypeScript/pull/46291)，我们非常感谢这一贡献！

## 移除 `react-jsx` 中不必要的参数

此前，当在 `--jsx react-jsx` 模式下编译如下代码时：

```tsx
export const el = <div>foo</div>
```

TypeScript 会生成如下 JavaScript 代码：

```jsx
import { jsx as _jsx } from 'react/jsx-runtime'
export const el = _jsx('div', { children: 'foo' }, void 0)
```

最后一个 `void 0` 参数在此生成模式下是不必要的，将其移除可以减小打包体积：

```diff
- export const el = _jsx("div", { children: "foo" }, void 0);
+ export const el = _jsx("div", { children: "foo" });
```

感谢 [Alexander Tarasyuk](https://github.com/a-tarasyuk) 提交的 [Pull Request](https://github.com/microsoft/TypeScript/pull/47467)，TypeScript 4.6 现在会省略该 `void 0` 参数。

## JSDoc 参数名建议

在 JSDoc 中，你可以使用 `@param` 标签为参数添加文档：

```js
/**
 * @param x 第一个操作数
 * @param y 第二个操作数
 */
function add(x, y) {
  return x + y
}
```

但如果这些注释过时了会怎样？
如果我们把 `x` 和 `y` 重命名为 `a` 和 `b` 呢？

```js
/**
 * @param x {number} 第一个操作数
 * @param y {number} 第二个操作数
 */
function add(a, b) {
  return a + b
}
```

此前，TypeScript 仅在对 JavaScript 文件执行类型检查时才会提示此信息——即使用 `checkJs` 选项，或在文件顶部添加 `// @ts-check` 注释。

现在，你可以在编辑器中为 TypeScript 文件获得类似的信息提示！
当函数与其 JSDoc 注释之间的参数名称不匹配时，TypeScript 现在会提供相应建议。

![编辑器中显示的建议诊断信息，针对 JSDoc 注释中与实际参数名称不匹配的参数名](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2022/02/jsdoc-comment-suggestions-4-6.png)

[该改动](https://github.com/microsoft/TypeScript/pull/47257) 由 [Alexander Tarasyuk](https://github.com/a-tarasyuk) 友情贡献！

## JavaScript 中更多的语法和绑定错误

TypeScript 扩展了其在 JavaScript 文件中的语法和绑定错误检查范围。
如果你在 Visual Studio 或 Visual Studio Code 等编辑器中打开 JavaScript 文件，或者通过 TypeScript 编译器运行 JavaScript 代码，即使未开启 `checkJs` 或未在文件顶部添加 `// @ts-check` 注释，你也会看到这些新错误。

举个例子，如果在 JavaScript 文件的同一作用域中有两个 `const` 声明，TypeScript 现在会对这些声明报错：

```ts
const foo = 1234
//    ~~~
// 错误：无法重新声明块作用域变量 'foo'。

// ...

const foo = 5678
//    ~~~
// 错误：无法重新声明块作用域变量 'foo'。
```

再比如，TypeScript 会提示修饰符是否被错误使用：

```ts
function container() {
  export function foo() {
    //  ~~~~~~
    // 错误：修饰符不能出现在此处。
  }
}
```

可以通过在文件顶部添加 `// @ts-nocheck` 来禁用这些错误，但我们非常希望获得关于这对你的 JavaScript 工作流影响的早期反馈。
你可以通过安装 [TypeScript and JavaScript Nightly 扩展](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-typescript-next) 轻松在 Visual Studio Code 中进行体验，并在 [第一个](https://github.com/microsoft/TypeScript/pull/47067) 和 [第二个](https://github.com/microsoft/TypeScript/pull/47075) Pull Request 中阅读更多内容。

## TypeScript 追踪分析器（Trace Analyzer）

有时，团队可能会遇到创建和与其他类型进行对比时计算开销极大的类型。
[TypeScript 提供了 `--generateTrace` 标志](https://github.com/microsoft/TypeScript/wiki/Performance#performance-tracing)，以帮助识别其中一些昂贵的类型，或者有时帮助诊断 TypeScript 编译器中的问题。
虽然 `--generateTrace` 生成的信息非常有用（尤其是在 TypeScript 4.6 中新增了一些信息），但在现有的追踪可视化工具中通常很难阅读。

我们最近发布了一个名为 [@typescript/analyze-trace](https://www.npmjs.com/package/@typescript/analyze-trace) 的工具，以更加通俗易懂的方式展示这些信息。
虽然我们不期望每个人都需要 `analyze-trace`，但我们认为它对于任何遇到 [TypeScript 构建性能问题](https://github.com/microsoft/TypeScript/wiki/Performance) 的团队都很有帮助。

更多信息请参阅 [`analyze-trace` 工具的代码仓库](https://github.com/microsoft/typescript-analyze-trace)。

## 破坏性变更

### 泛型对象的剩余属性解构会剔除不可展开的成员

对象剩余（rest）表达式现在会剔除在泛型对象上看似不可展开的成员。
在以下示例中……

```ts
class Thing {
  someProperty = 42

  someMethod() {
    // ...
  }
}

function foo<T extends Thing>(x: T) {
  let { someProperty, ...rest } = x

  // 过去可以运行，现在会报错！
  // 类型 'Omit<T, "someProperty" | "someMethod">' 上不存在属性 'someMethod'。
  rest.someMethod()
}
```

变量 `rest` 之前具有类型 `Omit<T, "someProperty">`，因为 TypeScript 当时会严格分析哪些其他属性被解构了。
这无法准确模拟从非泛型类型进行解构时 `...rest` 的工作方式，因为 `someMethod` 通常也会被丢弃。
在 TypeScript 4.6 中，`rest` 的类型为 `Omit<T, "someProperty" | "someMethod">`。

这在从 `this` 进行解构时也会出现。
当使用 `...rest` 元素解构 `this` 时，不可展开及非公共成员现在会被剔除，这与在其他地方解构类的实例保持一致：

```ts
class Thing {
  someProperty = 42

  someMethod() {
    // ...
  }

  someOtherMethod() {
    let { someProperty, ...rest } = this

    // 过去可以运行，现在会报错！
    // 类型 'Omit<T, "someProperty" | "someMethod">' 上不存在属性 'someMethod'。
    rest.someMethod()
  }
}
```

更多详细信息，请参阅 [此处的对应改动](https://github.com/microsoft/TypeScript/pull/47078)。

### JavaScript 文件始终会收到语法和绑定错误

此前，除了在 JavaScript 文件中误用 TypeScript 语法之外，TypeScript 会忽略 JavaScript 中的大多数语法错误。
TypeScript 现在会在文件中显示 JavaScript 语法和绑定错误，例如使用不正确的修饰符、重复声明等。
这些通常在 Visual Studio Code 或 Visual Studio 中最为明显，但在通过 TypeScript 编译器运行 JavaScript 代码时也会出现。

你可以通过在文件顶部添加 `// @ts-nocheck` 注释来显式关闭这些错误。

更多信息请参阅实现这些特性的 [第一个](https://github.com/microsoft/TypeScript/pull/47067) 和 [第二个](https://github.com/microsoft/TypeScript/pull/47075) Pull Request。
