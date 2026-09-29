---
title: 类
layout: docs
permalink: /zh/docs/handbook/2/classes.html
oneline: 'TypeScript 中类的工作原理'
---

<blockquote class='bg-reading'>
  <p>背景阅读：<br /><a href='https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes'>类 (MDN)</a></p>
</blockquote>

TypeScript 全面支持 ES2015 引入的 `class` 关键字。

与其他 JavaScript 语言特性一样，TypeScript 增加了类型注解及其他语法，让你能表达类与其他类型之间的关系。

## 类成员

这是一个最基本的空类：

```ts twoslash
class Point {}
```

这个类还没什么用处，接下来给它添加一些成员。

### 字段

字段声明会在类上创建一个公开的可写属性：

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

与其他位置一样，类型注解是可选的；如果不指定，隐式类型为 any。

字段也可以有 _initializers_（初始化器），它们会在类实例化时自动运行：

```ts twoslash
class Point {
  x = 0
  y = 0
}

const pt = new Point()
// 打印 0, 0
console.log(`${pt.x}, ${pt.y}`)
```

就像 const、let 和 var 一样，类属性的初始化器会用于推断其类型：

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

[`strictPropertyInitialization`](/tsconfig#strictPropertyInitialization) 设置控制是否必须在构造函数中初始化类字段。

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

注意，字段需要 _在构造函数本身中_ 进行初始化。TypeScript 不会分析你在构造函数中调用的方法来检测初始化，因为派生类可能会重写这些方法，从而导致成员未初始化。

如果打算通过构造函数以外的方式明确初始化字段（例如外部库帮你填充类的部分内容），可以使用 _明确赋值断言操作符_, `!`:

```ts twoslash
class OKGreeter {
  // 未初始化，但不报错
  name!: string
}
```

### `readonly`

可以为字段添加 `readonly` 修饰符前缀，这会阻止在构造函数外部对该字段进行赋值。

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
   <p>背景阅读:<br />
   <a href='https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes/constructor'>构造函数 (MDN)</a><br/>
   </p>
</blockquote>

类构造函数与普通函数非常相似。你可以添加带有类型注解、默认值和重载的参数：

```ts twoslash
class Point {
  x: number
  y: number

  // 带有默认值的常规签名
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

  // 构造函数重载
  constructor(x: number, y: number)
  constructor(xy: string)
  constructor(x: string | number, y: number = 0) {
    // 逻辑代码
  }
}
```

类构造函数签名与函数签名仅有几点区别：

- 构造函数不能有类型参数——类型参数属于外层类声明，稍后会介绍。
- 构造函数不能有返回类型注解——始终返回类实例类型。

#### super 调用

与 JavaScript 一样，如果存在基类，必须在使用任何 `this.` 成员之前，在构造函数体中调用 `super();`

```ts twoslash
// @errors: 17009
class Base {
  k = 4
}

class Derived extends Base {
  constructor() {
    // 在 ES5 中打印错误的值；在 ES6 中抛出异常
    console.log(this.k)
    super()
  }
}
```

在 JavaScript 中，忘记调用 `super` 是个容易犯的错，但 TypeScript 会在必要时提醒你。

### 方法

<blockquote class='bg-reading'>
   <p>背景阅读:<br />
   <a href='https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Method_definitions'>方法的定义</a><br/>
   </p>
</blockquote>

类上的函数属性称为 _方法_。方法可以使用与函数和构造函数相同的所有类型注解：

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

除了标准的类型注解，TypeScript 没有为方法增加任何新特性。

注意，在方法体内，仍然必须通过 `this`. 访问字段和其他方法。方法体中未限定名称的变量始终指向外层作用域中的内容：

```ts twoslash
// @errors: 2322
let x: number = 0

class C {
  x: string = 'hello'

  m() {
    // 这里试图修改第 1 行的 'x'，而不是类属性
    x = 'world'
  }
}
```

### 访问器 (Getters / Setters)

类也可以有 _访问器_：

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

> 注意，在 JavaScript 中，如果没有额外逻辑，单纯封装字段的 get/set 对几乎没有用处。
> 如果在获取/设置操作期间不需要附加逻辑，直接暴露公开字段即可。

TypeScript 对访问器有一些特殊的推断规则：

- 如果只有 `get` 没有 `set`，属性会自动变为 `readonly`。
- 如果没有指定 setter 参数的类型，它会根据 getter 的返回类型推断得出。

从 [TypeScript 4.3](https://devblogs.microsoft.com/typescript/announcing-typescript-4-3/) 开始，访问器在读取和设置时可以使用不同的类型。

```ts twoslash
class Thing {
  _size = 0

  get size(): number {
    return this._size
  }

  set size(value: string | number | boolean) {
    let num = Number(value)

    // 不允许 NaN、Infinity 等

    if (!Number.isFinite(num)) {
      this._size = 0
      return
    }

    this._size = num
  }
}
```

### 索引签名

类可以声明索引签名；它们的工作方式与 [其他对象类型的索引签名](/docs/handbook/2/objects.html#index-signatures) 相同：

```ts twoslash
class MyClass {
  [s: string]: boolean | ((s: string) => boolean)

  check(s: string) {
    return this[s] as boolean
  }
}
```

由于索引签名类型需要同时兼容方法的类型，因此很难有效利用这些类型。通常，最好将索引数据存储在其他地方，而不是直接放在类实例本身。

## 类继承

与具备面向对象特性的其他语言一样，JavaScript 中的类可以继承基类。

### `implements` 子句

可以使用 `implements` 子句检查类是否满足特定 `interface`（接口）。如果类未正确实现接口，会抛出错误：

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

类也可以实现多个接口，例如 `class C implements A, B {`。

#### 注意事项

需要明白，`implements` 子句仅仅检查类是否能被当作该接口类型处理。它 _完全_ 不会改变类或其方法的类型。一个常见的错误来源是假设 `implements` 子句会改变类类型——它并不会！

```ts twoslash
// @errors: 7006
interface Checkable {
  check(name: string): boolean
}

class NameChecker implements Checkable {
  check(s) {
    // 注意这里没有错误
    return s.toLowerCase() === 'ok'
    //         ^?
  }
}
```

在这个例子中，我们可能预期 `s` 的类型会受到 `check` 中 `name: string` 参数的影响。实际上并不会 —— `implements` 子句不改变类体的检查方式或类型推断。

同样，实现带有可选属性的接口并不会自动创建该属性：

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
   <p>背景阅读:<br />
   <a href='https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes/extends'>extends 关键字 (MDN)</a><br/>
   </p>
</blockquote>

类可以 `extend` 基类。派生类拥有基类的所有属性和方法，同时也可以定义附加成员。

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
// 基类方法
d.move()
// 派生类方法
d.woof(3)
```

#### 重写方法

<blockquote class='bg-reading'>
   <p>Background Reading:<br />
   <a href='https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/super'>super 关键字 (MDN)</a><br/>
   </p>
</blockquote>

派生类也可以重写基类的字段或属性。可以使用 `super.` 语法访问基类方法。注意，JavaScript 类只是简单的查找对象，不存在“父字段 (super field)”的概念。

TypeScript 强制要求派生类必须始终是其基类的子类型。

例如，下面是重写方法的合法方式：

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

派生类遵循其基类的契约非常重要。记住，通过基类引用指向派生类实例是非常常见（且始终合法）的做法：

```ts twoslash
class Base {
  greet() {
    console.log('Hello, world!')
  }
}
class Derived extends Base {}
const d = new Derived()
// ---cut---
// 通过基类引用建立派生实例的别名
const b: Base = d
// 没有问题
b.greet()
```

如果 `Derived` 不遵循 `Base` 的契约会怎样？

```ts twoslash
// @errors: 2416
class Base {
  greet() {
    console.log('Hello, world!')
  }
}

class Derived extends Base {
  // 使此参数成为必填项
  greet(name: string) {
    console.log(`Hello, ${name.toUpperCase()}`)
  }
}
```

如果忽略错误强行编译这段代码，示例就会崩溃：

```ts twoslash
declare class Base {
  greet(): void
}
declare class Derived extends Base {}
// ---cut---
const b: Base = new Derived()
// 崩溃，因为 "name" 将是 undefined
b.greet()
```

#### 仅类型字段声明

当 `target >= ES2022` 或 [`useDefineForClassFields`](/tsconfig#useDefineForClassFields) 为 `true` 时，类字段在父类构造函数完成后初始化，这会覆盖父类设置的任何值。如果你只想为继承的字段重新声明一个更精确的类型，这就成了问题。为了处理这些情况，可以写入 `declare` 来告诉 TypeScript 此字段声明不应产生运行时影响。

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
  // 不触发 JavaScript 代码生成，
  // 仅确保类型正确
  declare resident: Dog
  constructor(dog: Dog) {
    super(dog)
  }
}
```

#### 初始化顺序

在某些情况下，JavaScript 类的初始化顺序可能会让人感到意外。
来看这段代码：

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

// 打印 "base"，而不是 "derived"
const d = new Derived()
```

发生了什么？

根据 JavaScript 规定，类的初始化顺序为：

- 初始化基类字段
- 运行基类构造函数
- 初始化派生类字段
- 运行派生类构造函数

这意味着在基类构造函数执行期间，它看到的是自己定义的 `name` 值，因为派生类的字段初始化尚未运行。

#### 继承内置类型

> 注意：如果你不打算继承 `Array`、`Error`、`Map` 等内置类型，或者编译目标明确设置为 `ES6`/`ES2015` 及以上，可以跳过此节。

在 ES2015 中，返回对象的构造函数隐式地将 `this` 的值替换给任何 `super(...)` 调用者。生成的构造函数代码必须捕获 `super(...)` 的任何潜在返回值并将其替换为 `this`。

因此，继承 `Error`、`Array` 等类型可能不再按预期工作。这是因为 `Error`、`Array` 等构造函数使用 ECMAScript 6 的 `new.target` 调整原型链；然而，在 ECMAScript 5 中调用构造函数时，无法确保 `new.target` 的值。默认情况下，其他降级编译器通常也有同样的限制。

对于类似下面的子类：

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

- 在构造这些子类返回的对象上，方法可能是 `undefined`，因此调用 `sayHello` 会报错。
- 子类实例及其自身之间的 `instanceof` 会失效，因此 `(new MsgError()) instanceof MsgError` 将返回 `false`。

作为建议，可以在任何 `super(...)` 调用后立即手动调整原型。

```ts twoslash
class MsgError extends Error {
  constructor(m: string) {
    super(m)

    // 显式设置原型。
    Object.setPrototypeOf(this, MsgError.prototype)
  }

  sayHello() {
    return 'hello ' + this.message
  }
}
```

然而，`MsgError` 的任何子类也必须手动设置原型。对于不支持[`Object.setPrototypeOf`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf) 的运行时，也许可以使用[`__proto__`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/proto) 替代。

遗憾的是，[这些变通方法在 Internet Explorer 10 及更早版本上无效](<https://msdn.microsoft.com/en-us/library/s4esdbwz(v=vs.94).aspx>)。你可以手动将方法从原型复制到实例本身（即把 `MsgError.prototype` 复制到 `this`），但原型链本身无法修复。

## 成员可见性

可以使用 TypeScript 控制某些方法或属性对类外部的代码是否可见。

### `public`

类成员的默认可见性是 `public`。`public` 成员随处均可访问：

```ts twoslash
class Greeter {
  public greet() {
    console.log('hi!')
  }
}
const g = new Greeter()
g.greet()
```

因为 `public` 已经是默认的可见性修饰符，所以不 _需要_ 在类成员上显式编写，但出于风格或可读性考虑，可以选择这么做。

### `protected`

`protected` 成员仅对其所在类的子类可见。

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
    // 此处可以正常访问受保护成员
    console.log('Howdy, ' + this.getName())
    //                          ^^^^^^^^^^^^^^
  }
}
const g = new SpecialGreeter()
g.greet() // OK
g.getName()
```

#### 暴露 `protected` 成员

派生类需要遵循其基类契约，但可以选择暴露具备更多能力的基类子类型。这包括将 `protected` 成员变为 `public`：

```ts twoslash
class Base {
  protected m = 10
}
class Derived extends Base {
  // 无修饰符，默认是 'public'
  m = 15
}
const d = new Derived()
console.log(d.m) // OK
```

注意，`Derived` 本身就能自由读写 `m`，因此这并未实质性地改变这里的“安全性”。此处主要需要注意的是，在派生类中，如果不想有意暴露，必须小心重复使用 `protected` 修饰符。

#### 跨层级 `protected` 访问

TypeScript 不允许在类层级中访问兄弟类的 `protected` 成员：

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

这是因为在 `Derived2` 中访问 `x` 应该只对 `Derived2` 的子类合法，而 `Derived1` 并非其中之一。此外，如果通过 `Derived1` 引用访问 `x` 是非法的（理应如此），那么通过基类引用访问它也绝不应该改善这一状况。

See also [Why Can’t I Access A Protected Member From A Derived Class?](https://blogs.msdn.microsoft.com/ericlippert/2005/11/09/why-cant-i-access-a-protected-member-from-a-derived-class/) which explains more of C#'s reasoning on the same topic.

### `private`

`private` 类似 `protected`，但即使是子类也不允许访问该成员：

```ts twoslash
// @errors: 2341
class Base {
  private x = 0
}
const b = new Base()
// 无法在类外部访问
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
    // 无法在子类中访问
    console.log(this.x)
  }
}
```

因为 `private` 成员对派生类不可见，所以派生类无法提高它们的可见性：

```ts twoslash
// @errors: 2415
class Base {
  private x = 0
}
class Derived extends Base {
  x = 1
}
```

#### 跨实例 `private` 访问

对于相同类的不同实例是否可以访问彼此的 `private` 成员，不同的面向对象语言存在分歧。Java、C#、C++、Swift 和 PHP 等语言允许这种做法，但 Ruby 不允许。

TypeScript 允许跨实例的 `private` 访问：

```ts twoslash
class A {
  private x = 10

  public sameAs(other: A) {
    // 不报错
    return other.x === this.x
  }
}
```

#### 注意事项

与 TypeScript 类型系统的其他方面一样，`private` 和 `protected` [仅在类型检查期间强制执行](https://www.typescriptlang.org/play?removeComments=true&target=99&ts=4.3.4#code/PTAEGMBsEMGddAEQPYHNQBMCmVoCcsEAHPASwDdoAXLUAM1K0gwQFdZSA7dAKWkoDK4MkSoByBAGJQJLAwAeAWABQIUH0HDSoiTLKUaoUggAW+DHorUsAOlABJcQlhUy4KpACeoLJzrI8cCwMGxU1ABVPIiwhESpMZEJQTmR4lxFQaQxWMm4IZABbIlIYKlJkTlDlXHgkNFAAbxVQTIAjfABrAEEC5FZOeIBeUAAGAG5mmSw8WAroSFIqb2GAIjMiIk8VieVJ8Ar01ncAgAoASkaAXxVr3dUwGoQAYWpMHBgCYn1rekZmNg4eUi0Vi2icoBWJCsNBWoA6WE8AHcAiEwmBgTEtDovtDaMZQLM6PEoQZbA5wSk0q5SO4vD4-AEghZoJwLGYEIRwNBoqAzFRwCZCFUIlFMXECdSiAhId8YZgclx0PsiiVqOVOAAaUAFLAsxWgKiC35MFigfC0FKgSAVVDTSyk+W5dB4fplHVVR6gF7xJrKFotEk-HXIRE9PoDUDDcaTAPTWaceaLZYQlmoPBbHYx-KcQ7HPDnK43FQqfY5+IMDDISPJLCIuqoc47UsuUCofAME3Vzi1r3URvF5QV5A2STtPDdXqunZDgDaYlHnTDrrEAF0dm28B3mDZg6HJwN1+2-hg57ulwNV2NQGoZbjYfNrYiENBwEFaojFiZQK08C-4fFKTVCozWfTgfFgLkeT5AUqiAA)。

这意味着，像 `in` 或简单的属性查找等 JavaScript 运行时结构，依然可以访问 `private` 或 `protected` 成员：

```ts twoslash
class MySafe {
  private secretKey = 12345
}
```

```js
// 在 JavaScript 文件中...
const s = new MySafe()
// 会打印 12345
console.log(s.secretKey)
```

`private` 还允许在类型检查期间使用方括号表示法进行访问。这使得单元测试等场景更容易访问声明为 `private` 的字段，缺点是这些字段属于 _软私有_（soft private），并不严格保证隐私。

```ts twoslash
// @errors: 2341
class MySafe {
  private secretKey = 12345
}

const s = new MySafe()

// 类型检查期间不允许
console.log(s.secretKey)

// OK
console.log(s['secretKey'])
```

与 TypeScript 的 `private` 不同，JavaScript 的 [私有字段](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes/Private_class_fields) (`#`) 在编译后依然保持私有，并且不提供前面提到的方括号访问等逃生舱，属于 _硬私有_（hard private）。

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

当编译至 ES2021 或更低版本时，TypeScript 会使用 WeakMaps 代替 `#`。

```ts twoslash
// @target: es2015
// @showEmit
class Dog {
  #barkAmount = 0
  personality = 'happy'

  constructor() {}
}
```

如果需要保护类中的值免受恶意行为者侵害，应使用提供硬运行时隐私的机制，如闭包、WeakMaps 或私有字段。注意，这些运行时额外的隐私检查可能会影响性能。

## 静态成员

<blockquote class='bg-reading'>
   <p>背景阅读:<br />
   <a href='https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes/static'>静态成员 (MDN)</a><br/>
   </p>
</blockquote>

类可以有 `static` 成员。这些成员不与类的特定实例关联。可以通过类构造函数对象本身访问它们：

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

静态成员也可以使用相同的 `public`、`protected` 和 `private` 可见性修饰符：

```ts twoslash
// @errors: 2341
class MyClass {
  private static x = 0
}
console.log(MyClass.x)
```

静态成员同样会被继承：

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

### 特殊静态名称

从 `Function` 原型覆盖属性通常是不安全或不可能的。由于类本身可以是被 `new` 调用的函数，某些 `static` 名称不能使用。`name`、`length` 和 `call` 等函数属性不能定义为 `static` 成员：

```ts twoslash
// @errors: 2699
class S {
  static name = 'S!'
}
```

### 为什么没有静态类？

TypeScript（和 JavaScript）不像 C# 等语言那样具备名为 `static class` 的结构。

这些结构存在 _仅仅_ 是因为那些语言强制所有数据和函数都必须在类里面；而 TypeScript 没有这种限制，所以不需要它们。只有一个实例的类通常在 JavaScript/TypeScript 中直接表示为普通的 _对象_。

例如，TypeScript 中不需要“静态类”语法，因为普通对象（甚至顶层函数）同样能很好地完成任务：

```ts twoslash
// 没必要的“静态”类
class MyStaticClass {
  static doSomething() {}
}

// 推荐做法（选项 1）
function doSomething() {}

// 推荐做法（选项 2）
const MyHelperObject = {
  dosomething() {},
}
```

## 类中的 `static` 块

静态块允许你编写带有自身作用域的语句序列，这些语句可以访问包含类中的私有字段。这意味着我们可以编写具有全部语句能力的初始化代码，不会造成变量泄露，并且拥有访问类内部的完全权限。

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

类与接口非常相似，可以是泛型的。当使用 `new` 实例化泛型类时，其类型参数的推断方式与函数调用相同：

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

这段代码不合法，原因可能不那么显而易见：

```ts twoslash
// @errors: 2302
class Box<Type> {
  static defaultValue: Type
}
```

记住，类型总是会被完全擦除！在运行时，`Box.defaultValue` 属性槽只有 _一个_。这意味着设置 `Box<string>.defaultValue`（如果可行的话）会 _同时_ 改变 `Box<number>.defaultValue` —— 这很糟糕。泛型类的 `static` 成员永远不能引用该类的类型参数。

## 类中运行时的 `this`

<blockquote class='bg-reading'>
   <p>背景阅读:<br />
   <a href='https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this'>this 关键字 (MDN)</a><br/>
   </p>
</blockquote>

重要的是要记住，TypeScript 并不改变 JavaScript 的运行时行为，而 JavaScript 正以一些奇特的运行时行为而闻名。

JavaScript 对 `this` 的处理确实与众不同：

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

// 打印 "obj"，而不是 "MyClass"
console.log(obj.getName())
```

简而言之，默认情况下，函数内部 `this` 的值取决于 _函数是如何被调用的_。在这个例子中，因为函数是通过 `obj` 引用调用的，所以它的 `this` 值是 `obj` 而不是类实例。

这几乎不是你想要的结果！TypeScript 提供了一些方法来减轻或防止这种错误。

### 箭头函数

<blockquote class='bg-reading'>
   <p>背景阅读:<br />
   <a href='https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions'>箭头函数 (MDN)</a><br/>
   </p>
</blockquote>

如果有一个函数经常在丢失其 `this` 上下文的情况下被调用，使用箭头函数属性代替方法定义是合理的：

```ts twoslash
class MyClass {
  name = 'MyClass'
  getName = () => {
    return this.name
  }
}
const c = new MyClass()
const g = c.getName
// 打印 "MyClass" 而不是崩溃
console.log(g())
```

这伴随一些权衡：

- 保证在运行时 `this` 的值是正确的，即使代码没有经过 TypeScript 检查。
- 这会占用更多内存，因为每个类实例都会保留按此方式定义的函数的独立副本。
- 不能在派生类中使用 `super.getName`，因为原型链中没有供获取基类方法的条目。

### `this` 参数

在方法或函数定义中，名为 `this` 的初始参数在 TypeScript 中具有特殊含义。这些参数会在编译期间被擦除：

```ts twoslash
type SomeType = any
// ---cut---
// 带有 'this' 参数的 TypeScript 输入
function fn(this: SomeType, x: number) {
  /* ... */
}
```

```js
// JavaScript 输出
function fn(x) {
  /* ... */
}
```

TypeScript 检查调用带有 `this` 参数的函数时上下文是否正确。我们可以向方法定义添加 `this` 参数以静态强制方法被正确调用，而不是使用箭头函数：

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

// 错误，会崩溃
const g = c.getName
console.log(g())
```

这种方法在权衡上与箭头函数截然相反：

- JavaScript 调用者在未意识到的情况下，仍然可能错误地使用类方法。
- 每个类定义只分配一个函数，而不是每个类实例分配一个。
- 仍可以通过 `super` 调用基类方法定义。

## `this` 类型

在类中，名为 `this` 的特殊类型会 _动态_ 指向当前类的类型。我们来看看它有什么用：

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

此处，TypeScript 将 `set` 的返回类型推断为 `this`，而不是 `Box`。现在我们创建一个 `Box` 的子类：

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

也可以在参数类型注解中使用 `this`：

```ts twoslash
class Box {
  content: string = ''
  sameAs(other: this) {
    return other.content === this.content
  }
}
```

这与写 `other: Box` 不同——如果有一个派生类，它的 `sameAs` 方法现在只接受该相同派生类的其他实例：

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

可以在类和接口的方法返回位置使用 `this is Type`。当结合类型收窄（例如 `if` 语句）时，目标对象的类型将被收窄为指定的 `Type`。

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

基于 this 的类型守卫的一个常见用例是允许对特定字段进行延迟验证。例如，在下面这种情况中，当 `hasValue` 被验证为 true 时，就会移除 box 内部保留值中的 `undefined`：

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

TypeScript 提供了特殊语法，可将构造函数参数转换为具有相同名称和值的类属性。这些被称为 _参数属性_，通过在构造函数参数前加上 `public`、`private`、`protected` 或 `readonly` 等可见性修饰符来创建。结果字段也会得到相应的修饰符：

```ts twoslash
// @errors: 2341
class Params {
  constructor(
    public readonly x: number,
    protected y: number,
    private z: number,
  ) {
    // 不需要方法体
  }
}
const a = new Params(1, 2, 3)
console.log(a.x)
//            ^?
console.log(a.z)
```

## 类表达式

<blockquote class='bg-reading'>
   <p>背景阅读:<br />
   <a href='https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/class'>类表达式 (MDN)</a><br/>
   </p>
</blockquote>

类表达式与类声明非常相似。唯一真正的区别在于类表达式不需要名称，尽管我们可以通过它们绑定的任何标识符来引用它们：

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

## 构造函数签名

JavaScript 类使用 `new` 操作符实例化。给定类本身的类型，[InstanceType](/docs/handbook/utility-types.html#instancetypetype) 实用类型可对此操作建模。

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

TypeScript 中的类、方法和字段可以是 _抽象的_。

_抽象方法_ 或 _抽象字段_ 尚未提供实现。这些成员必须存在于 _抽象类_ 内部，抽象类不能直接实例化。

抽象类的作用是作为子类的基类，由子类实现所有的抽象成员。当一个类没有包含任何抽象成员时，它被称为 _具体类_（concrete）。

来看一个例子：

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

因为 `Base` 是抽象的，所以不能用 `new` 将其实例化。相反，需要创建一个派生类并实现抽象成员：

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

注意，如果忘记实现基类的抽象成员，将会报错：

```ts twoslash
// @errors: 2515
abstract class Base {
  abstract getName(): string
  printName() {}
}
// ---cut---
class Derived extends Base {
  // 忘记做任何事
}
```

### 抽象构造函数签名

有时，你希望接受能够生成派生自某个抽象类实例的类构造函数。

例如，你可能想写如下代码：

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

TypeScript 正确地指出你正在尝试实例化一个抽象类。毕竟，根据 `greet` 的定义，编写以下代码完全合法，但这将导致构造出抽象类：

```ts twoslash
declare const greet: any, Base: any
// ---cut---
// 糟糕！
greet(Base)
```

相反，你需要编写一个接受带有构造函数签名的参数的函数：

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

现在 TypeScript 可以正确告诉你哪些类构造函数能够被调用 —— `Derived` 可以调用，因为它是具体类，而 `Base` 不行。

## 类之间的关系

在大多数情况下，TypeScript 中的类结构对比与其他类型相同。

例如，以下两个类可以互换使用，因为它们结构完全相同：

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

同样，即使没有显式的继承关系，类之间也可能存在子类型关系：

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

这听起来很简单，但有几种情况似乎有些反常。

空类没有任何成员。在结构化类型系统中，没有成员的类型通常是任何其他类型的超类型。因此，如果你编写一个空类（请不要这样做！），任何东西都能用来代替它：

```ts twoslash
class Empty {}

function fn(x: Empty) {
  // 对 'x' 做不了任何事，所以我不做
}

// 都可以！
fn(window)
fn({})
fn(fn)
```
