---
title: 深入探讨
layout: docs
permalink: /zh/docs/handbook/declaration-files/deep-dive.html
oneline: '深入探讨 d.ts 声明文件的工作原理'
---

## 声明文件原理解析：深入探讨

组织模块结构以精确呈现你所需的 API 形态可能会有些棘手。
例如，我们可能希望一个模块既可以直接调用，也可以通过 `new` 调用以产生不同的类型；
或者在一个层级结构中暴露多种命名类型；
同时还在模块对象上挂载一些属性。

通过阅读本指南，你将掌握编写复杂声明文件并暴露友好 API 界面的技巧。
本指南重点介绍模块（或 UMD）库，因为这类库的场景更加丰富多样。

## 核心概念

只要理解 TypeScript 工作方式的若干核心概念，
你就能够完全掌握如何构建任意形态的声明。

### 类型

如果你正在阅读本指南，很可能已经大致了解了 TypeScript 中的类型是什么。
不过更明确地说，_类型_ 是通过以下声明引入的：

- 类型别名声明（`type sn = number | string;`）
- 接口声明（`interface I { x: number[]; }`）
- 类声明（`class C { }`）
- 枚举声明（`enum E { A, B, C }`）
- 指向某个类型的 `import` 声明

上述每种声明形式都会创建一个新的类型名称。

### 值

与类型一样，你可能也已经了解什么是值。
值是在运行时可供表达式引用的名称。
例如 `let x = 5;` 创建了一个名为 `x` 的值。

同样明确地说，以下内容会创建值：

- `let`、`const` 和 `var` 声明
- 包含值的 `namespace` 或 `module` 声明
- `enum` 声明
- `class` 声明
- 指向某个值的 `import` 声明
- `function` 声明

### 命名空间

类型可以存在于*命名空间*中。
例如，如果我们有声明 `let x: A.B.C`，
我们便说类型 `C` 来自 `A.B` 命名空间。

这种区分微妙且重要——这里的 `A.B` 不一定是类型，也不一定是值。

## 简单组合：同名多义

对于给定的名称 `A`，我们最多能为 `A` 找到三种不同的含义：类型、值或命名空间。
该名称如何被解析取决于其使用的上下文。
例如，在声明 `let m: A.A = A;` 中，
`A` 首先被用作命名空间，接着用作类型名称，最后用作值。
这些含义最终可能指向完全不同的声明！

这看起来可能令人困惑，但只要我们不过度重载，实际上会非常方便。
让我们来看看这种组合行为的一些实用特性。

### 内置组合

细心的读者会发现，例如 `class` 同时出现在了*类型*和*值*列表中。
声明 `class C { }` 实际上创建了两样东西：
一个代表类实例结构的*类型* `C`，
以及一个代表类构造函数的*值* `C`。
枚举声明的行为也与之类似。

### 用户自定义组合

假设我们编写了一个模块文件 `foo.d.ts`：

```ts
export var SomeVar: { a: SomeType }
export interface SomeType {
  count: number
}
```

然后使用它：

```ts
import * as foo from './foo'
let x: foo.SomeType = foo.SomeVar.a
console.log(x.count)
```

这可以正常工作，但我们可能会设想 `SomeType` 与 `SomeVar` 关系非常紧密，
以至于你希望它们拥有相同的名称。
我们可以使用组合，将这两个不同的实体（值和类型）以相同的名称 `Bar` 暴露出来：

```ts
export var Bar: { a: Bar }
export interface Bar {
  count: number
}
```

这为在消费端代码中进行解构提供了绝佳的机会：

```ts
import { Bar } from './foo'
let x: Bar = Bar.a
console.log(x.count)
```

同样地，在这里我们将 `Bar` 既作为类型使用，又作为值使用。
请注意，我们无需声明 `Bar` 值的类型为 `Bar`——它们是相互独立的。

## 高级组合

某些类型的声明可以跨多个声明进行组合。
例如，`class C { }` 与 `interface C { }` 可以共存，并共同向 `C` 类型贡献属性。

只要不产生冲突，这都是合法的。
一条通用的经验法则是：同名的值之间总是会发生冲突，除非它们被声明为 `namespace`；
如果类型是通过类型别名声明（`type s = string`）定义的，则它们会发生冲突；
而命名空间之间则永远不会冲突。

让我们来看看如何运用这一特性。

### 使用 `interface` 进行扩展

我们可以通过另一个 `interface` 声明向已有 `interface` 添加额外成员：

```ts
interface Foo {
  x: number;
}
// ... elsewhere ...
interface Foo {
  y: number;
}
let a: Foo = ...;
console.log(a.x + a.y); // OK
```

这也同样适用于类：

```ts
class Foo {
  x: number;
}
// ... elsewhere ...
interface Foo {
  y: number;
}
let a: Foo = ...;
console.log(a.x + a.y); // OK
```

请注意，我们无法使用接口向类型别名（`type s = string;`）添加成员。

### 使用 `namespace` 进行扩展

`namespace` 声明可用于以任何不引发冲突的方式添加新的类型、值和命名空间。

例如，我们可以向类添加静态成员：

```ts
class C {}
// ... elsewhere ...
namespace C {
  export let x: number
}
let y = C.x // OK
```

注意，在这个例子中，我们向 `C` 的*静态*部分（即它的构造函数）添加了一个值。
这是因为我们添加的是一个*值*，而所有值的容器是另一个值
（类型由命名空间包含，而命名空间由其他命名空间包含）。

我们还可以向类中添加带有命名空间的类型：

```ts
class C {}
// ... elsewhere ...
namespace C {
  export interface D {}
}
let y: C.D // OK
```

在这个例子中，在我们为它编写 `namespace` 声明之前，并不存在命名空间 `C`。
`C` 作为命名空间的含义，与由类创建的 `C` 作为值或类型的含义并不冲突。

最后，我们可以使用 `namespace` 声明执行多种不同的合并。
这并非一个特别贴合实际的例子，但展示了各种有趣的特性与行为：

```ts
namespace X {
  export interface Y {}
  export class Z {}
}

// ... elsewhere ...
namespace X {
  export var Y: number
  export namespace Z {
    export class C {}
  }
}
type X = string
```

在这个例子中，第一个代码块创建了以下名称含义：

- 值 `X`（因为 `namespace` 声明包含了一个值 `Z`）
- 命名空间 `X`（因为 `namespace` 声明包含了一个类型 `Y`）
- `X` 命名空间中的类型 `Y`
- `X` 命名空间中的类型 `Z`（类的实例结构）
- 作为值 `X` 之属性的值 `Z`（类的构造函数）

第二个代码块创建了以下名称含义：

- 作为值 `X` 之属性的值 `Y`（类型为 `number`）
- 命名空间 `Z`
- 作为值 `X` 之属性的值 `Z`
- `X.Z` 命名空间中的类型 `C`
- 作为值 `X.Z` 之属性的值 `C`
- 类型 `X`

<!-- TODO: Write more on that. -->
