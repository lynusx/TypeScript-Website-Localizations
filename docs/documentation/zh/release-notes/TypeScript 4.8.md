---
title: TypeScript 4.8
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-4-8.html
oneline: TypeScript 4.8 发布说明
---

## 交叉类型折叠、联合类型兼容性与类型收窄的改进

TypeScript 4.8 在 `--strictNullChecks` 下带来了一系列正确性和一致性方面的改进。这些改动影响了交叉类型和联合类型的工作机制，并在 TypeScript 的类型收窄逻辑中发挥了重要作用。

例如，`unknown` 在本质上与联合类型 `{} | null | undefined` 非常接近，因为它能接受 `null`、`undefined` 以及其他任何类型。TypeScript 现在能够识别出这一点，并允许将 `unknown` 赋值给 `{} | null | undefined`。

```ts
function f(x: unknown, y: {} | null | undefined) {
  x = y // always worked
  y = x // used to error, now works
}
```

另一项改变是，`{}` 与任何其他对象类型的交叉类型会直接简化为该对象类型本身。这意味着我们能够将 `NonNullable` 重写为仅使用与 `{}` 的交叉类型，因为 `{} & null` 和 `{} & undefined` 会被直接剔除。

```diff
- type NonNullable<T> = T extends null | undefined ? never : T;
+ type NonNullable<T> = T & {};
```

这是一项改进，因为像这样的交叉类型可以被折叠（reduce）和赋值，而条件类型目前无法做到这一点。因此，`NonNullable<NonNullable<T>>` 现在至少可以简化为 `NonNullable<T>`，而此前则无法做到。

```ts
function foo<T>(x: NonNullable<T>, y: NonNullable<NonNullable<T>>) {
  x = y // always worked
  y = x // used to error, now works
}
```

这些改动还让我们得以在控制流分析和类型收窄方面引入合理的改进。例如，在真值（truthy）分支中，`unknown` 现在能像 `{} | null | undefined` 一样被收窄。

```ts
function narrowUnknownishUnion(x: {} | null | undefined) {
  if (x) {
    x // {}
  } else {
    x // {} | null | undefined
  }
}

function narrowUnknown(x: unknown) {
  if (x) {
    x // used to be 'unknown', now '{}'
  } else {
    x // unknown
  }
}
```

泛型值也以类似的方式被收窄。在检查某个值不是 `null` 或 `undefined` 时，TypeScript 现在直接将其与 `{}` 进行交叉——这同样等价于声明它是 `NonNullable`。综合上述诸多改进，我们现在可以在无需任何类型断言的情况下定义如下函数：

```ts
function throwIfNullable<T>(value: T): NonNullable<T> {
  if (value === undefined || value === null) {
    throw Error('Nullable value!')
  }

  // Used to fail because 'T' was not assignable to 'NonNullable<T>'.
  // Now narrows to 'T & {}' and succeeds because that's just 'NonNullable<T>'.
  return value
}
```

`value` 现在被收窄为 `T & {}`，这与 `NonNullable<T>` 是等价的——因此该函数体无需任何 TypeScript 特有语法即可直接通过检查。

单看这些改动可能显得很小，但它们修复了多年来用户报告的大量细微体验痛点（paper cuts）。

