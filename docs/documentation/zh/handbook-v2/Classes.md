---
title: 类
layout: docs
permalink: /zh/docs/handbook/2/classes.html
oneline: 'TypeScript 中类的工作原理'
---

<blockquote class='bg-reading'>
  <p>背景阅读：<br /><a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes'>类 (MDN)</a></p>
</blockquote>

TypeScript 完全支持 ES2015 中引入的 `class` 关键字。

与其他 JavaScript 语言特性一样，TypeScript 添加了类型注解和其他语法，使你能够表达类与其他类型之间的关系。

## 类成员

这是最基础的类——一个空类：

```ts twoslash
class Point {}
```

这个类目前还没有什么实际用处，现在让我们开始添加一些成员。

### 字段

字段声明会在类上创建一个公共且可写的属性：

```ts twoslash
// @strictPropertyInitialization: false
class Point {
  x: number
  y: number
}

const pt = new Point()
pt.x = 0
pt.y = 0
```

与其他位置一样，类型注解是可选的；如果未显式指定，则会隐式推断为 `any`。

字段还可以拥有*初始值设定项*（initializers）；它们会在类实例化时自动运行：

```ts twoslash
class Point {
  x = 0
  y = 0
}

const pt = new Point()
// Prints 0, 0
console.log(`${pt.x}, ${pt.y}`)
```

就像 `const`、`let` 和 `var` 一样，类属性的初始值设定项也会用于推断其类型：

```ts twoslash
// @errors: 2322
class Point {
  x = 0
  y = 0
}
// ---cut---
const pt = new Point()
pt.x = '0'
```

#### `--strictPropertyInitialization`

