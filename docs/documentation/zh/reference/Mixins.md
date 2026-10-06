---
title: 混入（Mixins）
layout: docs
permalink: /zh/docs/handbook/mixins.html
oneline: 在 TypeScript 中使用混入模式
translatable: true
---

除了传统的面向对象层次结构之外，另一种通过可复用组件构建类的流行方式是通过组合更简单的局部类来进行构建。
你可能熟悉 Scala 等语言中的混入（mixins）或特质（traits）概念，这一模式在 JavaScript 社区中也颇受欢迎。

## 混入是如何工作的？

该模式依赖于结合泛型与类继承来扩展基类。
TypeScript 对混入的最佳支持是通过类表达式模式实现的。
你可以[在此处](https://justinfagnani.com/2015/12/21/real-mixins-with-javascript-classes/)阅读有关该模式在 JavaScript 中如何工作的更多信息。

首先，我们需要一个将被应用混入的基础类：

```ts twoslash
class Sprite {
  name = ''
  x = 0
  y = 0

  constructor(name: string) {
    this.name = name
  }
}
```

接下来，你需要一个类型以及一个返回继承自基类的类表达式的工厂函数。

```ts twoslash
// To get started, we need a type which we'll use to extend
// other classes from. The main responsibility is to declare
// that the type being passed in is a class.

type Constructor = new (...args: any[]) => {}

// This mixin adds a scale property, with getters and setters
// for changing it with an encapsulated private property:

function Scale<TBase extends Constructor>(Base: TBase) {
  return class Scaling extends Base {
    // Mixins may not declare private/protected properties
    // however, you can use ES2020 private fields
    _scale = 1

    setScale(scale: number) {
      this._scale = scale
    }

    get scale(): number {
      return this._scale
    }
  }
}
```

完成这些准备工作后，你就可以创建一个表示应用了混入的基类：

```ts twoslash
class Sprite {
  name = ''
  x = 0
  y = 0

  constructor(name: string) {
    this.name = name
  }
}
type Constructor = new (...args: any[]) => {}
function Scale<TBase extends Constructor>(Base: TBase) {
  return class Scaling extends Base {
    // Mixins may not declare private/protected properties
    // however, you can use ES2020 private fields
    _scale = 1

    setScale(scale: number) {
      this._scale = scale
    }

    get scale(): number {
      return this._scale
    }
  }
}
// ---cut---
// Compose a new class from the Sprite class,
// with the Mixin Scale applier:
const EightBitSprite = Scale(Sprite)

const flappySprite = new EightBitSprite('Bird')
flappySprite.setScale(0.8)
console.log(flappySprite.scale)
```

## 受约束的混入（Constrained Mixins）

在上面的形式中，混入并不了解被混入类的任何底层信息，这可能会使你难以实现想要的设计。

为了对此建模，我们修改原始的构造函数类型以接受泛型参数。

```ts twoslash
// This was our previous constructor:
type Constructor = new (...args: any[]) => {}
// Now we use a generic version which can apply a constraint on
// the class which this mixin is applied to
type GConstructor<T = {}> = new (...args: any[]) => T
```

这允许创建仅适用于受约束基类的类：

```ts twoslash
type GConstructor<T = {}> = new (...args: any[]) => T
class Sprite {
  name = ''
  x = 0
  y = 0

  constructor(name: string) {
    this.name = name
  }
}
// ---cut---
type Positionable = GConstructor<{ setPos: (x: number, y: number) => void }>
type Spritable = GConstructor<Sprite>
type Loggable = GConstructor<{ print: () => void }>
```

然后，你就可以创建仅在拥有特定基类时才能工作的混入：

```ts twoslash
type GConstructor<T = {}> = new (...args: any[]) => T
class Sprite {
  name = ''
  x = 0
  y = 0

  constructor(name: string) {
    this.name = name
  }
}
type Positionable = GConstructor<{ setPos: (x: number, y: number) => void }>
type Spritable = GConstructor<Sprite>
type Loggable = GConstructor<{ print: () => void }>
// ---cut---

function Jumpable<TBase extends Positionable>(Base: TBase) {
  return class Jumpable extends Base {
    jump() {
      // This mixin will only work if it is passed a base
      // class which has setPos defined because of the
      // Positionable constraint.
      this.setPos(0, 20)
    }
  }
}
```

## 替代模式（Alternative Pattern）

本文档的早期版本推荐过一种编写混入的方式，即分别独立创建运行时和类型层次结构，最后再将它们合并：

```ts twoslash
// @strict: false
// Each mixin is a traditional ES class
class Jumpable {
  jump() {}
}

class Duckable {
  duck() {}
}

// Including the base
class Sprite {
  x = 0
  y = 0
}

// Then you create an interface which merges
// the expected mixins with the same name as your base
interface Sprite extends Jumpable, Duckable {}
// Apply the mixins into the base class via
// the JS at runtime
applyMixins(Sprite, [Jumpable, Duckable])

let player = new Sprite()
player.jump()
console.log(player.x, player.y)

// This can live anywhere in your codebase:
function applyMixins(derivedCtor: any, constructors: any[]) {
  constructors.forEach((baseCtor) => {
    Object.getOwnPropertyNames(baseCtor.prototype).forEach((name) => {
      Object.defineProperty(
        derivedCtor.prototype,
        name,
        Object.getOwnPropertyDescriptor(baseCtor.prototype, name) ||
          Object.create(null),
      )
    })
  })
}
```

这种模式对编译器的依赖较少，更多地依赖于你的代码库来确保运行时与类型系统保持正确同步。

## 限制

TypeScript 编译器通过控制流分析原生支持混入模式。
在某些情况下，你可能会触碰到原生支持的边界。

#### 装饰器与混入 [`#4881`](https://github.com/microsoft/TypeScript/issues/4881)

你不能通过控制流分析使用装饰器来提供混入：

```ts twoslash
// @experimentalDecorators
// @errors: 2339
// A decorator function which replicates the mixin pattern:
const Pausable = (target: typeof Player) => {
  return class Pausable extends target {
    shouldFreeze = false
  }
}

@Pausable
class Player {
  x = 0
  y = 0
}

// The Player class does not have the decorator's type merged:
const player = new Player()
player.shouldFreeze

// The runtime aspect could be manually replicated via
// type composition or interface merging.
type FreezablePlayer = Player & { shouldFreeze: boolean }

const playerTwo = new Player() as unknown as FreezablePlayer
playerTwo.shouldFreeze
```

#### 静态属性混入 [`#17829`](https://github.com/microsoft/TypeScript/issues/17829)

这更像是一个陷阱而非限制。
类表达式模式创建的是单例，因此无法在类型系统级别进行映射以支持不同的变量类型。

你可以通过使用函数返回根据泛型而不同的类来解决这个问题：

```ts twoslash
function base<T>() {
  class Base {
    static prop: T
  }
  return Base
}

function derived<T>() {
  class Derived extends base<T>() {
    static anotherProp: T
  }
  return Derived
}

class Spec extends derived<string>() {}

Spec.prop // string
Spec.anotherProp // string
```
