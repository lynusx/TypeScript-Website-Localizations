---
title: 声明合并
layout: docs
permalink: /zh/docs/handbook/declaration-merging.html
oneline: 命名空间与接口的合并机制
translatable: true
---

## 介绍

TypeScript 中有一些独特的概念，用于在类型层面描述 JavaScript 对象的结构。
其中一个尤为独特的概念就是“声明合并”（declaration merging）。
理解这一概念有助于你在处理现有的 JavaScript 代码时更加得心应手。
它也为你开启了通向更高级抽象概念的大门。

在本文中，“声明合并”是指编译器将以相同名称声明的两个独立声明合并为一个单一的定义。
这个合并后的定义具备两个原始声明的所有特性。
可以合并的声明数量没有限制，并不仅限于两个声明。

## 基本概念

在 TypeScript 中，声明会在以下三个分组中至少创建一个实体：命名空间（namespace）、类型（type）或值（value）。
创建命名空间的声明会生成一个命名空间，其中包含使用点号表示法访问的名称。
创建类型的声明顾名思义：它们会创建一个具有所声明结构并绑定到指定名称的可见类型。
最后，创建值的声明会创建在输出的 JavaScript 中可见的值。

| 声明类型               | 命名空间 | 类型 | 值  |
| ---------------------- | :------: | :--: | :-: |
| 命名空间（Namespace）  |    X     |      |  X  |
| 类（Class）            |          |  X   |  X  |
| 枚举（Enum）           |          |  X   |  X  |
| 接口（Interface）      |          |  X   |     |
| 类型别名（Type Alias） |          |  X   |     |
| 函数（Function）       |          |      |  X  |
| 变量（Variable）       |          |      |  X  |

了解每个声明所创建的实体，有助于你理解在进行声明合并时究竟合并了什么。

## 合并接口

最简单、也可能是最常见的声明合并类型是接口合并。
在最基本的层面上，合并操作会将两个声明中的成员机械地整合到一个同名的单一接口中。

```ts
interface Box {
  height: number
  width: number
}

interface Box {
  scale: number
}

let box: Box = { height: 5, width: 6, scale: 10 }
```

接口中的非函数成员应当是唯一的。
如果不唯一，它们必须具有相同的类型。
如果两个接口都声明了同名但类型不同的非函数成员，编译器将会报错。

对于函数成员，同名的每个函数成员都会被视为同一函数的重载描述。
同样需要注意的是，当接口 `A` 与后声明的接口 `A` 合并时，后声明的接口将拥有比先声明的接口更高的优先级。

也就是说，在以下示例中：

```ts
interface Cloner {
  clone(animal: Animal): Animal
}

interface Cloner {
  clone(animal: Sheep): Sheep
}

interface Cloner {
  clone(animal: Dog): Dog
  clone(animal: Cat): Cat
}
```

这三个接口合并后会生成如下单一声明：

```ts
interface Cloner {
  clone(animal: Dog): Dog
  clone(animal: Cat): Cat
  clone(animal: Sheep): Sheep
  clone(animal: Animal): Animal
}
```

请注意，每个组内部的元素保持了原有的顺序，但组与组之间合并时，后声明的重载签名集合会排在前面。

该规则的一个例外是特定化签名（specialized signatures）。
如果一个签名的参数类型为*单一*字符串字面量类型（例如，不能是字符串字面量的联合类型），那么该签名将被提升到其合并后重载列表的最顶端。

例如，以下接口将会合并在一起：

```ts
interface Document {
  createElement(tagName: any): Element
}
interface Document {
  createElement(tagName: 'div'): HTMLDivElement
  createElement(tagName: 'span'): HTMLSpanElement
}
interface Document {
  createElement(tagName: string): HTMLElement
  createElement(tagName: 'canvas'): HTMLCanvasElement
}
```

合并后得到的 `Document` 声明如下所示：

```ts
interface Document {
  createElement(tagName: 'canvas'): HTMLCanvasElement
  createElement(tagName: 'div'): HTMLDivElement
  createElement(tagName: 'span'): HTMLSpanElement
  createElement(tagName: string): HTMLElement
  createElement(tagName: any): Element
}
```

## 合并命名空间

与接口类似，同名的命名空间也会合并它们的成员。
由于命名空间同时创建了命名空间和值，因此我们需要理解这两者是如何合并的。

为了合并命名空间，每个命名空间中声明的导出接口的类型定义本身会进行合并，形成一个内部包含已合并接口定义的单一命名空间。

为了合并命名空间的值，在每个声明位置上，如果已经存在指定名称的命名空间，则通过将第二个命名空间的导出成员添加到第一个命名空间中，来对现有命名空间进行进一步扩展。

在这个例子中，`Animals` 的声明合并：

```ts
namespace Animals {
  export class Zebra {}
}

namespace Animals {
  export interface Legged {
    numberOfLegs: number
  }
  export class Dog {}
}
```

等同于：

```ts
namespace Animals {
  export interface Legged {
    numberOfLegs: number
  }

  export class Zebra {}
  export class Dog {}
}
```

