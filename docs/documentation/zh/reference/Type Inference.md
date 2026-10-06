---
title: 类型推断
layout: docs
permalink: /zh/docs/handbook/type-inference.html
oneline: TypeScript 中代码流分析的工作原理
translatable: true
---

在 TypeScript 中，在没有显式类型注解的情况下，有若干处会使用类型推断来提供类型信息。例如，在这段代码中：

```ts twoslash
let x = 3
//  ^?
```

变量 `x` 的类型被推断为 `number`。
这种推断发生在初始化变量和成员、设置参数默认值以及确定函数返回类型时。

在大多数情况下，类型推断都是直截了当的。
在接下来的几节中，我们将探讨类型推断的一些细微机制。

## 最佳通用类型（Best common type）

当需要从多个表达式推断类型时，这些表达式的类型会被用来计算出一个“最佳通用类型”。例如：

```ts twoslash
let x = [0, 1, null]
//  ^?
```

要推断上例中 `x` 的类型，我们必须考虑每个数组元素的类型。
这里数组的类型有两个候选值：`number` 和 `null`。
最佳通用类型算法会考量各个候选类型，并选出与所有其他候选类型都兼容的类型。

由于最佳通用类型必须从提供的候选类型中选出，因此在某些情况下，各类型虽然共享通用结构，但没有某一个类型是所有候选类型的超类型。例如：

```ts twoslash
// @strict: false
class Animal {}
class Rhino extends Animal {
  hasHorn: true
}
class Elephant extends Animal {
  hasTrunk: true
}
class Snake extends Animal {
  hasLegs: false
}
// ---cut---
let zoo = [new Rhino(), new Elephant(), new Snake()]
//    ^?
```

理想情况下，我们可能希望 `zoo` 被推断为 `Animal[]`，但因为数组中没有严格属于 `Animal` 类型的对象，所以我们无法推断出该数组元素的具体类型。
为了纠正这一点，当没有任何一个类型是所有其他候选类型的超类型时，请显式提供类型：

```ts twoslash
// @strict: false
class Animal {}
class Rhino extends Animal {
  hasHorn: true
}
class Elephant extends Animal {
  hasTrunk: true
}
class Snake extends Animal {
  hasLegs: false
}
// ---cut---
let zoo: Animal[] = [new Rhino(), new Elephant(), new Snake()]
//    ^?
```

如果未找到最佳通用类型，推断的结果将是联合数组类型 `(Rhino | Elephant | Snake)[]`。

## 上下文类型（Contextual Typing）

在 TypeScript 中，类型推断在某些情况下也会“反向”进行。
这被称为“上下文类型”（contextual typing）。当一个表达式的类型由其所在的位置隐式确定时，就会发生上下文类型推断。例如：

```ts twoslash
// @errors: 2339
window.onmousedown = function (mouseEvent) {
  console.log(mouseEvent.button)
  console.log(mouseEvent.kangaroo)
}
```

在这里，TypeScript 类型检查器使用 `Window.onmousedown` 函数的类型来推断赋值操作右侧函数表达式的类型。
在这样做时，它能够推断出 `mouseEvent` 参数的[类型](https://developer.mozilla.org/docs/Web/API/MouseEvent)，该类型确实包含 `button` 属性，但不包含 `kangaroo` 属性。

这之所以有效，是因为 `window` 的类型中已经声明了 `onmousedown`：

```ts
// Declares there is a global variable called 'window'
declare var window: Window & typeof globalThis

// Which is declared as (simplified):
interface Window extends GlobalEventHandlers {
  // ...
}

// Which defines a lot of known handler events
interface GlobalEventHandlers {
  onmousedown: ((this: GlobalEventHandlers, ev: MouseEvent) => any) | null
  // ...
}
```

TypeScript 足够智能，在其他上下文中也能推断类型：

```ts twoslash
// @errors: 2339
window.onscroll = function (uiEvent) {
  console.log(uiEvent.button)
}
```

基于上述函数被赋值给 `Window.onscroll` 的事实，TypeScript 得知 `uiEvent` 是一个 [UIEvent](https://developer.mozilla.org/docs/Web/API/UIEvent)，而不是像前一个示例中的 [MouseEvent](https://developer.mozilla.org/docs/Web/API/MouseEvent)。`UIEvent` 对象不包含 `button` 属性，因此 TypeScript 会报错。

如果该函数不在受上下文类型影响的位置，该函数的参数将隐式具有 `any` 类型，并且不会报错（除非你启用了 [`noImplicitAny`](/tsconfig#noImplicitAny) 选项）：

```ts twoslash
// @noImplicitAny: false
const handler = function (uiEvent) {
  console.log(uiEvent.button) // <- OK
}
```

我们也可以显式为函数的参数提供类型信息，以覆盖任何上下文类型：

```ts twoslash
window.onscroll = function (uiEvent: any) {
  console.log(uiEvent.button) // <- Now, no error is given
}
```

然而，这段代码将输出 `undefined`，因为 `uiEvent` 上并没有名为 `button` 的属性。

上下文类型适用于多种情况。
常见的场景包括：函数调用的参数、赋值操作的右侧、类型断言、对象和数组字面量的成员以及 return 语句。
上下文类型还可以作为最佳通用类型中的候选类型。例如：

```ts twoslash
// @strict: false
class Animal {}
class Rhino extends Animal {
  hasHorn: true
}
class Elephant extends Animal {
  hasTrunk: true
}
class Snake extends Animal {
  hasLegs: false
}
// ---cut---
function createZoo(): Animal[] {
  return [new Rhino(), new Elephant(), new Snake()]
}
```

在这个示例中，最佳通用类型拥有由四个候选类型组成的集合：`Animal`、`Rhino`、`Elephant` 和 `Snake`。
在这些候选项中，最佳通用类型算法可以选择 `Animal`。
