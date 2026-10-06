---
title: TypeScript 4.3
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-4-3.html
oneline: TypeScript 4.3 发布说明
---

## 属性上独立的读写类型

在 JavaScript 中，API 在存储传入的值之前对其进行转换是非常普遍的做法。
这也经常发生在 getter 和 setter 中。
例如，设想我们有一个类，它的 setter 总是会在将值保存到私有字段之前先将其转换为 `number`。

```js twoslash
class Thing {
  #size = 0

  get size() {
    return this.#size
  }
  set size(value) {
    let num = Number(value)

    // Don't allow NaN and stuff.
    if (!Number.isFinite(num)) {
      this.#size = 0
      return
    }

    this.#size = num
  }
}
```

我们该如何在 TypeScript 中为这段 JavaScript 代码定义类型？
从技术上讲，我们在这里无需做任何特殊处理——TypeScript 即使在没有显式类型的情况下也能推断出 `size` 是一个数字。

问题在于，`size` 允许被赋予除 `number` 之外的其他类型。
我们可以通过将 `size` 声明为 `unknown` 或 `any` 来变通处理，如下所示：

```ts
class Thing {
  // ...
  get size(): unknown {
    return this.#size
  }
}
```

但这并不理想——`unknown` 会迫使读取 `size` 的人进行类型断言，而 `any` 则无法捕获任何错误。
如果我们确实想为此类转换值的 API 建模，在先前版本的 TypeScript 中，我们不得不在“严格精确”（读取更方便，写入更困难）和“宽容宽泛”（写入更方便，读取更困难）之间做出二选一的妥协。

为此，TypeScript 4.3 允许为属性的读取和写入分别指定类型。

```ts twoslash
class Thing {
  #size = 0

  get size(): number {
    return this.#size
  }

  set size(value: string | number | boolean) {
    let num = Number(value)

    // Don't allow NaN and stuff.
    if (!Number.isFinite(num)) {
      this.#size = 0
      return
    }

    this.#size = num
  }
}
```

在上面的示例中，我们的 `set` 访问器接受更宽泛的类型集合（`string`、`boolean` 和 `number`），而我们的 `get` 访问器始终保证返回 `number`。
现在，我们终于可以毫无错误地向这些属性赋值其他类型了！

```ts twoslash
class Thing {
  #size = 0

  get size(): number {
    return this.#size
  }

  set size(value: string | number | boolean) {
    let num = Number(value)

    // Don't allow NaN and stuff.
    if (!Number.isFinite(num)) {
      this.#size = 0
      return
    }

    this.#size = num
  }
}
// ---cut---
let thing = new Thing()

// Assigning other types to `thing.size` works!
thing.size = 'hello'
thing.size = true
thing.size = 42

// Reading `thing.size` always produces a number!
let mySize: number = thing.size
```

当判断同名属性之间的类型关系时，TypeScript 只会使用“读取”类型（例如上面 `get` 访问器上的类型）。
“写入”类型仅在直接向属性赋值时才会被考量。

需要注意的是，这种模式并不局限于类。
你也可以在对象字面量中编写具有不同类型的 getter 和 setter。

```ts
function makeThing(): Thing {
  let size = 0
  return {
    get size(): number {
      return size
    },
    set size(value: string | number | boolean) {
      let num = Number(value)

      // Don't allow NaN and stuff.
      if (!Number.isFinite(num)) {
        size = 0
        return
      }

      size = num
    },
  }
}
```

事实上，我们还向接口和对象类型添加了语法，以支持属性上不同的读取/写入类型。

```ts
// Now valid!
interface Thing {
  get size(): number
  set size(value: number | string | boolean)
}
```

为属性读取和写入使用不同类型的唯一限制是：用于读取属性的类型必须能够赋值给写入属性的类型。
换言之，getter 的类型必须可以赋值给 setter 的类型。
这确保了一定程度的一致性，使得属性始终可以赋值给自身。