这种命名空间合并模型是一个很好的起点，但我们还需要了解未导出成员会发生什么。
未导出成员仅在原始（未合并）的命名空间中可见。这意味着在合并之后，来自其他声明的已合并成员无法访问这些未导出成员。

我们可以通过以下示例更清楚地看到这一点：

```ts
namespace Animal {
  let haveMuscles = true

  export function animalsHaveMuscles() {
    return haveMuscles
  }
}

namespace Animal {
  export function doAnimalsHaveMuscles() {
    return haveMuscles // 错误，因为此处无法访问 haveMuscles
  }
}
```

因为 `haveMuscles` 没有被导出，所以只有处于同一个未合并命名空间中的 `animalsHaveMuscles` 函数才能访问该符号。
而 `doAnimalsHaveMuscles` 函数尽管也是合并后的 `Animal` 命名空间的一部分，却无法访问这个未导出的成员。

## 将命名空间与类、函数和枚举合并

命名空间非常灵活，还可以与其他类型的声明进行合并。
为此，命名空间声明必须紧随在其要合并的声明之后。合并后的声明同时拥有两种声明类型的属性。
TypeScript 利用这一能力来对 JavaScript 以及其他编程语言中的某些模式进行建模。

### 将命名空间与类合并

这为开发者提供了一种描述内部类（inner class）的方法。

```ts
class Album {
  label: Album.AlbumLabel
}
namespace Album {
  export class AlbumLabel {}
}
```

合并成员的可见性规则与[合并命名空间](./declaration-merging.html#merging-namespaces)小节中所述相同，因此我们必须导出 `AlbumLabel` 类，合并后的类才能访问它。
最终的效果是在一个类内部管理另一个类。
你也可以使用命名空间为现有类添加更多的静态成员。

除了内部类模式外，你可能还熟悉 JavaScript 中先创建一个函数，然后通过在该函数上添加属性来对其进行扩展的做法。
TypeScript 通过声明合并，以类型安全的方式构建此类定义。

```ts
function buildLabel(name: string): string {
  return buildLabel.prefix + name + buildLabel.suffix
}

namespace buildLabel {
  export let suffix = ''
  export let prefix = 'Hello, '
}

console.log(buildLabel('Sam Smith'))
```

类似地，命名空间也可以用来为枚举扩展静态成员：

```ts
enum Color {
  red = 1,
  green = 2,
  blue = 4,
}

namespace Color {
  export function mixColor(colorName: string) {
    if (colorName == 'yellow') {
      return Color.red + Color.green
    } else if (colorName == 'white') {
      return Color.red + Color.green + Color.blue
    } else if (colorName == 'magenta') {
      return Color.red + Color.blue
    } else if (colorName == 'cyan') {
      return Color.green + Color.blue
    }
  }
}
```

## 不允许的合并

TypeScript 中并非所有的合并都被允许。
目前，类不能与其他类或变量进行合并。
有关模拟类合并的信息，请参阅 [TypeScript 中的混入（Mixins）](/docs/handbook/mixins.html) 章节。

## 模块扩展（Module Augmentation）

尽管 JavaScript 模块不支持合并，但你可以通过先导入现有对象然后再更新它们的方式来给它们打补丁（patch）。
来看一个简单的 Observable 示例：

```ts
// observable.ts
export class Observable<T> {
  // ... 具体实现留给读者作为练习 ...
}

// map.ts
import { Observable } from './observable'
Observable.prototype.map = function (f) {
  // ... 给读者的另一个练习
}
```

这在 TypeScript 中也能正常运行，但编译器并不知道 `Observable.prototype.map` 的存在。
你可以使用模块扩展来将它告知编译器：

```ts
// observable.ts
export class Observable<T> {
  // ... 具体实现留给读者作为练习 ...
}

// map.ts
import { Observable } from './observable'
declare module './observable' {
  interface Observable<T> {
    map<U>(f: (x: T) => U): Observable<U>
  }
}
Observable.prototype.map = function (f) {
  // ... 给读者的另一个练习
}

// consumer.ts
import { Observable } from './observable'
import './map'
let o: Observable<number>
o.map((x) => x.toFixed())
```

模块名称的解析方式与 `import`/`export` 中的模块说明符相同。
有关更多信息，请参阅[模块（Modules）](/docs/handbook/modules.html)。
然后，扩展中的声明就会合并，就如同它们与原始声明在同一个文件中声明一样。

但是，需要牢记两个限制：

1. 不能在扩展中声明新的顶级声明——只能对现有声明打补丁。
2. 默认导出（default exports）也不能被扩展，只有具名导出（named exports）才能被扩展（因为你需要通过导出的名称来对其进行扩展，而 `default` 是一个保留字——详情请参阅 [#14080](https://github.com/Microsoft/TypeScript/issues/14080)）。

### 全局扩展（Global augmentation）

你也可以在模块内部向全局作用域添加声明：

```ts
// observable.ts
export class Observable<T> {
  // ... 依然没有实现 ...
}

declare global {
  interface Array<T> {
    toObservable(): Observable<T>
  }
}

Array.prototype.toObservable = function () {
  // ...
}
```

全局扩展与模块扩展具有相同的行为和限制。
