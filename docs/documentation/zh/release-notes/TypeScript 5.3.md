---
title: TypeScript 5.3
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-5-3.html
oneline: TypeScript 5.3 发布说明
---

## 导入属性（Import Attributes）

TypeScript 5.3 支持了最新的[导入属性（import attributes）](https://github.com/tc39/proposal-import-attributes)提案。

导入属性的一个用例是向运行时提供有关模块预期格式的信息。

```ts
// 我们只希望将其解释为 JSON，
// 而不是带有 `.json` 扩展名的可运行/恶意 JavaScript 文件。
import obj from './something.json' with { type: 'json' }
```

TypeScript 不会检查这些属性的具体内容，因为它们是由宿主环境决定的；TypeScript 只是保留它们，以便浏览器和运行时能够进行处理（并可能报错）。

```ts
// TypeScript 可以接受此写法。
// 但你的浏览器？很可能不行。
import * as foo from './foo.js' with { type: 'fluffy bunny' }
```

动态 `import()` 调用也可以通过第二个参数来使用导入属性。

```ts
const obj = await import('./something.json', {
  with: { type: 'json' },
})
```

该第二个参数的预期类型由名为 `ImportCallOptions` 的类型定义，默认情况下它只期望一个名为 `with` 的属性。

请注意，导入属性是由早期名为[“导入断言（import assertions）”的提案演进而来，后者已在 TypeScript 4.5 中实现](https://devblogs.microsoft.com/typescript/announcing-typescript-4-5/#import-assertions)。
最显著的区别在于使用 `with` 关键字替代了 `assert` 关键字。
但更深层次的区别在于，运行时现在可以自由使用属性来指导导入路径的解析和解释，而导入断言只能在加载模块之后断言某些特征。

随着时间的推移，TypeScript 将弃用旧的导入断言语法，转而采用提议的导入属性语法。
使用 `assert` 的现有代码应当向 `with` 关键字迁移。
需要导入属性的新代码则应一律使用 `with`。

我们要感谢 [Oleksandr Tarasiuk](https://github.com/a-tarasyuk) [实现了该提案](https://github.com/microsoft/TypeScript/pull/54242)！
同时也要感谢 [Wenlu Wang](https://github.com/Kingwl) 此前对[导入断言](https://github.com/microsoft/TypeScript/pull/40698)的实现！

## 在类型导入中稳定支持 `resolution-mode`

在 TypeScript 4.7 中，TypeScript 添加了对 `/// <reference types="..." />` 中 `resolution-mode` 属性的支持，以控制模块说明符应通过 `import` 还是 `require` 语义进行解析。

```ts
/// <reference types="pkg" resolution-mode="require" />

// or

/// <reference types="pkg" resolution-mode="import" />
```

在纯类型导入（type-only imports）的导入断言中也添加了对应的字段；
然而，它之前仅在 TypeScript 的 nightly 版本中受支持。
其原由在于从设计初衷来看，导入*断言*并不是用来指导模块解析的。
因此该特性仅作为实验性功能在 nightly 版本中发布，以收集更多反馈。

但鉴于*[导入属性](#import-attributes)*可以指导解析，而且我们确实看到了合理的用例，TypeScript 5.3 现在正式为 `import type` 支持了 `resolution-mode` 属性。

```ts
// 按照使用 `require()` 导入的方式解析 `pkg`
import type { TypeFromRequire } from 'pkg' with {
  'resolution-mode': 'require',
}

// 按照使用 `import` 导入的方式解析 `pkg`
import type { TypeFromImport } from 'pkg' with {
  'resolution-mode': 'import',
}

export interface MergedType extends TypeFromRequire, TypeFromImport {}
```

这些导入属性也可以在 `import()` 类型中使用。

```ts
export type TypeFromRequire = import('pkg', {
  with: { 'resolution-mode': 'require' },
}).TypeFromRequire

export type TypeFromImport = import('pkg', {
  with: { 'resolution-mode': 'import' },
}).TypeFromImport

export interface MergedType extends TypeFromRequire, TypeFromImport {}
```

欲了解更多信息，请[查看此处的更改](https://github.com/microsoft/TypeScript/pull/55725)。

## 在所有模块模式中支持 `resolution-mode`

此前，仅在 `moduleResolution` 选项为 `node16` 和 `nodenext` 时才允许使用 `resolution-mode`。
为了更方便地专门针对类型目的查找模块，`resolution-mode` 现在可以在所有其他 `moduleResolution` 选项（如 `bundler`、`node10`）中正常工作，并且在 `classic` 下也不会报错。

欲了解更多信息，请[参阅实现该特性的 Pull Request](https://github.com/microsoft/TypeScript/pull/55725)。

## `switch (true)` 中的类型收窄

TypeScript 5.3 现在可以根据 `switch (true)` 中每个 `case` 子句的条件来执行类型收窄。

```ts
function f(x: unknown) {
  switch (true) {
    case typeof x === 'string':
      // 此处 'x' 是 'string'
      console.log(x.toUpperCase())
    // fallthrough 穿透...

    case Array.isArray(x):
      // 此处 'x' 是 'string | any[]'。
      console.log(x.length)
    // fallthrough 穿透...

    default:
    // 此处 'x' 是 'unknown'。
    // ...
  }
}
```

[该特性](https://github.com/microsoft/TypeScript/pull/55991)由 [Mateusz Burzyński](https://github.com/Andarist) 的[先期工作](https://github.com/microsoft/TypeScript/pull/53681)推动实现。
我们对这项贡献表示由衷的感谢！

## 对布尔值的比较收窄（Narrowing On Comparisons to Booleans）

有时你可能会在条件判断中直接与 `true` 或 `false` 进行比较。
通常这些比较是不必要的，但出于代码风格喜好，或是为了避免 JavaScript 中关于真值（truthiness）的某些问题，你可能会倾向于这样写。
无论如何，此前 TypeScript 在执行类型收窄时并不能识别这种形式。

TypeScript 5.3 现已跟进，在收窄变量类型时能够正确理解这些表达式。

```ts
interface A {
  a: string
}

interface B {
  b: string
}

type MyType = A | B

function isA(x: MyType): x is A {
  return 'a' in x
}

function someFn(x: MyType) {
  if (isA(x) === true) {
    console.log(x.a) // 正常！
  }
}
```

感谢 [Mateusz Burzyński](https://github.com/Andarist) 提交了实现该特性的 [Pull Request](https://github.com/microsoft/TypeScript/pull/53681)。

## 通过 `Symbol.hasInstance` 进行 `instanceof` 收窄

JavaScript 中有一个稍显小众的特性：可以重写 `instanceof` 运算符的行为。
为此，`instanceof` 运算符右侧的值需要拥有一个以 `Symbol.hasInstance` 命名的特定方法。

```js
class Weirdo {
  static [Symbol.hasInstance](testedValue) {
    // 等等，什么？
    return testedValue === undefined
  }
}

// false
console.log(new Thing() instanceof Weirdo)

// true
console.log(undefined instanceof Weirdo)
```

为了更好地对 `instanceof` 中的这种行为建模，TypeScript 现在会检查是否存在这样一个 `[Symbol.hasInstance]` 方法，且该方法是否被声明为类型谓词函数。
如果是，`instanceof` 运算符左侧被测试的值将通过该类型谓词进行适当的类型收窄。

```ts
interface PointLike {
  x: number
  y: number
}

class Point implements PointLike {
  x: number
  y: number

  constructor(x: number, y: number) {
    this.x = x
    this.y = y
  }

  distanceFromOrigin() {
    return Math.sqrt(this.x ** 2 + this.y ** 2)
  }

  static [Symbol.hasInstance](val: unknown): val is PointLike {
    return (
      !!val &&
      typeof val === 'object' &&
      'x' in val &&
      'y' in val &&
      typeof val.x === 'number' &&
      typeof val.y === 'number'
    )
  }
}

function f(value: unknown) {
  if (value instanceof Point) {
    // 可以访问这两项 —— 正确！
    value.x
    value.y

    // 无法访问此项 —— 我们拥有的是 'PointLike'，
    // 但我们并*没有*真正的 'Point'。
    value.distanceFromOrigin()
  }
}
```

正如本例所示，`Point` 定义了自己的 `[Symbol.hasInstance]` 方法。
它实际上充当了针对另一个独立类型 `PointLike` 的自定义类型守卫。
在函数 `f` 中，我们能够通过 `instanceof` 将 `value` 收窄为 `PointLike`，但*不是* `Point`。
这意味着我们可以访问属性 `x` 和 `y`，但不能访问方法 `distanceFromOrigin`。

欲了解更多信息，你可以[在此阅读有关此更改的内容](https://github.com/microsoft/TypeScript/pull/55052)。

## 检查在实例字段上对 `super` 属性的访问

在 JavaScript 中，可以通过 `super` 关键字访问基类中的声明。

```js
class Base {
  someMethod() {
    console.log('Base method called!')
  }
}

class Derived extends Base {
  someMethod() {
    console.log('Derived method called!')
    super.someMethod()
  }
}

new Derived().someMethod()
// 输出：
//   Derived method called!
//   Base method called!
```

这与编写类似 `this.someMethod()` 的代码不同，因为后者可能会调用被重写的方法。
这是一个细微的区别；而当某个声明根本没有被重写时，两者通常可以互换使用，这使得该区别更加微妙。

```js
class Base {
  someMethod() {
    console.log('someMethod called!')
  }
}

class Derived extends Base {
  someOtherMethod() {
    // 这两者的行为完全相同。
    this.someMethod()
    super.someMethod()
  }
}

new Derived().someOtherMethod()
// 输出：
//   someMethod called!
//   someMethod called!
```

将它们混用的问题在于：`super` 仅适用于原型上声明的成员&mdash;&mdash;而*不适用于*实例属性。
这意味着如果你编写了 `super.someMethod()`，但 `someMethod` 是定义为一个字段，你就会遇到运行时错误！

```ts
class Base {
  someMethod = () => {
    console.log('someMethod called!')
  }
}

class Derived extends Base {
  someOtherMethod() {
    super.someMethod()
  }
}

new Derived().someOtherMethod()
// 💥
// 无法运行，因为 'super.someMethod' 是 'undefined'。
```

TypeScript 5.3 现在会更严格地检查 `super` 属性访问/方法调用，以确认它们是否对应于类字段。
如果是，现在将产生类型检查错误。

感谢 [Jack Works](https://github.com/Jack-Works) 贡献了[此项检查](https://github.com/microsoft/TypeScript/pull/54056)！

## 针对类型的交互式内联提示（Interactive Inlay Hints）

TypeScript 的内联提示（inlay hints）现已支持跳转到类型定义！
这使得在代码中浏览跳转变得更加轻松便捷。

![Ctrl-clicking an inlay hint to jump to the definition of a parameter type.](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2023/10/clickable-inlay-hints-for-types-5-3-beta.gif)

欲了解更多信息，请参阅[此处的具体实现](https://github.com/microsoft/TypeScript/pull/55141)。

## 优先使用 `type` 自动导入的设置项

此前，当 TypeScript 为类型位置的内容生成自动导入时，它会根据你的设置添加 `type` 修饰符。
例如，当在以下代码中对 `Person` 执行自动导入时：

```ts
export let p: Person
```

TypeScript 的编辑器体验通常会将 `Person` 的导入添加为：

```ts
import { Person } from './types'

export let p: Person
```

而在某些特定设置（如 `verbatimModuleSyntax`）下，它会添加 `type` 修饰符：

```ts
import { type Person } from './types'

export let p: Person
```

然而，你的代码库可能无法使用这些配置选项；或者你只是个人偏好在可能的情况下显式使用 `type` 导入。

[通过最近的一项改动](https://github.com/microsoft/TypeScript/pull/56090)，TypeScript 现在允许将其配置为特定于编辑器的选项。
在 Visual Studio Code 中，你可以在界面设置中启用 “TypeScript › Preferences: Prefer Type Only Auto Imports”，或在 JSON 配置中设置 `typescript.preferences.preferTypeOnlyAutoImports`。

<!--
## Triggerable Refactor to Convert to Template String

https://github.com/microsoft/TypeScript/pull/54647

-->

## 通过跳过 JSDoc 解析实现优化

当通过 `tsc` 运行 TypeScript 时，编译器现在会避免解析 JSDoc。
这不仅降低了解析时间本身，还减少了存储注释的内存占用以及垃圾回收所花费的时间。
总的来说，你应该会体验到稍快的编译速度以及在 `--watch` 模式下更迅速的反馈。

[具体的改动可以在此处查看](https://github.com/microsoft/TypeScript/pull/52921)。

由于并非所有使用 TypeScript 的工具都需要存储 JSDoc（例如 typescript-eslint 和 Prettier），这种解析策略已作为 API 的一部分对外暴露。
这使得这些工具能够获得我们为 TypeScript 编译器带来的同等内存与速度提升。
注释解析策略的新选项在 `JSDocParsingMode` 中进行了说明。
更多信息可以在[该 Pull Request 中查看](https://github.com/microsoft/TypeScript/pull/55739)。

## 通过比较非规范化交叉类型实现优化

在 TypeScript 中，联合类型和交叉类型总是遵循一种特定的形式，其中交叉类型不能包含联合类型。
这意味着当我们在联合类型上创建交叉类型（如 `A & (B | C)`）时，该交叉类型会被规范化为 `(A & B) | (A & C)`。
尽管如此，在某些情况下类型系统仍会保留原始形式以供展示使用。

事实证明，原始形式可用于在类型之间进行某些巧妙的快速路径（fast-path）比较。

例如，假设我们有 `SomeType & (Type1 | Type2 | ... | Type99999NINE)`，并且想要检查它是否可以赋值给 `SomeType`。
回顾一下，我们的源类型实际上并不是一个交叉类型&mdash;&mdash;而是一个形如 `(SomeType & Type1) | (SomeType & Type2) | ... |(SomeType & Type99999NINE)` 的联合类型。
在检查联合类型是否可赋值给某个目标类型时，我们必须检查联合类型的*每个*成员是否都可以赋值给目标类型，而这可能会非常慢。

在 TypeScript 5.3 中，我们会审视之前得以保留的原始交叉类型形式。
当我们比较类型时，会进行一次快速检查，以查看目标类型是否存在于源交叉类型的任何组成部分中。

欲了解更多信息，请[参阅此 Pull Request](https://github.com/microsoft/TypeScript/pull/55851)。

## 合并 `tsserverlibrary.js` 与 `typescript.js`

TypeScript 本身随附了两个库文件：`tsserverlibrary.js` 和 `typescript.js`。
某些 API 仅在 `tsserverlibrary.js` 中提供（例如 `ProjectService` API），这可能对某些导入者很有用。
但两者是具有大量重叠的独立打包文件，导致包中存在重复代码。
此外，由于自动导入或肌肉记忆，要始终如一地使用其中某一个可能会很困难。
一不小心就极易同时加载这两个模块，而且代码在不同的 API 实例上可能无法正常工作。
即使能正常工作，加载第二个打包文件也会增加资源消耗。

鉴于此，我们决定将两者进行合并。
`typescript.js` 现在包含了 `tsserverlibrary.js` 原先包含的内容，而 `tsserverlibrary.js` 现在仅直接重新导出 `typescript.js`。
对比合并前后的情况，我们观察到了如下包体积缩减：

|          | 变更前    | 变更后    | 差值      | 变化幅度 |
| -------- | --------- | --------- | --------- | -------- |
| 打包体积 | 6.90 MiB  | 5.48 MiB  | -1.42 MiB | -20.61%  |
| 解包体积 | 38.74 MiB | 30.41 MiB | -8.33 MiB | -21.50%  |

|                            | 变更前     | 变更后     | 差值        | 变化幅度 |
| -------------------------- | ---------- | ---------- | ----------- | -------- |
| `lib/tsserverlibrary.d.ts` | 570.95 KiB | 865.00 B   | -570.10 KiB | -99.85%  |
| `lib/tsserverlibrary.js`   | 8.57 MiB   | 1012.00 B  | -8.57 MiB   | -99.99%  |
| `lib/typescript.d.ts`      | 396.27 KiB | 570.95 KiB | +174.68 KiB | +44.08%  |
| `lib/typescript.js`        | 7.95 MiB   | 8.57 MiB   | +637.53 KiB | +7.84%   |

换言之，包体积缩减了超过 20.5%。

欲了解更多信息，你可以[在此查看相关工作](https://github.com/microsoft/TypeScript/pull/55273)。

## 破坏性变更与正确性改进

### `lib.d.ts` 变更

为 DOM 生成的类型可能会对你的代码库产生影响。
欲了解更多信息，请[参阅 TypeScript 5.3 的 DOM 更新](https://github.com/microsoft/TypeScript/pull/55798)。

### 检查在实例属性上对 `super` 的访问

TypeScript 5.3 现在会检测通过 `super.` 属性访问所引用的声明是否为类字段，并在是类字段时报错。
这可以防止在运行时可能出现的错误。

[在此查看关于此更改的更多信息](https://github.com/microsoft/TypeScript/pull/54056)。