欲了解有关该特性的更多信息，请查看[实现该特性的 Pull Request](https://github.com/microsoft/TypeScript/pull/42425)。

## `override` 关键字与 `--noImplicitOverride` 选项

在 JavaScript 中继承类时，重写方法是件轻而易举的事（这里带有 super 的双关）——但遗憾的是，你可能会遇到一些易犯的错误。

其中最主要的一个就是重命名时遗漏了同步更新。
例如以下类：

```ts
class SomeComponent {
  show() {
    // ...
  }
  hide() {
    // ...
  }
}

class SpecializedComponent extends SomeComponent {
  show() {
    // ...
  }
  hide() {
    // ...
  }
}
```

`SpecializedComponent` 继承自 `SomeComponent`，并重写了 `show` 和 `hide` 方法。
如果有人决定移除 `show` 和 `hide`，并将其替换为单一方法，会发生什么？

```diff
 class SomeComponent {
-    show() {
-        // ...
-    }
-    hide() {
-        // ...
-    }
+    setVisible(value: boolean) {
+        // ...
+    }
 }
 class SpecializedComponent extends SomeComponent {
     show() {
         // ...
     }
     hide() {
         // ...
     }
 }
```

_糟糕！_
我们的 `SpecializedComponent` 没有得到更新。
现在它平白添加了这两个可能永远不会被调用的无用方法 `show` 和 `hide`。

造成该问题的部分原因在于，用户无法明确表明自己是打算添加一个新方法，还是要重写已有的方法。
因此，TypeScript 4.3 增加了 `override` 关键字。

```ts
class SpecializedComponent extends SomeComponent {
  override show() {
    // ...
  }
  override hide() {
    // ...
  }
}
```

当一个方法被标记为 `override` 时，TypeScript 会始终确保基类中存在同名的方法。

```ts twoslash
// @noImplicitOverride
// @errors: 4113
class SomeComponent {
  setVisible(value: boolean) {
    // ...
  }
}
class SpecializedComponent extends SomeComponent {
  override show() {}
}
```

这是一项重大改进，但如果你*忘记*在方法上添加 `override`，它就无能为力了——而这也正是用户经常会踩到的一个大坑。

例如，你可能会在不知情的情况下意外“覆盖”了基类中已存在的方法：

```ts
class Base {
  someHelperMethod() {
    // ...
  }
}

class Derived extends Base {
  // Oops! We weren't trying to override here,
  // we just needed to write a local helper method.
  someHelperMethod() {
    // ...
  }
}
```

因此，TypeScript 4.3 *还*提供了一个新的 [`noImplicitOverride`](/tsconfig#noImplicitOverride) 编译选项。
当启用此选项时，除非显式使用 `override` 关键字，否则重写父类的任何方法都会报错。
在上面的最后一个示例中，TypeScript 在启用 [`noImplicitOverride`](/tsconfig#noImplicitOverride) 时会报错，提示我们可能需要重命名 `Derived` 内部的方法。

我们衷心感谢社区在此处的实现工作。
这些功能由 [Wenlu Wang](https://github.com/Kingwl) 在[一个 Pull Request](https://github.com/microsoft/TypeScript/pull/39669) 中实现，而早前由 [Paul Cody Johnston](https://github.com/pcj) 提交的仅实现 `override` 关键字的 Pull Request 则奠定了方向并引发了讨论。
我们对他们为这些特性付出的时间与精力深表感谢。

## 模板字符串类型改进

在近期的版本中，TypeScript 引入了一种新的类型构造：模板字符串类型（Template String Types）。
这些类型要么通过拼接来构造新的类似字符串的类型……

```ts
type Color = 'red' | 'blue'
type Quantity = 'one' | 'two'

type SeussFish = `${Quantity | Color} fish`
// same as
//   type SeussFish = "one fish" | "two fish"
//                  | "red fish" | "blue fish";
```

……要么匹配其他类似字符串类型的模式。

```ts
declare let s1: `${number}-${number}-${number}`
declare let s2: `1-2-3`

// Works!
s1 = s2
```

我们所做的第一项改动是关于 TypeScript 何时推断模板字符串类型。
当模板字符串被类似字符串字面量的类型*提供上下文类型*时（即当 TypeScript 发现我们将模板字符串传递给接收字面量类型的位置时），它将尝试赋予该表达式模板类型。

```ts
function bar(s: string): `hello ${string}` {
  // Previously an error, now works!
  return `hello ${s}`
}
```

在推断类型且类型参数 `extends string` 时，此特性同样会生效：

```ts
declare let s: string
declare function f<T extends string>(x: T): T

// Previously: string
// Now       : `hello ${string}`
let x2 = f(`hello ${s}`)
```

第二项重大改动是：TypeScript 现在可以更好地关联不同模板字符串类型，并在它们之间进行*相互推断*。

来看下面这个示例代码：

```ts
declare let s1: `${number}-${number}-${number}`
declare let s2: `1-2-3`
declare let s3: `${number}-2-3`

s1 = s2
s1 = s3
```

当针对像 `s2` 这样的字符串字面量类型进行检查时，TypeScript 能够匹配字符串内容并判断出在第一次赋值中 `s2` 与 `s1` 兼容；
然而一旦遇到另一个模板字符串，它就会直接放弃。
因此，像将 `s3` 赋值给 `s1` 这样的操作过去根本行不通。

现在，TypeScript 会切实验证模板字符串的各个部分是否能够成功匹配。
你现在可以混用具有不同占位替换的模板字符串，TypeScript 会很好地判断它们之间是否真正兼容。

```ts
declare let s1: `${number}-${number}-${number}`
declare let s2: `1-2-3`
declare let s3: `${number}-2-3`
declare let s4: `1-${number}-3`
declare let s5: `1-2-${number}`
declare let s6: `${number}-2-${number}`

// Now *all of these* work!
s1 = s2
s1 = s3
s1 = s4
s1 = s5
s1 = s6
```

在完成这项工作的同时，我们还确保添加了更强大的推断能力。
你可以看到这些特性的实际应用示例：

```ts
declare function foo<V extends string>(arg: `*${V}*`): V

function test<T extends string>(s: string, n: number, b: boolean, t: T) {
  let x1 = foo('*hello*') // "hello"
  let x2 = foo('**hello**') // "*hello*"
  let x3 = foo(`*${s}*` as const) // string
  let x4 = foo(`*${n}*` as const) // `${number}`
  let x5 = foo(`*${b}*` as const) // "true" | "false"
  let x6 = foo(`*${t}*` as const) // `${T}`
  let x7 = foo(`**${s}**` as const) // `*${string}*`
}
```

欲了解更多信息，请参阅[利用上下文类型的原始 Pull Request](https://github.com/microsoft/TypeScript/pull/43376)，以及[改进模板类型之间的推断与检查的 Pull Request](https://github.com/microsoft/TypeScript/pull/43361)。

## ECMAScript `#private` 类元素

TypeScript 4.3 扩展了类中可以使用 `#private` `#names`（私有名称）的元素范围，使它们在运行时真正成为私有的。
除属性外，方法和访问器现在也可以赋予私有名称。

```ts
class Foo {
  #someMethod() {
    //...
  }

  get #someValue() {
    return 100
  }

  publicMethod() {
    // These work.
    // We can access private-named members inside this class.
    this.#someMethod()
    return this.#someValue
  }
}

new Foo().#someMethod()
//        ~~~~~~~~~~~
// error!
// Property '#someMethod' is not accessible
// outside class 'Foo' because it has a private identifier.

new Foo().#someValue
//        ~~~~~~~~~~
// error!
// Property '#someValue' is not accessible
// outside class 'Foo' because it has a private identifier.
```

更广泛地，静态成员现在也可以拥有私有名称。

```ts
class Foo {
  static #someMethod() {
    // ...
  }
}

Foo.#someMethod()
//  ~~~~~~~~~~~
// error!
// Property '#someMethod' is not accessible
// outside class 'Foo' because it has a private identifier.
```

该特性由彭博社（Bloomberg）的朋友在[一个 Pull Request](https://github.com/microsoft/TypeScript/pull/42458) 中贡献——由 [Titian Cernicova-Dragomir](https://github.com/dragomirtitian) 和 [Kubilay Kahveci](https://github.com/mkubilayk) 编写，并得到了 [Joey Watts](https://github.com/joeywatts)、[Rob Palmer](https://github.com/robpalme) 与 [Tim McClure](https://github.com/tim-mc) 的专业支持与协助。
我们向他们所有人表示由衷的感谢！

## `ConstructorParameters` 支持抽象类

在 TypeScript 4.3 中，`ConstructorParameters` 工具类型现已支持 `abstract` 抽象类。

```ts
abstract class C {
  constructor(a: string, b: number) {
    // ...
  }
}

// Has the type '[a: string, b: number]'.
type CParams = ConstructorParameters<typeof C>
```

这得益于 TypeScript 4.2 中完成的工作，其中构造签名可以被标记为抽象：

```ts
type MyConstructorOf<T> = {
    abstract new(...args: any[]): T;
}

// or using the shorthand syntax:

type MyConstructorOf<T> = abstract new (...args: any[]) => T;
```

你可以[在 GitHub 上查看该改动的更多详情](https://github.com/microsoft/TypeScript/pull/43380)。

## 泛型的上下文类型收窄

TypeScript 4.3 现已针对泛型值包含更智能的类型收窄逻辑。
这使得 TypeScript 能够接受更多编程模式，有时甚至能捕获潜在错误。

作为背景示例，假设我们正在编写一个名为 `makeUnique` 的函数。
它接收一个包含元素的 `Set` 或 `Array`；如果传入的是 `Array`，它会根据某个比较函数对该 `Array` 进行排序并去除重复项。
完成上述操作后，它将返回原始集合。

```ts
function makeUnique<T>(
  collection: Set<T> | T[],
  comparer: (x: T, y: T) => number,
): Set<T> | T[] {
  // Early bail-out if we have a Set.
  // We assume the elements are already unique.
  if (collection instanceof Set) {
    return collection
  }

  // Sort the array, then remove consecutive duplicates.
  collection.sort(comparer)
  for (let i = 0; i < collection.length; i++) {
    let j = i
    while (
      j < collection.length &&
      comparer(collection[i], collection[j + 1]) === 0
    ) {
      j++
    }
    collection.splice(i + 1, j - i)
  }
  return collection
}
```

我们先撇开该函数具体实现中的细节不谈，假设它是出于更大规模应用的需求而产生的。
你可能会注意到，该函数的签名并没有捕获 `collection` 的原始类型。
我们可以通过在原来写 `Set<T> | T[]` 的地方添加一个名为 `C` 的类型参数来实现这一点：

```diff
- function makeUnique<T>(collection: Set<T> | T[], comparer: (x: T, y: T) => number): Set<T> | T[]
+ function makeUnique<T, C extends Set<T> | T[]>(collection: C, comparer: (x: T, y: T) => number): C
```

在 TypeScript 4.2 及更早版本中，一旦尝试这样写，你就会遭遇一大堆编译报错：

```ts
function makeUnique<T, C extends Set<T> | T[]>(
  collection: C,
  comparer: (x: T, y: T) => number,
): C {
  // Early bail-out if we have a Set.
  // We assume the elements are already unique.
  if (collection instanceof Set) {
    return collection
  }

  // Sort the array, then remove consecutive duplicates.
  collection.sort(comparer)
  //         ~~~~
  // error: Property 'sort' does not exist on type 'C'.
  for (let i = 0; i < collection.length; i++) {
    //                             ~~~~~~
    // error: Property 'length' does not exist on type 'C'.
    let j = i
    while (
      j < collection.length &&
      comparer(collection[i], collection[j + 1]) === 0
    ) {
      //                    ~~~~~~
      // error: Property 'length' does not exist on type 'C'.
      //                                       ~~~~~~~~~~~~~  ~~~~~~~~~~~~~~~~~
      // error: Element implicitly has an 'any' type because expression of type 'number'
      //        can't be used to index type 'Set<T> | T[]'.
      j++
    }
    collection.splice(i + 1, j - i)
    //         ~~~~~~
    // error: Property 'splice' does not exist on type 'C'.
  }
  return collection
}
```

哎呀，全是错误！
为什么 TypeScript 对我们如此苛刻？

问题在于，当我们执行 `collection instanceof Set` 检查时，我们期望它作为一个类型守卫，根据所在分支将类型从 `Set<T> | T[]` 收窄为 `Set<T>` 或 `T[]`；
然而，我们面对的并不是 `Set<T> | T[]`，而是试图收窄类型为 `C` 的泛型值 `collection`。

这是一个非常微妙的区别，但影响巨大。
TypeScript 不能简单地提取 `C` 的约束（即 `Set<T> | T[]`）并对其进行收窄。
如果 TypeScript *确实*尝试从 `Set<T> | T[]` 进行收窄，它会在每个分支中忘记 `collection` 同时也是 `C`，因为没有简便的方法来保留该信息。
假使 TypeScript 采取了那种方式，上面的示例就会以另一种方式崩溃：
在函数期望返回类型为 `C` 的返回位置，我们在各个分支中得到的将分别是 `Set<T>` 和 `T[]`，而 TypeScript 会拒绝这种返回。

```ts
function makeUnique<T>(
  collection: Set<T> | T[],
  comparer: (x: T, y: T) => number,
): Set<T> | T[] {
  // Early bail-out if we have a Set.
  // We assume the elements are already unique.
  if (collection instanceof Set) {
    return collection
    //     ~~~~~~~~~~
    // error: Type 'Set<T>' is not assignable to type 'C'.
    //          'Set<T>' is assignable to the constraint of type 'C', but
    //          'C' could be instantiated with a different subtype of constraint 'Set<T> | T[]'.
  }

  // ...

  return collection
  //     ~~~~~~~~~~
  // error: Type 'T[]' is not assignable to type 'C'.
  //          'T[]' is assignable to the constraint of type 'C', but
  //          'C' could be instantiated with a different subtype of constraint 'Set<T> | T[]'.
}
```

那么 TypeScript 4.3 是如何改变这一现状的呢？
基本上，在编写代码的几个关键场景中，类型系统真正关心的只有类型的约束。
例如，当我们编写 `collection.length` 时，TypeScript 并不在乎 `collection` 的类型是 `C`，它只在乎可用的属性，而这些属性是由约束 `T[] | Set<T>` 决定的。

在类似这种情况下，TypeScript 会提取该约束收窄后的类型，因为这能提供你所关心的属性信息；
而在任何其他情况下，我们只需尝试收窄原始的泛型类型（通常最终仍保持原始的泛型类型）。

换句话说，根据你使用泛型值的方式，TypeScript 的收窄策略会有所不同。
最终的结果是：上述整个示例在编译时没有任何类型检查错误。

欲了解更多详情，可以[查看 GitHub 上的原始 Pull Request](https://github.com/microsoft/TypeScript/pull/43183)。

## 针对始终为真的 Promise 检查

在启用了 [`strictNullChecks`](/tsconfig#strictNullChecks) 的情况下，在条件判断中检查 `Promise` 是否为“真值（truthy）”将触发错误。

```ts
async function foo(): Promise<boolean> {
  return false
}

async function bar(): Promise<string> {
  if (foo()) {
    //  ~~~~~
    // Error!
    // This condition will always return true since
    // this 'Promise<boolean>' appears to always be defined.
    // Did you forget to use 'await'?
    return 'true'
  }
  return 'false'
}
```

[该变更](https://github.com/microsoft/TypeScript/pull/39175) 由 [Jack Works](https://github.com/Jack-Works) 贡献，我们向其表示衷心的感谢！

## `static` 索引签名

索引签名允许我们在值上设置超出类型显式声明范围的更多属性。

```ts
class Foo {
  hello = 'hello'
  world = 1234;

  // This is an index signature:
  [propName: string]: string | number | undefined
}

let instance = new Foo()

// Valid assignment
instance['whatever'] = 42

// Has type 'string | number | undefined'.
let x = instance['something']
```

在此之前，索引签名只能在类的实例端进行声明。
感谢 [Wenlu Wang](https://github.com/microsoft/TypeScript/pull/37797) 提交的 [Pull Request](https://github.com/microsoft/TypeScript/pull/37797)，索引签名现在可以声明为 `static`。

```ts
class Foo {
  static hello = 'hello'
  static world = 1234

  static [propName: string]: string | number | undefined
}

// Valid.
Foo['whatever'] = 42

// Has type 'string | number | undefined'
let x = Foo['something']
```

类静态端的索引签名与实例端遵循相同的规则——即所有其他静态属性都必须与索引签名兼容。

```ts
class Foo {
  static prop = true
  //     ~~~~
  // Error! Property 'prop' of type 'boolean'
  // is not assignable to string index type
  // 'string | number | undefined'.

  static [propName: string]: string | number | undefined
}
```

## `.tsbuildinfo` 文件体积缩减

在 TypeScript 4.3 中，作为 [`incremental`](/tsconfig#incremental) 增量构建一部分生成的 `.tsbuildinfo` 文件体积将显著减小。
这得益于对内部格式的多项优化：在整个文件中使用带有数字标识符的表格，而不是反复记录完整路径及类似信息。
该项工作由 [Tobias Koppers](https://github.com/sokra) 在其 [Pull Request](https://github.com/microsoft/TypeScript/pull/43079) 中率先发起，并激发了[随后的 Pull Request](https://github.com/microsoft/TypeScript/pull/43155) 和[进一步的优化](https://github.com/microsoft/TypeScript/pull/43695)。

我们看到了 `.tsbuildinfo` 文件大小的显著缩减，例如：

- 从 1MB 降至 411 KB
- 从 14.9MB 降至 1MB
- 从 1345MB 降至 467MB

毋庸置疑，文件体积的缩减同时也带来了更快的构建速度。

## `--incremental` 和 `--watch` 编译中的更懒计算

[`incremental`](/tsconfig#incremental) 和 `--watch` 模式存在的一个问题是：虽然它们能加快后续编译的速度，但首次编译可能会略微变慢——在某些情况下甚至会明显变慢。
这是因为这些模式必须进行大量的记账工作，计算当前项目的信息，有时还需要将这些数据保存到 `.tsbuildinfo` 文件中以供后续构建使用。

因此，除了缩减 `.tsbuildinfo` 的文件体积外，TypeScript 4.3 还对 [`incremental`](/tsconfig#incremental) 和 `--watch` 模式做出了一些改进，使得开启这些标志时的项目首次构建速度与普通构建一样快！
为此，许多原本需要预先计算的信息被改为在后续构建中按需计算。
虽然这可能会给后续构建增加少量开销，但 TypeScript 的 [`incremental`](/tsconfig#incremental) 和 `--watch` 功能通常只会在极少量的文件集上运行，并且所需的信息会在之后保存。
从某种意义上说，在多次更新文件后，[`incremental`](/tsconfig#incremental) 和 `--watch` 构建将会逐步“预热”并变得更加快速。

在一个包含 3000 个文件的代码仓库中，**这使初始构建时间缩短至接近原先的三分之一**！

[该项工作最初由 Tobias Koppers 发起](https://github.com/microsoft/TypeScript/pull/42960)，并在此基础上促成了此功能的[最终变更](https://github.com/microsoft/TypeScript/pull/43314)。
我们非常感谢 Tobias 帮助我们发现了这些改进机会！

## 导入语句补全

用户在 JavaScript 中编写导入和导出语句时遇到的最大痛点之一就是书写顺序——具体而言，导入语句的写法是：

```ts
import { func } from './module.js'
```

而不是：

```ts
from "./module.js" import { func };
```

这在从头编写完整的导入语句时会带来一些痛苦，因为自动补全无法正常工作。
例如，如果你一开始键入 `import {`，TypeScript 完全不知道你打算从哪个模块导入，因此无法提供限定范围的补全候选项。

为了缓解这一问题，我们借助了自动导入（auto-import）的强大能力！
自动导入本身就已经解决了无法从特定模块缩小补全范围的问题——它的核心目的就是提供所有可能的导出，并在文件顶部自动插入导入语句。

因此，现在当你开始编写一个尚未填写路径的 `import` 语句时，我们将为你提供一个可能的导入列表。
当你确认补全时，我们将补全整个导入语句，包括你正准备书写的路径。

![导入语句补全动图](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2021/05/auto-import-statement-4-3.gif)

该功能需要编辑器进行专门支持。
你可以通过使用最新版的 [Visual Studio Code Insiders 版本](https://code.visualstudio.com/insiders/) 来体验该特性。

欲了解更多信息，请查看[实现该特性的 Pull Request](https://github.com/microsoft/TypeScript/pull/43149)！

## 编辑器支持 `@link` 标签

TypeScript 现在能够理解 `@link` 标签，并将尝试解析它们所链接到的声明。
这意味着你可以在 `@link` 标签内的名称上悬停以获取快速信息，或者使用跳转到定义、查找所有引用等命令。

例如，在下面的示例中，你可以在 `@link plantCarrot` 中的 `plantCarrot` 上执行跳转到定义，支持 TypeScript 的编辑器将跳转到 `plantCarrot` 的函数声明处。

```ts
/**
 * To be called 70 to 80 days after {@link plantCarrot}.
 */
function harvestCarrot(carrot: Carrot) {}

/**
 * Call early in spring for best results. Added in v2.1.0.
 * @param seed Make sure it's a carrot seed!
 */
function plantCarrot(seed: Seed) {
  // TODO: some gardening
}
```

![在 @link 标签上跳转到定义和请求快速信息](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2021/05/link-tag-4-3.gif)

欲了解更多信息，请参阅 [GitHub 上的 Pull Request](https://github.com/microsoft/TypeScript/pull/41877)！

## 跳转到非 JavaScript 文件路径的定义

许多打包加载器允许用户在应用程序中通过 JavaScript 导入来引入资源文件。
它们通常写成类似 `import "./styles.css"` 的形式。

在此之前，TypeScript 的编辑器功能甚至不会尝试读取此类文件，因此跳转到定义通常会失败。
充其量，如果能找到类似声明的话，跳转到定义也只会跳转到 `declare module "*.css"` 这样的声明处。

现在，当你在相对文件路径上执行跳转到定义时，TypeScript 的语言服务会尝试跳转到正确的文件，即便它们不是 JavaScript 或 TypeScript 文件！
不妨在导入 CSS、SVG、PNG、字体文件、Vue 文件等场景中尝试一下。

欲了解更多信息，请查看[实现该特性的 Pull Request](https://github.com/microsoft/TypeScript/pull/42539)。

## 破坏性改动

### `lib.d.ts` 变更

与每个 TypeScript 版本一样，`lib.d.ts` 的声明（特别是为 Web 上下文生成的声明）发生了一些改动。
在本次发布中，我们借助了 [Mozilla 的 browser-compat-data](https://github.com/mdn/browser-compat-data) 移除了所有浏览器均未实现的 API。
尽管你不太可能用到它们，但诸如 `Account`、`AssertionOptions`、`RTCStatsEventInit`、`MSGestureEvent`、`DeviceLightEvent`、`MSPointerEvent`、`ServiceWorkerMessageEvent` 和 `WebAuthentication` 等 API 均已从 `lib.d.ts` 中移除。
[在此处对此有详细讨论](https://github.com/microsoft/TypeScript-DOM-lib-generator/issues/991)。

https://github.com/microsoft/TypeScript-DOM-lib-generator/issues/991

### `useDefineForClassFields` 在 `esnext` 及后续 `es2022` 中默认为 true

2021 年，类字段特性被纳入 JavaScript 规范，其行为与 TypeScript 原先的实现有所不同。为此，TypeScript 3.7 添加了一个编译选项（[`useDefineForClassFields`](/tsconfig#useDefineForClassFields)），以使生成的 JavaScript 迁移并契合 JavaScript 标准行为。

既然该特性已正式成为 JavaScript 的一部分，我们将 ES2022 及以上版本（包括 ESNext）的该选项默认值更改为 `true`。

### 针对始终为真的 Promise 检查报错

在启用了 [`strictNullChecks`](/tsconfig#strictNullChecks) 的情况下，在条件检查中使用一个看似始终有定义的 `Promise` 现在会被视为错误。

```ts
declare var p: Promise<number>

if (p) {
  //  ~
  // Error!
  // This condition will always return true since
  // this 'Promise<number>' appears to always be defined.
  //
  // Did you forget to use 'await'?
}
```

欲了解更多详情，请[参阅原始变更](https://github.com/microsoft/TypeScript/pull/39175)。

### 联合枚举不能与任意数字进行比较

当某些 `enum` 的成员是自动填充或简单书写的值时，它们会被视为*联合枚举（union enum）*。
在这些情况下，枚举能够记忆它可能代表的每一个具体值。

在 TypeScript 4.3 中，如果将联合枚举类型的值与一个永远不可能相等的数字字面量进行比较，类型检查器将报告错误。

```ts
enum E {
  A = 0,
  B = 1,
}

function doSomething(x: E) {
  // Error! This condition will always return 'false' since the types 'E' and '-1' have no overlap.
  if (x === -1) {
    // ...
  }
}
```

作为变通方案，你可以重写类型注解以包含相应的字面量类型：

```ts
enum E {
  A = 0,
  B = 1,
}

// Include -1 in the type, if we're really certain that -1 can come through.
function doSomething(x: E | -1) {
  if (x === -1) {
    // ...
  }
}
```

你也可以在值上使用类型断言：

```ts
enum E {
  A = 0,
  B = 1,
}

function doSomething(x: E) {
  // Use a type assertion on 'x' because we know we're not actually just dealing with values from 'E'.
  if ((x as number) === -1) {
    // ...
  }
}
```

或者，你可以重新声明你的枚举，为其指定非平凡的初始化表达式，以便任何数字都可以赋值给该枚举并与其进行比较。如果枚举的意图仅仅是指定几个已知常量值，这可能会很有用。

```ts
enum E {
  // the leading + on 0 opts TypeScript out of inferring a union enum.
  A = +0,
  B = 1,
}
```

欲了解更多详情，请[参阅原始变更](https://github.com/microsoft/TypeScript/pull/42472)。