欲了解这些改进的更多细节，可以[在此阅读更多内容](https://github.com/microsoft/TypeScript/pull/49119)。

## 模板字符串类型中 `infer` 类型推断的改进

TypeScript 最近引入了一种在条件类型中为 `infer` 类型变量添加 `extends` 约束的方法：

```ts
// Grabs the first element of a tuple if it's assignable to 'number',
// and returns 'never' if it can't find one.
type TryGetNumberIfFirst<T> = T extends [infer U extends number, ...unknown[]]
  ? U
  : never
```

如果这些 `infer` 类型出现在模板字符串类型中，并且被约束为原始类型，TypeScript 现在会尝试解析出字面量类型：

```ts
// SomeNum used to be 'number'; now it's '100'.
type SomeNum = '100' extends `${infer U extends number}` ? U : never

// SomeBigInt used to be 'bigint'; now it's '100n'.
type SomeBigInt = '100' extends `${infer U extends bigint}` ? U : never

// SomeBool used to be 'boolean'; now it's 'true'.
type SomeBool = 'true' extends `${infer U extends boolean}` ? U : never
```

这现在能够更好地反映库在运行时的行为，并提供更精确的类型。

关于这一点需要注意：当 TypeScript 解析出这些字面量类型时，它会贪婪地尝试解析出尽可能多看起来符合相应原始类型的内容；然后，它会检查该原始类型转换回字符串后是否与原字符串内容相匹配。换句话说，TypeScript 会检查从字符串到原始类型、再转换回来的往返过程（round-trip）是否一致。如果无法完全匹配，它将回退到基础原始类型。

```ts
// JustNumber is `number` here because TypeScript parses out `"1.0"`, but `String(Number("1.0"))` is `"1"` and doesn't match.
type JustNumber = '1.0' extends `${infer T extends number}` ? T : never
```

你可以[在此查看有关此特性的更多信息](https://github.com/microsoft/TypeScript/pull/48094)。

## `--build`、`--watch` 与 `--incremental` 性能改进

TypeScript 4.8 引入了若干优化，旨在加速涉及 `--watch` 和 `--incremental` 的场景，以及使用 `--build` 构建项目引用的场景。例如，TypeScript 现在能够在 `--watch` 模式下发生无操作更改（no-op changes）时避免耗费时间更新时间戳，这加快了重新构建的速度，并避免了干扰其他可能正在监视 TypeScript 输出的构建工具。我们还引入了许多其他优化，能够在 `--build`、`--watch` 和 `--incremental` 之间复用信息。

这些改进有多显著？在某个相当大的内部代码库中，我们在许多简单的常见操作上观察到了 10% 到 25% 的时间缩减，在无变更场景下缩减了约 40%。我们在 TypeScript 自身的代码库上也看到了类似的结果。

你可以在 GitHub 上查看[这些更改以及具体的性能测试结果](https://github.com/microsoft/TypeScript/pull/48784)。

## 比较对象与数组字面量时的错误提示

在许多语言中，诸如 `==` 之类的运算符会对对象执行所谓的“值”相等性检查。例如在 Python 中，通过 `==` 检查某个值是否等于空列表来判断列表是否为空是完全合法的：

```py
if people_at_home == []:
    print("here's where I lie, broken inside. </3")
    adopt_animals()
```

但这在 JavaScript 中并非如此，在 JavaScript 中对象（以及数组）之间的 `==` 和 `===` 检查的是两个引用是否指向同一个值。我们认为，JavaScript 中的类似代码充其量是给开发者设下的早期隐患，最坏的情况则是生产代码中的 bug。因此，TypeScript 现在禁止了如下代码：

```ts
if (peopleAtHome === []) {
  //  ~~~~~~~~~~~~~~~~~~~
  // This condition will always return 'false' since JavaScript compares objects by reference, not value.
  console.log("here's where I lie, broken inside. </3")
  adoptAnimals()
}
```

我们要向贡献了此项检查的 [Jack Works](https://github.com/Jack-Works) 表示感谢。你可以[在此查看相关改动](https://github.com/microsoft/TypeScript/pull/45978)。

## 改进来自解构绑定模式的推断

在某些情况下，TypeScript 会从解构绑定模式中提取类型以做出更好的推断。

```ts
declare function chooseRandomly<T>(x: T, y: T): T

let [a, b, c] = chooseRandomly([42, true, 'hi!'], [0, false, 'bye!'])
//   ^  ^  ^
//   |  |  |
//   |  |  string
//   |  |
//   |  boolean
//   |
//   number
```

当 `chooseRandomly` 需要确定 `T` 的类型时，它主要会查看 `[42, true, "hi!"]` 和 `[0, false, "bye!"]`；但 TypeScript 需要判断这两个类型应当是 `Array<number | boolean | string>` 还是元组类型 `[number, boolean, string]`。为此，它会查找现有的候选类型作为提示，以确定是否存在元组类型。当 TypeScript 看到绑定模式 `[a, b, c]` 时，它会创建类型 `[any, any, any]`，该类型被作为 `T` 的低优先级候选类型，同时也用作推断 `[42, true, "hi!"]` 和 `[0, false, "bye!"]` 类型的提示。

你可以看出这在 `chooseRandomly` 中表现良好，但在其他情况下却差强人意。例如以下代码：

```ts
declare function f<T>(x?: T): T

let [x, y, z] = f()
```

绑定模式 `[x, y, z]` 提示 `f` 应当产生一个 `[any, any, any]` 元组；但 `f` 实际上不应该根据绑定模式改变其类型参数。它无法根据赋值的目标凭空捏造出一个新的类数组值，因此绑定模式类型对生成的类型产生了过大的影响。最重要的是，由于绑定模式类型充满了 `any`，导致 `x`、`y` 和 `z` 的类型最终都变成了 `any`。

在 TypeScript 4.8 中，这些绑定模式绝不会再被用作类型参数的候选。相反，只有在参数需要更具体的类型时（如上述 `chooseRandomly` 示例中），才会参考它们。如果你需要恢复旧行为，随时可以提供显式的类型参数。

如果你想了解更多，可以[在 GitHub 上查看此项变更](https://github.com/microsoft/TypeScript/pull/49086)。

## 文件监视修复（特别是在跨 `git checkout` 时）

我们长期以来一直存在一个 bug：TypeScript 在 `--watch` 模式和编辑器场景下处理某些文件变更时非常吃力。有时其症状表现为出现陈旧或不准确的错误提示，需要重启 `tsc` 或 VS Code 才能解决。这些问题频繁发生在 Unix 系统上，你可能在用 vim 保存文件或在 git 中切换分支后遇到过。

这是由于对 Node.js 如何在不同文件系统间处理重命名事件所做的假设导致的。Linux 和 macOS 使用的文件系统采用 [inode](https://en.wikipedia.org/wiki/Inode)，而 [Node.js 会将文件监视器附加到 inode 而非文件路径](https://nodejs.org/api/fs.html#inodes)。因此，当 Node.js 返回[监视器对象（watcher object）](https://nodejs.org/api/fs.html#class-fsfswatcher)时，根据平台和文件系统的不同，它监视的可能是路径或 inode。

为了提高效率，TypeScript 如果检测到磁盘上仍然存在该路径，就会尝试复用相同的监视器对象。问题就出在这里：即使该路径上仍然存在文件，但可能已经创建了一个独立的新文件，而该文件具有不同的 inode。结果是 TypeScript 复用了旧的监视器对象，而不是在原位置安装新的监视器，导致它监视的可能是一个完全无关的文件的变更。因此，TypeScript 4.8 现在在 inode 系统上妥善处理了这些情况，正确安装了新的监视器并修复了该问题。

我们要衷心感谢 Airtable 的 [Marc Celani](https://github.com/MarcCelani-at) 及其团队，他们投入了大量时间调查所遇到的问题并指出了根本原因。你可以[在此查看有关文件监视的具体修复](https://github.com/microsoft/TypeScript/pull/48997)。

## “查找所有引用”性能改进

在编辑器中运行“查找所有引用（Find-All-References）”时，TypeScript 现在在聚合引用时能够表现得更为智能。这使得 TypeScript 在自身代码库中搜索广泛使用的标识符所需的时间缩减了约 20%。

[你可以在此阅读有关此项改进的更多信息](https://github.com/microsoft/TypeScript/pull/49581)。

## 从自动导入中排除特定文件

TypeScript 4.8 引入了一项编辑器首选项，用于从自动导入中排除特定文件。在 Visual Studio Code 中，可以在“设置”界面的“Auto Import File Exclude Patterns”下，或在 `.vscode/settings.json` 文件中添加文件名或 glob 模式：

```jsonc
{
  // Note that `javascript.preferences.autoImportFileExcludePatterns` can be specified for JavaScript too.
  "typescript.preferences.autoImportFileExcludePatterns": [
    "**/node_modules/@types/node",
  ],
}
```

当你无法避免在编译中包含某些模块或库，但极少希望从它们导入内容时，这会非常有用。这些模块可能包含大量导出项，从而污染自动导入列表并导致难以导航，而该选项可以在这些情况下提供帮助。

你可以[在此查看有关该实现的更多细节](https://github.com/microsoft/TypeScript/pull/49578)。

## 正确性修复与破坏性变更

由于类型系统变更的本质，几乎很少有变更不会对*某些*代码产生影响；不过，有几项变更更有可能需要对现有代码进行适配。

### `lib.d.ts` 更新

虽然 TypeScript 竭力避免重大破坏性变更，但内置库中哪怕很小的改动也可能引发问题。我们预计 DOM 和 `lib.d.ts` 的更新不会带来重大破坏，但一个显著的变化是 `Error` 上的 `cause` 属性现在的类型是 `unknown`，而不是 `Error`。

### 无约束泛型不再可赋值给 `{}`

在 TypeScript 4.8 中，对于启用了 `strictNullChecks` 的项目，当无约束的类型参数被用于不允许 `null` 或 `undefined` 作为有效值的位置时，TypeScript 现在会正确报错。这将包括期望 `{}`、`object` 或所有属性均为可选的对象类型的任何位置。

一个简单的示例如下：

```ts
// Accepts any non-null non-undefined value
function bar(value: {}) {
  Object.keys(value) // This call throws on null/undefined at runtime.
}

// Unconstrained type parameter T...
function foo<T>(x: T) {
  bar(x) // Used to be allowed, now is an error in 4.8.
  //  ~
  // error: Argument of type 'T' is not assignable to parameter of type '{}'.
}

foo(undefined)
```

如上所示，此类代码存在潜在的 bug——值 `null` 和 `undefined` 可以通过这些无约束的类型参数间接传递给不应该接收这些值的代码。

此行为在类型位置中也会体现。例如：

```ts
interface Foo<T> {
  x: Bar<T>
}

interface Bar<T extends {}> {}
```

不想处理 `null` 和 `undefined` 的现有代码可以通过传递适当的约束来修复：

```diff
- function foo<T>(x: T) {
+ function foo<T extends {}>(x: T) {
```

另一种变通方案是在运行时检查 `null` 和 `undefined`：

```diff
  function foo<T>(x: T) {
+     if (x !== null && x !== undefined) {
          bar(x);
+     }
  }
```

如果你确信出于某种原因你的泛型值不可能是 `null` 或 `undefined`，也可以直接使用非空断言：

```diff
  function foo<T>(x: T) {
-     bar(x);
+     bar(x!);
  }
```

在类型层面，通常要么需要传递约束，要么需要将类型与 `{}` 进行交叉。

欲了解更多信息，可以查看[引入该变更的 PR](https://github.com/microsoft/TypeScript/pull/49119) 以及[关于无约束泛型如何运作的具体讨论 issue](https://github.com/microsoft/TypeScript/issues/49489)。

### 装饰器在 TypeScript 语法树中放置于 `modifiers` 上

TC39 中装饰器的当前发展方向意味着 TypeScript 必须处理装饰器位置上的破坏性变更。此前，TypeScript 假定装饰器始终放置在所有关键字/修饰符之前。例如：

```ts
@decorator
export class Foo {
  // ...
}
```

按照目前的提案，装饰器不支持这种语法。相反，`export` 关键字必须位于装饰器之前：

```ts
export
@decorator
class Foo {
  // ...
}
```

遗憾的是，TypeScript 的语法树是*具体的（concrete）*而非*抽象的（abstract）*，并且我们的架构期望语法树节点字段在顺序上彼此完全明确。为了同时支持旧版装饰器和提案中的装饰器，TypeScript 必须能够顺畅地解析并穿插修饰符和装饰器。

为此，它公开了一个名为 `ModifierLike` 的新类型别名，表示 `Modifier` 或 `Decorator`：

```ts
export type ModifierLike = Modifier | Decorator
```

装饰器现在与 `modifiers` 放置在同一个字段中，设置时该字段类型为 `NodeArray<ModifierLike>`，并且整个字段已被标记为废弃。

```diff
- readonly modifiers?: NodeArray<Modifier> | undefined;
+ /**
+  * @deprecated ...
+  * Use `ts.canHaveModifiers()` to test whether a `Node` can have modifiers.
+  * Use `ts.getModifiers()` to get the modifiers of a `Node`.
+  * ...
+  */
+ readonly modifiers?: NodeArray<ModifierLike> | undefined;
```

所有现有的 `decorators` 属性均已被标记为废弃，并且读取时将始终返回 `undefined`。其类型也已更改为 `undefined`，以便现有工具能够正确处理它们。

```diff
- readonly decorators?: NodeArray<Decorator> | undefined;
+ /**
+  * @deprecated ...
+  * Use `ts.canHaveDecorators()` to test whether a `Node` can have decorators.
+  * Use `ts.getDecorators()` to get the decorators of a `Node`.
+  * ...
+  */
+ readonly decorators?: undefined;
```

为了避免新的废弃警告和其他问题，TypeScript 现在公开了四个新函数来代替 `decorators` 和 `modifiers` 属性。它们分别提供了用于测试节点是否支持修饰符和装饰器的断言函数，以及用于提取它们的相应访问器函数：

```ts
function canHaveModifiers(node: Node): node is HasModifiers
function getModifiers(node: HasModifiers): readonly Modifier[] | undefined

function canHaveDecorators(node: Node): node is HasDecorators
function getDecorators(node: HasDecorators): readonly Decorator[] | undefined
```

作为从节点访问修饰符的示例，你可以编写：

```ts
const modifiers = canHaveModifiers(myNode) ? getModifiers(myNode) : undefined
```

请注意，每次调用 `getModifiers` 和 `getDecorators` 都可能会分配一个新数组。

欲了解更多信息，请参阅以下相关改动：

- [语法树节点的重构](https://github.com/microsoft/TypeScript/pull/49089)
- [废弃处理](https://github.com/microsoft/TypeScript/pull/50343)
- [公开断言函数](https://github.com/microsoft/TypeScript/pull/50399)

### JavaScript 文件中无法导入/导出类型

此前，TypeScript 允许 JavaScript 文件在 `import` 和 `export` 语句中导入和导出以类型声明但没有值的实体。这种行为是不正确的，因为在 ECMAScript 模块下，对不存在的值进行具名导入和导出将导致运行时错误。当 JavaScript 文件在 `--checkJs` 下或通过 `// @ts-check` 注释进行类型检查时，TypeScript 现在会报错。

```ts
// @ts-check

// Will fail at runtime because 'SomeType' is not a value.
import { someValue, SomeType } from 'some-module'

/**
 * @type {SomeType}
 */
export const myValue = someValue

/**
 * @typedef {string | number} MyType
 */

// Will fail at runtime because 'MyType' is not a value.
export { MyType as MyExportedType }
```

要引用来自另一个模块的类型，可以直接限定导入路径（qualified import）：

```diff
- import { someValue, SomeType } from "some-module";
+ import { someValue } from "some-module";

  /**
-  * @type {SomeType}
+  * @type {import("some-module").SomeType}
   */
  export const myValue = someValue;
```

要导出类型，只需在 JSDoc 中使用 `/** @typedef */` 注释即可。`@typedef` 注释已经会自动从其所在模块中导出类型：

```diff
  /**
   * @typedef {string | number} MyType
   */

+ /**
+  * @typedef {MyType} MyExportedType
+  */
- export { MyType as MyExportedType };
```

你可以[在此处阅读关于此项变更的更多信息](https://github.com/microsoft/TypeScript/pull/49580)。

### 解构绑定模式不再直接作为推断候选

如上所述，绑定模式在函数调用中不再改变推断结果的类型。你可以[在此处阅读关于原始变更的更多信息](https://github.com/microsoft/TypeScript/pull/49086)。

### 类型签名中解构绑定模式未使用的重命名现在会报错

TypeScript 的类型标注语法在解构值时往往容易被误用。例如以下函数：

```ts
declare function makePerson({ name: string, age: number }): Person
```

你可能会读到这个签名并认为 `makePerson` 显然接受一个带有 `string` 类型的 `name` 属性和 `number` 类型的 `age` 属性的对象；然而，JavaScript 的解构语法实际上在这里占了上风。`makePerson` 确实说明它将接受一个具有 `name` 和 `age` 属性的对象，但它并没有为它们指定类型，而只是说明将 `name` 和 `age` 分别重命名为 `string` 和 `number`。

在纯类型结构中，编写此类代码是无用的，而且通常是一个错误，因为开发者通常以为自己在编写类型标注。

TypeScript 4.8 将这些情况标记为错误，除非它们在签名中稍后被引用。编写上述签名的正确方式如下：

```ts
declare function makePerson(options: { name: string; age: number }): Person

// or

declare function makePerson({
  name,
  age,
}: {
  name: string
  age: number
}): Person
```

此更改可以捕获声明中的 bug，并且有助于改进现有代码。我们要向 [GitHub 用户 uhyo](https://github.com/uhyo) 致谢，是他提供了这项检查。[你可以在此阅读关于此项变更的内容](https://github.com/microsoft/TypeScript/pull/41044)。
