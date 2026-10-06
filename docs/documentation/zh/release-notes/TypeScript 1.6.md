---
title: TypeScript 1.6
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-1-6.html
oneline: TypeScript 1.6 发布说明
---

## JSX 支持

JSX 是一种可嵌入的类 XML 语法。
它旨在被转换为合法的 JavaScript，但该转换的具体语义取决于具体实现。
JSX 随着 React 库而流行起来，但此后也出现了其他应用场景。
TypeScript 1.6 支持嵌入、类型检查，并可选择直接将 JSX 编译为 JavaScript。

#### 新的 `.tsx` 文件扩展名与 `as` 操作符

TypeScript 1.6 引入了全新的 `.tsx` 文件扩展名。
该扩展名有两个作用：一是在 TypeScript 文件中启用 JSX；二是将新的 `as` 操作符作为类型断言的默认方式（消除了 JSX 表达式与 TypeScript 前缀类型断言操作符之间的二义性）。
例如：

```ts
var x = <any>foo
// is equivalent to:
var x = foo as any
```

#### 使用 React

若要配合 React 使用 JSX 支持，你应该使用 [React 类型定义](https://github.com/borisyankov/DefinitelyTyped/tree/master/react)。这些类型定义声明了 `JSX` 命名空间，从而使 TypeScript 能够正确检查适用于 React 的 JSX 表达式。例如：

```ts
/// <reference path="react.d.ts" />

interface Props {
  name: string;
}

class MyComponent extends React.Component<Props, {}> {
  render() {
    return <span>{this.props.name}</span>;
  }
}

<MyComponent name="bar" />; // OK
<MyComponent name={0} />; // error, `name` is not a number
```

#### 使用其他 JSX 框架

JSX 元素名称和属性会根据 `JSX` 命名空间进行验证。
关于如何为你的框架定义 `JSX` 命名空间，请参阅 [[JSX]] wiki 页面。

#### 输出生成

TypeScript 内置了两种 JSX 模式：`preserve` 和 `react`。

- `preserve` 模式会将 JSX 表达式保留在输出中，以便供后续的另一转换步骤处理。_此外，输出文件的扩展名为 `.jsx`。_
- `react` 模式会生成 `React.createElement` 调用，在使用前无需再经过 JSX 转换，且输出文件的扩展名为 `.js`。

关于在 TypeScript 中使用 JSX 的更多信息，请参阅 [[JSX]] wiki 页面。

## 交叉类型

TypeScript 1.6 引入了交叉类型（intersection types），它是联合类型的逻辑互补。
联合类型 `A | B` 表示一个实体可以是类型 `A` 或类型 `B`，而交叉类型 `A & B` 则表示一个实体同时具备类型 `A` _和_ 类型 `B`。

##### 示例

```ts
function extend<T, U>(first: T, second: U): T & U {
  let result = <T & U>{}
  for (let id in first) {
    result[id] = first[id]
  }
  for (let id in second) {
    if (!result.hasOwnProperty(id)) {
      result[id] = second[id]
    }
  }
  return result
}

var x = extend({ a: 'hello' }, { b: 42 })
var s = x.a
var n = x.b
```

```ts
type LinkedList<T> = T & { next: LinkedList<T> }

interface Person {
  name: string
}

var people: LinkedList<Person>
var s = people.name
var s = people.next.name
var s = people.next.next.name
var s = people.next.next.next.name
```

```ts
interface A {
  a: string
}
interface B {
  b: string
}
interface C {
  c: string
}

var abc: A & B & C
abc.a = 'hello'
abc.b = 'hello'
abc.c = 'hello'
```

更多信息请参阅 [issue #1256](https://github.com/Microsoft/TypeScript/issues/1256)。

## 局部类型声明

局部类、接口、枚举以及类型别名声明现在可以出现在函数声明内部。局部类型具有块级作用域，类似于使用 `let` 和 `const` 声明的变量。例如：

```ts
function f() {
  if (true) {
    interface T {
      x: number
    }
    let v: T
    v.x = 5
  } else {
    interface T {
      x: string
    }
    let v: T
    v.x = 'hello'
  }
}
```

推断出的函数返回类型可以是函数内部局部声明的类型。函数的调用方无法直接引用此类局部类型，但当然可以通过结构化方式进行匹配。例如：

```ts
interface Point {
  x: number
  y: number
}

function getPointFactory(x: number, y: number) {
  class P {
    x = x
    y = y
  }
  return P
}

var PointZero = getPointFactory(0, 0)
var PointOne = getPointFactory(1, 1)
var p1 = new PointZero()
var p2 = new PointZero()
var p3 = new PointOne()
```

局部类型可以引用外层的类型参数，且局部类和接口自身也可以是泛型的。例如：

```ts
function f3() {
  function f<X, Y>(x: X, y: Y) {
    class C {
      public x = x
      public y = y
    }
    return C
  }
  let C = f(10, 'hello')
  let v = new C()
  let x = v.x // number
  let y = v.y // string
}
```

## 类表达式

TypeScript 1.6 增加了对 ES6 类表达式的支持。在类表达式中，类名是可选的；如果指定了类名，则该名称仅在类表达式自身的作用域内有效。这与函数表达式的可选名称类似。在类表达式外部无法直接引用该类表达式的类实例类型，但该类型可以通过结构化方式进行匹配。例如：

```ts
let Point = class {
  constructor(
    public x: number,
    public y: number,
  ) {}
  public length() {
    return Math.sqrt(this.x * this.x + this.y * this.y)
  }
}
var p = new Point(3, 4) // p has anonymous class type
console.log(p.length())
```

## 继承表达式

TypeScript 1.6 增加了对类继承求值结果为构造函数的任意表达式的支持。这意味着内置类型现在也可以在类声明中被继承。

此前类的 `extends` 子句要求指定一个类型引用。现在它允许接收一个表达式，并可后跟类型参数列表。该表达式的类型必须是构造函数类型，且至少具有一个构造签名，该签名的类型参数数量需与 `extends` 子句中指定的类型参数数量一致。匹配的构造签名的返回类型即为类实例类型所继承的基类型。实际上，这使得真正的类与“类式（class-like）”表达式都可以出现在 `extends` 子句中。

示例：

```ts
// Extend built-in types

class MyArray extends Array<number> {}
class MyError extends Error {}

// Extend computed base class

class ThingA {
  getGreeting() {
    return 'Hello from A'
  }
}

class ThingB {
  getGreeting() {
    return 'Hello from B'
  }
}

interface Greeter {
  getGreeting(): string
}

interface GreeterConstructor {
  new (): Greeter
}

function getGreeterBase(): GreeterConstructor {
  return Math.random() >= 0.5 ? ThingA : ThingB
}

class Test extends getGreeterBase() {
  sayHello() {
    console.log(this.getGreeting())
  }
}
```

## `abstract` 抽象类与抽象方法

TypeScript 1.6 增加了对类及其方法的 `abstract` 关键字支持。抽象类允许包含没有实现的方法，且无法被实例化。

##### 示例

```ts
abstract class Base {
  abstract getThing(): string
  getOtherThing() {
    return 'hello'
  }
}

let x = new Base() // Error, 'Base' is abstract

// Error, must either be 'abstract' or implement concrete 'getThing'
class Derived1 extends Base {}

class Derived2 extends Base {
  getThing() {
    return 'hello'
  }
  foo() {
    super.getThing() // Error: cannot invoke abstract members through 'super'
  }
}

var x = new Derived2() // OK
var y: Base = new Derived2() // Also OK
y.getThing() // OK
y.getOtherThing() // OK
```

## 泛型类型别名

在 TypeScript 1.6 中，类型别名可以是泛型的。例如：

```ts
type Lazy<T> = T | (() => T)

var s: Lazy<string>
s = 'eager'
s = () => 'lazy'

interface Tuple<A, B> {
  a: A
  b: B
}

type Pair<T> = Tuple<T, T>
```

## 更严格的对象字面量赋值检查

TypeScript 1.6 实施了更严格的对象字面量赋值检查，旨在捕获多余或拼写错误的属性。具体而言，当全新的对象字面量被赋值给变量，或作为实参传递给非空目标类型时，如果对象字面量指定了目标类型中不存在的属性，则会报错。

##### 示例

```ts
var x: { foo: number }
x = { foo: 1, baz: 2 } // Error, excess property `baz`

var y: { foo: number; bar?: number }
y = { foo: 1, baz: 2 } // Error, excess or misspelled property `baz`
```

类型可以通过包含索引签名来显式允许存在多余属性：

```ts
var x: { foo: number; [x: string]: any }
x = { foo: 1, baz: 2 } // Ok, `baz` matched by index signature
```

## ES6 生成器

TypeScript 1.6 增加了在面向 ES6 时的生成器支持。

生成器函数与普通函数一样可以带有返回类型注解。该注解表示该函数返回的生成器的类型。示例如下：

```ts
function* g(): Iterable<string> {
  for (var i = 0; i < 100; i++) {
    yield '' // string is assignable to string
  }
  yield* otherStringGenerator() // otherStringGenerator must be iterable and element type assignable to string
}
```

没有类型注解的生成器函数可以自动推断其类型注解。
因此在以下情况下，类型将从 yield 语句中推断得出：

```ts
function* g() {
  for (var i = 0; i < 100; i++) {
    yield '' // infer string
  }
  yield* otherStringGenerator() // infer element type of otherStringGenerator
}
```

## 对 `async` 函数的实验性支持

TypeScript 1.6 引入了在面向 ES6 时对 `async` 函数的实验性支持。
异步函数能够调用异步操作并等待其结果，而不会阻塞程序的正常执行。
这是通过使用兼容 ES6 的 `Promise` 实现，并将函数体转换为兼容形式以便在所等待的异步操作完成时恢复执行来实现的。

_async 函数_ 是带有 `async` 修饰符前缀的函数或方法。该修饰符通知编译器需要对函数体进行转换，并将关键字 `await` 视为一元表达式而不是标识符。
_async 函数_ 必须提供指向兼容 `Promise` 类型的返回类型注解。只有在存在全局定义的兼容 `Promise` 类型时，才能使用返回类型推断。

##### 示例

```ts
var p: Promise<number> = /* ... */;
async function fn(): Promise<number> {
  var i = await p; // suspend execution until 'p' is settled. 'i' has type "number"
  return 1 + i;
}

var a = async (): Promise<number> => 1 + await p; // suspends execution.
var a = async () => 1 + await p; // suspends execution. return type is inferred as "Promise<number>" when compiling with --target ES6
var fe = async function(): Promise<number> {
  var i = await p; // suspend execution until 'p' is settled. 'i' has type "number"
  return 1 + i;
}

class C {
  async m(): Promise<number> {
    var i = await p; // suspend execution until 'p' is settled. 'i' has type "number"
    return 1 + i;
  }

  async get p(): Promise<number> {
    var i = await p; // suspend execution until 'p' is settled. 'i' has type "number"
    return 1 + i;
  }
}
```

## 每夜构建版本

虽然严格来说这不属于语言特性的变更，但现在可以通过以下命令安装每夜构建版本（Nightly builds）：

```Shell
npm install -g typescript@next
```

## 模块解析逻辑的调整

从 1.6 版本开始，当目标设置为 'commonjs' 时，TypeScript 编译器将采用一套不同的规则来解析模块名称。
这些[规则](https://github.com/Microsoft/TypeScript/issues/2338)旨在模拟 Node 所使用的模块查找流程。
这实际上意味着 Node 模块可以包含其类型定义信息，且 TypeScript 编译器能够找到它。
不过，用户可以使用 [`moduleResolution`](/tsconfig#moduleResolution) 命令行选项覆盖编译器所选的模块解析规则。可选值为：

- 'classic' - 1.6 之前的 TypeScript 编译器所使用的模块解析规则
- 'node' - 类似 Node 的模块解析

## 合并环境类与接口声明

环境类（ambient class）声明的实例侧可以通过接口声明进行扩展。类构造函数对象保持不变。
例如：

```ts
declare class Foo {
  public x: number
}

interface Foo {
  y: string
}

function bar(foo: Foo) {
  foo.x = 1 // OK, declared in the class Foo
  foo.y = '1' // OK, declared in the interface Foo
}
```

## 用户自定义类型保护函数

除 `typeof` 和 `instanceof` 之外，TypeScript 1.6 增加了一种在 `if` 代码块内部收窄变量类型的新方法。
用户自定义类型保护函数是返回类型注解形如 `x is T` 的函数，其中 `x` 是签名中声明的形参，`T` 是任意类型。
在 `if` 块中对变量调用用户自定义类型保护函数时，该变量的类型将被收窄为 `T`。

##### 示例

```ts
function isCat(a: any): a is Cat {
  return a.name === 'kitty'
}

var x: Cat | Dog
if (isCat(x)) {
  x.meow() // OK, x is Cat in this block
}
```

## tsconfig.json 支持 `exclude` 属性

未指定 files 属性（因而隐式引用所有子目录中的所有 \*.ts 文件）的 tsconfig.json 文件，现在可以包含一个 exclude 属性，用于指定要从编译中排除的文件和/或目录列表。
exclude 属性必须是字符串数组，其中每个字符串指定相对于 tsconfig.json 文件位置的文件或文件夹名称。
例如：

```json tsconfig
{
  "compilerOptions": {
    "out": "test.js"
  },
  "exclude": ["node_modules", "test.ts", "utils/t2.ts"]
}
```

[`exclude`](/tsconfig#exclude) 列表不支持通配符。它必须仅仅是文件和/或目录的列表。

## `--init` 命令行选项

在某个目录下运行 `tsc --init`，即可在该目录中使用预设的默认值创建初始的 `tsconfig.json`。
还可以配合 `--init` 传入命令行参数，以便在创建时直接保存到你的初始 tsconfig.json 中。