[`strictPropertyInitialization`](/tsconfig#strictPropertyInitialization) 设置用于控制类字段是否必须在构造函数中进行初始化。

```ts twoslash
// @errors: 2564
class BadGreeter {
  name: string
}
```

```ts twoslash
class GoodGreeter {
  name: string

  constructor() {
    this.name = 'hello'
  }
}
```

请注意，字段必须*在构造函数本身内部*进行初始化。
TypeScript 不会分析你在构造函数中调用的方法来检测初始化行为，因为派生类可能会重写这些方法，从而导致成员未能成功初始化。

如果你打算通过构造函数之外的其他方式明确初始化某个字段（例如，可能有外部库为你填充类的某些部分），你可以使用*明确赋值断言运算符*（definite assignment assertion operator）`!`：

```ts twoslash
class OKGreeter {
  // Not initialized, but no error
  name!: string
}
```

### `readonly`

字段可以添加 `readonly` 修饰符前缀。
这可以防止在构造函数之外对该字段进行赋值。

```ts twoslash
// @errors: 2540 2540
class Greeter {
  readonly name: string = 'world'

  constructor(otherName?: string) {
    if (otherName !== undefined) {
      this.name = otherName
    }
  }

  err() {
    this.name = 'not ok'
  }
}
const g = new Greeter()
g.name = 'also not ok'
```

### 构造函数

<blockquote class='bg-reading'>
   <p>背景阅读：<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/constructor'>构造函数 (MDN)</a><br/>
   </p>
</blockquote>

类构造函数与普通函数非常相似。
你可以为参数添加类型注解、默认值以及重载：

```ts twoslash
class Point {
  x: number
  y: number

  // Normal signature with defaults
  constructor(x = 0, y = 0) {
    this.x = x
    this.y = y
  }
}
```

```ts twoslash
class Point {
  x: number = 0
  y: number = 0

  // Constructor overloads
  constructor(x: number, y: number)
  constructor(xy: string)
  constructor(x: string | number, y: number = 0) {
    // Code logic here
  }
}
```

类构造函数签名与普通函数签名之间仅存在少数几点区别：

- 构造函数不能拥有类型参数——类型参数属于外部的类声明，我们稍后会介绍这一点
- 构造函数不能拥有返回类型注解——其返回的始终是类实例类型

#### 调用 Super

正如在 JavaScript 中一样，如果你有基类，则必须在构造函数体中使用任何 `this.` 成员之前调用 `super();`：

```ts twoslash
// @errors: 17009
class Base {
  k = 4
}

class Derived extends Base {
  constructor() {
    // Prints a wrong value in ES5; throws exception in ES6
    console.log(this.k)
    super()
  }
}
```

忘记调用 `super` 是在 JavaScript 中容易犯的错误，但 TypeScript 会在需要调用时提示你。

### 方法

<blockquote class='bg-reading'>
   <p>背景阅读：<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Method_definitions'>方法定义</a><br/>
   </p>
</blockquote>

类中的函数属性被称为*方法*。
方法可以使用与普通函数和构造函数完全相同的类型注解：

```ts twoslash
class Point {
  x = 10
  y = 10

  scale(n: number): void {
    this.x *= n
    this.y *= n
  }
}
```

除了标准的类型注解之外，TypeScript 没有为方法添加其他任何新特性。

请注意，在方法体内部，依然必须通过 `this.` 来访问字段和其他方法。
在方法体中未加限定的名称将始终引用外层作用域中的内容：

```ts twoslash
// @errors: 2322
let x: number = 0

class C {
  x: string = 'hello'

  m() {
    // This is trying to modify 'x' from line 1, not the class property
    x = 'world'
  }
}
```

### Getter 与 Setter

类还可以拥有*访问器*（accessors）：

```ts twoslash
class C {
  _length = 0
  get length() {
    return this._length
  }
  set length(value) {
    this._length = value
  }
}
```

> 请注意，在 JavaScript 中，没有任何额外逻辑、仅仅包装某个字段的 get/set 组合几乎没有任何用处。
> 如果你不需要在 get/set 操作期间添加额外逻辑，直接暴露 public 字段完全没有问题。

TypeScript 对访问器有一些特殊的推断规则：

- 如果存在 `get` 但不存在 `set`，则该属性会自动变为 `readonly`
- 如果未指定 setter 的参数类型，则会根据 getter 的返回类型进行推断

自 [TypeScript 4.3](https://devblogs.microsoft.com/typescript/announcing-typescript-4-3/) 起，访问器的读取（get）和设置（set）可以拥有不同的类型。

```ts twoslash
class Thing {
  _size = 0

  get size(): number {
    return this._size
  }

  set size(value: string | number | boolean) {
    let num = Number(value)

    // Don't allow NaN, Infinity, etc

    if (!Number.isFinite(num)) {
      this._size = 0
      return
    }

    this._size = num
  }
}
```

### 索引签名

类可以声明索引签名；它们的工作方式与[其他对象类型的索引签名](/docs/handbook/2/objects.html#index-signatures)相同：

```ts twoslash
class MyClass {
  [s: string]: boolean | ((s: string) => boolean)

  check(s: string) {
    return this[s] as boolean
  }
}
```

由于索引签名类型还需要涵盖方法的类型，因此想要合理地使用这类类型并不容易。
通常更好的做法是将带有索引的数据存储在其他地方，而不是直接存储在类实例本身上。

## 类继承

与其他具备面向对象特性的语言一样，JavaScript 中的类可以继承自基类。

### `implements` 子句

你可以使用 `implements` 子句来检查某个类是否满足了特定的 `interface`。
如果类未能正确实现该接口，则会发出错误提示：

```ts twoslash
// @errors: 2420
interface Pingable {
  ping(): void
}

class Sonar implements Pingable {
  ping() {
    console.log('ping!')
  }
}

class Ball implements Pingable {
  pong() {
    console.log('pong!')
  }
}
```

类还可以实现多个接口，例如 `class C implements A, B {`。

#### 注意事项

必须明确的是，`implements` 子句仅用于检查该类是否可以被视为该接口类型。
它*根本不会*改变类本身或其方法的类型。
一个常见的错误来源就是误以为 `implements` 子句会改变类的类型——它并不会！

```ts twoslash
// @errors: 7006
interface Checkable {
  check(name: string): boolean
}

class NameChecker implements Checkable {
  check(s) {
    // Notice no error here
    return s.toLowerCase() === 'ok'
    //         ^?
  }
}
```

在这个例子中，我们或许期望 `s` 的类型会受到 `check` 的 `name: string` 参数的影响。
但事实并非如此——`implements` 子句不会改变类体被检查的方式，也不会影响其类型推断。

同样地，实现包含可选属性的接口并不会自动创建该属性：

```ts twoslash
// @errors: 2339
interface A {
  x: number
  y?: number
}
class C implements A {
  x = 0
}
const c = new C()
c.y = 10
```

### `extends` 子句

<blockquote class='bg-reading'>
   <p>背景阅读：<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/extends'>extends 关键字 (MDN)</a><br/>
   </p>
</blockquote>

类可以使用 `extend` 继承自基类。
派生类拥有其基类的所有属性和方法，同时还可以定义额外的成员。

```ts twoslash
class Animal {
  move() {
    console.log('Moving along!')
  }
}

class Dog extends Animal {
  woof(times: number) {
    for (let i = 0; i < times; i++) {
      console.log('woof!')
    }
  }
}

const d = new Dog()
// Base class method
d.move()
// Derived class method
d.woof(3)
```

#### 重写方法

<blockquote class='bg-reading'>
   <p>背景阅读：<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/super'>super 关键字 (MDN)</a><br/>
   </p>
</blockquote>

派生类还可以重写基类的字段或属性。
你可以使用 `super.` 语法来访问基类的方法。
请注意，由于 JavaScript 的类本质上是简单的查找对象，因此不存在“super 字段”（super field）的概念。

TypeScript 强制要求派生类必须始终是其基类的子类型。

例如，下面是一种合法的重写方法的方式：

```ts twoslash
class Base {
  greet() {
    console.log('Hello, world!')
  }
}

class Derived extends Base {
  greet(name?: string) {
    if (name === undefined) {
      super.greet()
    } else {
      console.log(`Hello, ${name.toUpperCase()}`)
    }
  }
}

const d = new Derived()
d.greet()
d.greet('reader')
```

派生类遵循其基类的契约至关重要。
请记住，通过基类引用来指向派生类实例是非常普遍（且始终合法）的做法：

```ts twoslash
class Base {
  greet() {
    console.log('Hello, world!')
  }
}
class Derived extends Base {}
const d = new Derived()
// ---cut---
// Alias the derived instance through a base class reference
const b: Base = d
// No problem
b.greet()
```

如果 `Derived` 没有遵循 `Base` 的契约会怎样？

```ts twoslash
// @errors: 2416
class Base {
  greet() {
    console.log('Hello, world!')
  }
}

class Derived extends Base {
  // Make this parameter required
  greet(name: string) {
    console.log(`Hello, ${name.toUpperCase()}`)
  }
}
```

如果我们忽略错误强行编译这段代码，该示例随后就会发生运行时崩溃：

```ts twoslash
declare class Base {
  greet(): void
}
declare class Derived extends Base {}
// ---cut---
const b: Base = new Derived()
// Crashes because "name" will be undefined
b.greet()
```

#### 仅类型字段声明

当 `target >= ES2022` 或 [`useDefineForClassFields`](/tsconfig#useDefineForClassFields) 为 `true` 时，类字段会在父类构造函数执行完毕后进行初始化，这会覆盖父类设置的任何值。当你只想为继承的字段重新声明一个更精确的类型时，这就会产生问题。为了处理这些情况，你可以使用 `declare` 语法来向 TypeScript 表明该字段声明不应产生任何运行时效果。

```ts twoslash
interface Animal {
  dateOfBirth: any
}

interface Dog extends Animal {
  breed: any
}

class AnimalHouse {
  resident: Animal
  constructor(animal: Animal) {
    this.resident = animal
  }
}

class DogHouse extends AnimalHouse {
  // Does not emit JavaScript code,
  // only ensures the types are correct
  declare resident: Dog
  constructor(dog: Dog) {
    super(dog)
  }
}
```

#### 初始化顺序

在某些情况下，JavaScript 类的初始化顺序可能会让人感到意外。
让我们看看下面的代码：

```ts twoslash
class Base {
  name = 'base'
  constructor() {
    console.log('My name is ' + this.name)
  }
}

class Derived extends Base {
  name = 'derived'
}

// Prints "base", not "derived"
const d = new Derived()
```

这里发生了什么？

根据 JavaScript 的规范定义，类的初始化顺序如下：

- 基类字段完成初始化
- 基类构造函数运行
- 派生类字段完成初始化
- 派生类构造函数运行

这意味着基类构造函数在其执行过程中读取到的是它自己的 `name` 值，因为此时派生类字段的初始化尚未执行。

#### 继承内置类型

> 注意：如果你不打算继承 `Array`、`Error`、`Map` 等内置类型，或者你的编译目标已明确设置为 `ES6`/`ES2015` 或更高版本，你可以跳过本节。

在 ES2015 中，返回对象的构造函数会隐式地将 `this` 的值替换为 `super(...)` 的任何调用者。
生成的构造函数代码必须捕获 `super(...)` 的任何潜在返回值并将其替换为 `this`。

因此，子类化 `Error`、`Array` 等类型可能不再符合预期。
这是因为 `Error`、`Array` 等的构造函数使用 ECMAScript 6 的 `new.target` 来调整原型链；
然而，在 ECMAScript 5 中调用构造函数时，无法确保 `new.target` 的值。
其他降级编译器（downlevel compilers）默认通常也有相同的限制。

对于如下的子类：

```ts twoslash
class MsgError extends Error {
  constructor(m: string) {
    super(m)
  }
  sayHello() {
    return 'hello ' + this.message
  }
}
```

你可能会发现：

- 构造这些子类返回的对象上的方法可能是 `undefined`，因此调用 `sayHello` 会报错。
- 子类与其对应实例之间的 `instanceof` 关系会失效，因此 `(new MsgError()) instanceof MsgError` 会返回 `false`。

作为建议，你可以在调用任何 `super(...)` 后立即手动调整原型。

```ts twoslash
class MsgError extends Error {
  constructor(m: string) {
    super(m)

    // Set the prototype explicitly.
    Object.setPrototypeOf(this, MsgError.prototype)
  }

  sayHello() {
    return 'hello ' + this.message
  }
}
```

然而，`MsgError` 的任何子类也必须手动设置原型。
对于不支持 [`Object.setPrototypeOf`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf) 的运行时环境，你也可以改用 [`__proto__`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/proto)。

遗憾的是，[这些变通方案在 Internet Explorer 10 及更早版本中不起作用](<https://msdn.microsoft.com/en-us/library/s4esdbwz(v=vs.94).aspx>)。
开发者可以手动将原型中的方法复制到实例本身（即把 `MsgError.prototype` 复制到 `this`），但原型链本身无法被修复。

## 成员可见性

你可以使用 TypeScript 来控制某些方法或属性对类外部代码是否可见。

### `public`

类成员的默认可见性是 `public`。
`public` 成员可以在任何地方被访问：

```ts twoslash
class Greeter {
  public greet() {
    console.log('hi!')
  }
}
const g = new Greeter()
g.greet()
```

由于 `public` 已经是默认的可见性修饰符，因此你永远*不需要*在类成员上显式编写它，但出于代码风格或可读性考虑，你可以选择显式写出。

### `protected`

`protected` 成员仅对其所声明类的子类可见。

```ts twoslash
// @errors: 2445
class Greeter {
  public greet() {
    console.log('Hello, ' + this.getName())
  }
  protected getName() {
    return 'hi'
  }
}

class SpecialGreeter extends Greeter {
  public howdy() {
    // OK to access protected member here
    console.log('Howdy, ' + this.getName())
    //                          ^^^^^^^^^^^^^^
  }
}
const g = new SpecialGreeter()
g.greet() // OK
g.getName()
```

#### 暴露 `protected` 成员

派生类需要遵循其基类的契约，但可以选择暴露具有更多能力的基类子类型。
这包括将 `protected` 成员设为 `public`：

```ts twoslash
class Base {
  protected m = 10
}
class Derived extends Base {
  // No modifier, so default is 'public'
  m = 15
}
const d = new Derived()
console.log(d.m) // OK
```

请注意，`Derived` 本来就能够自由读取和写入 `m`，因此这并不会实质性地改变这种情况的“安全性”。
这里需要注意的关键点是：在派生类中，如果这种暴露并非有意为之，我们需要注意重复添加 `protected` 修饰符。

#### 跨层级的 `protected` 访问

在类的继承层级中，TypeScript 不允许访问同级（兄弟）类的 `protected` 成员：

```ts twoslash
// @errors: 2446
class Base {
  protected x: number = 1
}
class Derived1 extends Base {
  protected x: number = 5
}
class Derived2 extends Base {
  f1(other: Derived2) {
    other.x = 10
  }
  f2(other: Derived1) {
    other.x = 10
  }
}
```

这是因为在 `Derived2` 中访问 `x` 应当仅在其子类中合法，而 `Derived1` 并不是它的子类。
此外，如果通过 `Derived1` 引用访问 `x` 是非法的（事实显然如此！），那么通过基类引用来访问它也不应该破例放宽这一限制。

另请参阅 [Why Can’t I Access A Protected Member From A Derived Class?](https://blogs.msdn.microsoft.com/ericlippert/2005/11/09/why-cant-i-access-a-protected-member-from-a-derived-class/)，其中详细阐述了 C# 在同一问题上的设计思路。

### `private`

`private` 与 `protected` 类似，但即使是在子类中也不允许访问该成员：

```ts twoslash
// @errors: 2341
class Base {
  private x = 0
}
const b = new Base()
// Can't access from outside the class
console.log(b.x)
```

```ts twoslash
// @errors: 2341
class Base {
  private x = 0
}
// ---cut---
class Derived extends Base {
  showX() {
    // Can't access in subclasses
    console.log(this.x)
  }
}
```

由于 `private` 成员对派生类不可见，因此派生类无法提高它们的可见性：

```ts twoslash
// @errors: 2415
class Base {
  private x = 0
}
class Derived extends Base {
  x = 1
}
```

#### 跨实例的 `private` 访问

不同的面向对象语言在同一类的不同实例是否可以相互访问各自的 `private` 成员这一问题上存在分歧。
虽然 Java、C#、C++、Swift 和 PHP 等语言允许这样做，但 Ruby 不允许。

TypeScript 允许跨实例的 `private` 访问：

```ts twoslash
class A {
  private x = 10

  public sameAs(other: A) {
    // No error
    return other.x === this.x
  }
}
```

#### 注意事项

与 TypeScript 类型系统的其他方面一样，`private` 和 `protected` [仅在类型检查期间强制执行](https://www.typescriptlang.org/play?removeComments=true&target=99&ts=4.3.4#code/PTAEGMBsEMGddAEQPYHNQBMCmVoCcsEAHPASwDdoAXLUAM1K0gwQFdZSA7dAKWkoDK4MkSoByBAGJQJLAwAeAWABQIUH0HDSoiTLKUaoUggAW+DHorUsAOlABJcQlhUy4KpACeoLJzrI8cCwMGxU1ABVPIiwhESpMZEJQTmR4lxFQaQxWMm4IZABbIlIYKlJkTlDlXHgkNFAAbxVQTIAjfABrAEEC5FZOeIBeUAAGAG5mmSw8WAroSFIqb2GAIjMiIk8VieVJ8Ar01ncAgAoASkaAXxVr3dUwGoQAYWpMHBgCYn1rekZmNg4eUi0Vi2icoBWJCsNBWoA6WE8AHcAiEwmBgTEtDovtDaMZQLM6PEoQZbA5wSk0q5SO4vD4-AEghZoJwLGYEIRwNBoqAzFRwCZCFUIlFMXECdSiAhId8YZgclx0PsiiVqOVOAAaUAFLAsxWgKiC35MFigfC0FKgSAVVDTSyk+W5dB4fplHVVR6gF7xJrKFotEk-HXIRE9PoDUDDcaTAPTWaceaLZYQlmoPBbHYx-KcQ7HPDnK43FQqfY5+IMDDISPJLCIuqoc47UsuUCofAME3Vzi1r3URvF5QV5A2STtPDdXqunZDgDaYlHnTDrrEAF0dm28B3mDZg6HJwN1+2-hg57ulwNV2NQGoZbjYfNrYiENBwEFaojFiZQK08C-4fFKTVCozWfTgfFgLkeT5AUqiAA)。

这意味着 JavaScript 的运行时语法结构（如 `in` 或简单的属性查找）仍然可以访问 `private` 或 `protected` 成员：

```ts twoslash
class MySafe {
  private secretKey = 12345
}
```

```js
// In a JavaScript file...
const s = new MySafe()
// Will print 12345
console.log(s.secretKey)
```

在类型检查期间，`private` 还允许使用中括号语法进行访问。这使得被声明为 `private` 的字段在进行单元测试等操作时可能更容易被访问，但缺点是这些字段只是*软私有*（soft private），并没有严格强制私密性。

```ts twoslash
// @errors: 2341
class MySafe {
  private secretKey = 12345
}

const s = new MySafe()

// Not allowed during type checking
console.log(s.secretKey)

// OK
console.log(s['secretKey'])
```

与 TypeScript 的 `private` 不同，JavaScript 的[私有字段](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields)（`#`）在编译后依然保持私有，并且不提供前面提到的诸如中括号语法访问之类的逃生通道，从而实现了*硬私有*（hard private）。

```ts twoslash
class Dog {
  #barkAmount = 0
  personality = 'happy'

  constructor() {}
}
```

```ts twoslash
// @target: esnext
// @showEmit
class Dog {
  #barkAmount = 0
  personality = 'happy'

  constructor() {}
}
```

当编译目标为 ES2021 或更低版本时，TypeScript 会使用 WeakMap 来替代 `#`。

```ts twoslash
// @target: es2015
// @showEmit
class Dog {
  #barkAmount = 0
  personality = 'happy'

  constructor() {}
}
```

如果你需要保护类中的值免遭恶意攻击者的破坏，应当使用提供硬运行时私有性的机制，例如闭包、WeakMap 或私有字段。请注意，在运行时增加这些私密性检查可能会影响性能。

## 静态成员

<blockquote class='bg-reading'>
   <p>背景阅读：<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/static'>静态成员 (MDN)</a><br/>
   </p>
</blockquote>

类可以拥有 `static` 成员。
这些成员不与类的特定实例相关联。
它们可以通过类构造函数对象本身来访问：

```ts twoslash
class MyClass {
  static x = 0
  static printX() {
    console.log(MyClass.x)
  }
}
console.log(MyClass.x)
MyClass.printX()
```

静态成员同样可以使用 `public`、`protected` 和 `private` 这些可见性修饰符：

```ts twoslash
// @errors: 2341
class MyClass {
  private static x = 0
}
console.log(MyClass.x)
```

静态成员也会被继承：

```ts twoslash
class Base {
  static getGreeting() {
    return 'Hello world'
  }
}
class Derived extends Base {
  myGreeting = Derived.getGreeting()
}
```

### 特殊的静态名称

覆盖 `Function` 原型上的属性通常是不安全或不可能的。
因为类本身就是可以使用 `new` 调用的函数，所以某些 `static` 名称是不能使用的。
诸如 `name`、`length` 和 `call` 等函数属性不能被定义为 `static` 成员：

```ts twoslash
// @errors: 2699
class S {
  static name = 'S!'
}
```

### 为什么没有静态类？

TypeScript（以及 JavaScript）中并没有类似 C# 那样被称为 `static class`（静态类）的语言结构。

这些结构*仅仅*是因为那些语言强制所有数据和函数都必须存在于类内部才存在的；而由于 TypeScript 中并不存在这种限制，因此完全没有必要引入它们。
只有一个实例的类，在 JavaScript/TypeScript 中通常直接用普通*对象*表示即可。

例如，在 TypeScript 中我们不需要“静态类”语法，因为普通对象（甚至顶层函数）就能同样完美地完成这项工作：

```ts twoslash
// Unnecessary "static" class
class MyStaticClass {
  static doSomething() {}
}

// Preferred (alternative 1)
function doSomething() {}

// Preferred (alternative 2)
const MyHelperObject = {
  dosomething() {},
}
```

## 类中的 `static` 块

静态块（static blocks）允许你编写具有自身独立作用域的语句序列，并且这些语句可以访问所在类中的私有字段。这意味着我们在编写初始化代码时，不仅能具备编写语句的全部能力、不会造成变量泄漏，还能完全访问类的内部细节。

```ts twoslash
declare function loadLastInstances(): any[]
// ---cut---
class Foo {
  static #count = 0

  get count() {
    return Foo.#count
  }

  static {
    try {
      const lastInstances = loadLastInstances()
      Foo.#count += lastInstances.length
    } catch {}
  }
}
```

## 泛型类

与接口非常相似，类也可以是泛型的。
当使用 `new` 实例化泛型类时，其类型参数的推断方式与函数调用中的推断方式相同：

```ts twoslash
class Box<Type> {
  contents: Type
  constructor(value: Type) {
    this.contents = value
  }
}

const b = new Box('hello!')
//    ^?
```

类可以像接口一样使用泛型约束和默认值。

### 静态成员中的类型参数

以下代码是非法的，原因可能并不显而易见：

```ts twoslash
// @errors: 2302
class Box<Type> {
  static defaultValue: Type
}
```

请记住，类型在运行时总是被完全擦除的！
在运行时，实际上只存在*一个* `Box.defaultValue` 属性槽。
这意味着，设置 `Box<string>.defaultValue`（如果真能这么做的话）*也会*改变 `Box<number>.defaultValue`——这显然不是期望的行为。
泛型类的 `static` 成员永远不能引用该类的类型参数。

## 类在运行时的 `this`

<blockquote class='bg-reading'>
   <p>背景阅读：<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this'>this 关键字 (MDN)</a><br/>
   </p>
</blockquote>

牢记这一点非常重要：TypeScript 不会改变 JavaScript 的运行时行为，而 JavaScript 又因具备某些独特的运行时行为而闻名。

JavaScript 处理 `this` 的方式确实不同寻常：

```ts twoslash
class MyClass {
  name = 'MyClass'
  getName() {
    return this.name
  }
}
const c = new MyClass()
const obj = {
  name: 'obj',
  getName: c.getName,
}

// Prints "obj", not "MyClass"
console.log(obj.getName())
```

长话短说，默认情况下，函数内部 `this` 的值取决于*该函数是如何被调用的*。
在这个例子中，因为该函数是通过 `obj` 引用调用的，所以它的 `this` 值为 `obj` 而不是类实例。

这通常不是你所期望的结果！
TypeScript 提供了一些方法来缓解或防止此类错误。

### 箭头函数

<blockquote class='bg-reading'>
   <p>背景阅读：<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions'>箭头函数 (MDN)</a><br/>
   </p>
</blockquote>

如果你有一个函数，它经常会在丢失 `this` 上下文的方式下被调用，那么使用箭头函数属性而不是方法定义是有意义的：

```ts twoslash
class MyClass {
  name = 'MyClass'
  getName = () => {
    return this.name
  }
}
const c = new MyClass()
const g = c.getName
// Prints "MyClass" instead of crashing
console.log(g())
```

这存在一些权衡：

- 即使对于未经 TypeScript 检查的代码，`this` 的值在运行时也保证是正确的
- 这会消耗更多内存，因为每个类实例都会为以此方式定义的每个函数保留一份独立的副本
- 你不能在派生类中使用 `super.getName`，因为在原型链中没有相应的条目来获取基类方法

### `this` 参数

在方法或函数定义中，名为 `this` 的前置参数在 TypeScript 中具有特殊含义。
这些参数在编译过程中会被擦除：

```ts twoslash
type SomeType = any
// ---cut---
// TypeScript input with 'this' parameter
function fn(this: SomeType, x: number) {
  /* ... */
}
```

```js
// JavaScript output
function fn(x) {
  /* ... */
}
```

TypeScript 会检查带有 `this` 参数的函数调用是否在正确的上下文中进行。
我们可以不用箭头函数，而是向方法定义中添加一个 `this` 参数，从而静态地强制要求该方法在正确的上下文中被调用：

```ts twoslash
// @errors: 2684
class MyClass {
  name = 'MyClass'
  getName(this: MyClass) {
    return this.name
  }
}
const c = new MyClass()
// OK
c.getName()

// Error, would crash
const g = c.getName
console.log(g())
```

这种做法与箭头函数方案的权衡取舍正好相反：

- JavaScript 调用方可能仍会在不知不觉中错误地使用该类方法
- 每个类定义只会分配一个函数，而不是每个类实例分配一个
- 基类方法定义仍然可以通过 `super` 调用。

## `this` 类型

在类中，名为 `this` 的特殊类型*动态地*引用当前类的类型。
让我们看看这有什么用：

<!-- prettier-ignore -->
```ts twoslash
class Box {
  contents: string = "";
  set(value: string) {
//  ^?
    this.contents = value;
    return this;
  }
}
```

在这里，TypeScript 推断 `set` 的返回类型为 `this`，而不是 `Box`。
现在让我们创建 `Box` 的一个子类：

```ts twoslash
class Box {
  contents: string = ''
  set(value: string) {
    this.contents = value
    return this
  }
}
// ---cut---
class ClearableBox extends Box {
  clear() {
    this.contents = ''
  }
}

const a = new ClearableBox()
const b = a.set('hello')
//    ^?
```

你还可以在参数类型注解中使用 `this`：

```ts twoslash
class Box {
  content: string = ''
  sameAs(other: this) {
    return other.content === this.content
  }
}
```

这与编写 `other: Box` 不同——如果你有一个派生类，它的 `sameAs` 方法现在将只接受该派生类自身的其他实例：

```ts twoslash
// @errors: 2345
class Box {
  content: string = ''
  sameAs(other: this) {
    return other.content === this.content
  }
}

class DerivedBox extends Box {
  otherContent: string = '?'
}

const base = new Box()
const derived = new DerivedBox()
derived.sameAs(base)
```

### 基于 `this` 的类型守卫

你可以在类和接口的方法的返回值位置使用 `this is Type`。
当与类型收窄（例如 `if` 语句）结合使用时，目标对象的类型将被收窄为指定的 `Type`。

<!-- prettier-ignore -->
```ts twoslash
// @strictPropertyInitialization: false
class FileSystemObject {
  isFile(): this is FileRep {
    return this instanceof FileRep;
  }
  isDirectory(): this is Directory {
    return this instanceof Directory;
  }
  isNetworked(): this is Networked & this {
    return this.networked;
  }
  constructor(public path: string, private networked: boolean) {}
}

class FileRep extends FileSystemObject {
  constructor(path: string, public content: string) {
    super(path, false);
  }
}

class Directory extends FileSystemObject {
  children: FileSystemObject[];
}

interface Networked {
  host: string;
}

const fso: FileSystemObject = new FileRep("foo/bar.txt", "foo");

if (fso.isFile()) {
  fso.content;
// ^?
} else if (fso.isDirectory()) {
  fso.children;
// ^?
} else if (fso.isNetworked()) {
  fso.host;
// ^?
}
```

基于 this 的类型守卫的一个常见用例是允许对特定字段进行惰性验证。例如，在验证 `hasValue` 为 true 后，下例会从 box 所保存的值中移除 `undefined`：

```ts twoslash
class Box<T> {
  value?: T

  hasValue(): this is { value: T } {
    return this.value !== undefined
  }
}

const box = new Box<string>()
box.value = 'Gameboy'

box.value
//  ^?

if (box.hasValue()) {
  box.value
  //  ^?
}
```

## 参数属性

TypeScript 提供了特殊的语法，可将构造函数参数直接转换为同名且同值的类属性。
这些被称为*参数属性*（parameter properties），通过在构造函数参数前添加可见性修饰符 `public`、`private`、`protected` 或 `readonly` 之一来创建。
由此生成的字段将获得这些修饰符：

```ts twoslash
// @errors: 2341
class Params {
  constructor(
    public readonly x: number,
    protected y: number,
    private z: number,
  ) {
    // No body necessary
  }
}
const a = new Params(1, 2, 3)
console.log(a.x)
//            ^?
console.log(a.z)
```

## 类表达式

<blockquote class='bg-reading'>
   <p>背景阅读：<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/class'>类表达式 (MDN)</a><br/>
   </p>
</blockquote>

类表达式与类声明非常相似。
它们之间唯一的真正区别在于类表达式不需要名称，不过我们仍然可以通过它们最终绑定到的任何标识符来引用它们：

```ts twoslash
const someClass = class<Type> {
  content: Type
  constructor(value: Type) {
    this.content = value
  }
}

const m = new someClass('Hello, world')
//    ^?
```

## 构造签名

JavaScript 类使用 `new` 运算符进行实例化。给定类本身的类型，[InstanceType](/docs/handbook/utility-types.html#instancetypetype) 工具类型可以为该操作建立类型模型。

```ts twoslash
class Point {
  createdAt: number
  x: number
  y: number
  constructor(x: number, y: number) {
    this.createdAt = Date.now()
    this.x = x
    this.y = y
  }
}
type PointInstance = InstanceType<typeof Point>

function moveRight(point: PointInstance) {
  point.x += 5
}

const point = new Point(3, 4)
moveRight(point)
point.x // => 8
```

## `abstract` 类与成员

TypeScript 中的类、方法和字段都可以是*抽象的*（abstract）。

*抽象方法*或*抽象字段*是指尚未提供具体实现的方法或字段。
这些成员必须存在于*抽象类*中，而抽象类不能被直接实例化。

抽象类的作用是作为子类的基类，由子类来实现所有抽象成员。
当一个类没有任何抽象成员时，它被称为*具体类*（concrete）。

让我们看一个例子：

```ts twoslash
// @errors: 2511
abstract class Base {
  abstract getName(): string

  printName() {
    console.log('Hello, ' + this.getName())
  }
}

const b = new Base()
```

我们不能使用 `new` 来实例化 `Base`，因为它是抽象类。
相反，我们需要创建一个派生类并实现其中的抽象成员：

```ts twoslash
abstract class Base {
  abstract getName(): string
  printName() {}
}
// ---cut---
class Derived extends Base {
  getName() {
    return 'world'
  }
}

const d = new Derived()
d.printName()
```

注意，如果我们忘记实现基类的抽象成员，就会收到一个错误提示：

```ts twoslash
// @errors: 2515
abstract class Base {
  abstract getName(): string
  printName() {}
}
// ---cut---
class Derived extends Base {
  // forgot to do anything
}
```

### 抽象构造签名

有时你希望接收某个类的构造函数，该函数能生成继承自某个抽象类的类的实例。

例如，你可能想编写这样的代码：

```ts twoslash
// @errors: 2511
abstract class Base {
  abstract getName(): string
  printName() {}
}
class Derived extends Base {
  getName() {
    return ''
  }
}
// ---cut---
function greet(ctor: typeof Base) {
  const instance = new ctor()
  instance.printName()
}
```

TypeScript 正确地提示你正在尝试实例化一个抽象类。
毕竟根据 `greet` 的定义，编写以下代码是完全合法的，但这最终会导致尝试构造一个抽象类：

```ts twoslash
declare const greet: any, Base: any
// ---cut---
// Bad!
greet(Base)
```

相反，你需要编写一个接收具有构造签名的实参的函数：

```ts twoslash
// @errors: 2345
abstract class Base {
  abstract getName(): string
  printName() {}
}
class Derived extends Base {
  getName() {
    return ''
  }
}
// ---cut---
function greet(ctor: new () => Base) {
  const instance = new ctor()
  instance.printName()
}
greet(Derived)
greet(Base)
```

现在，TypeScript 可以正确提示哪些类构造函数可以被调用——`Derived` 可以被调用，因为它是具体类；而 `Base` 则不能。

## 类之间的关系

在大多数情况下，TypeScript 中的类是按结构进行比较的，这与其他类型完全相同。

例如，以下两个类可以相互替换使用，因为它们的结构完全相同：

```ts twoslash
class Point1 {
  x = 0
  y = 0
}

class Point2 {
  x = 0
  y = 0
}

// OK
const p: Point1 = new Point2()
```

同样地，即使没有显式的继承关系，类之间也存在子类型关系：

```ts twoslash
// @strict: false
class Person {
  name: string
  age: number
}

class Employee {
  name: string
  age: number
  salary: number
}

// OK
const p: Person = new Employee()
```

这听起来很直观，但在某些情况下可能会显得有些奇特。

空类没有任何成员。
在结构化类型系统中，没有成员的类型通常是其他任何类型的超类型。
因此，如果你编写了一个空类（切勿这么做！），任何东西都可以替代它：

```ts twoslash
class Empty {}

function fn(x: Empty) {
  // can't do anything with 'x', so I won't
}

// All OK!
fn(window)
fn({})
fn(fn)
```
