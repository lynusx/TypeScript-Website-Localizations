---
title: 函数进阶
layout: docs
permalink: /zh/docs/handbook/2/functions.html
oneline: '了解 TypeScript 中函数的工作原理。'
---

函数是任何应用程序的基本构建块，无论是局部函数、从其他模块导入的函数，还是类的方法。函数也是值。与其他值一样，TypeScript 提供了多种方式来描述函数的调用方式。下面我们来学习如何编写描述函数的类型。

## 函数类型表达式

描述函数最简单的方式是使用 _函数类型表达式_。这种类型在语法上类似于箭头函数：

```ts twoslash
function greeter(fn: (a: string) => void) {
  fn('Hello, World')
}

function printToConsole(s: string) {
  console.log(s)
}

greeter(printToConsole)
```

语法 `(a: string) => void` 表示“一个具有名为 `a`、类型为 `string` 的参数，且没有返回值的函数”。与函数声明一样，如果没有指定参数类型，它会隐式地变为 `any` 类型。

> 注意，参数名是 **必需的**。函数类型 `(string) => void` 表示“一个名为 `string`，类型为 `any` 的参数的函数”！

当然，你可以使用类型别名来为函数类型命名：

```ts twoslash
type GreetFunction = (a: string) => void
function greeter(fn: GreetFunction) {
  // ...
}
```

## 调用签名

在 JavaScript 中，函数除了可以调用，还可以拥有属性。然而，函数类型表达式语法不允许声明属性。如果想描述带有属性的可调用对象，可以在对象类型中编写 _调用签名_：

```ts twoslash
type DescribableFunction = {
  description: string;
  (someArg: number): boolean
}
function doSomething(fn: DescribableFunction) {
  console.log(fn.description + ' returned ' + fn(6))
}

function myFunc(someArg: number) {
  return someArg > 3
}
myFunc.description = 'default description'

doSomething(myFunc)
```

注意，该语法与函数类型表达式略有不同：在参数列表和返回值类型之间使用的是 `:` 而不是 `=>`。

## 构造签名

JavaScript 函数也可以使用 `new` 运算符调用。TypeScript 将其称为 _构造函数_，因为它们通常会创建一个新对象。你可以在调用签名前加上 `new` 关键字来编写 _构造签名_：

```ts twoslash
type SomeObject = any
// ---cut---
type SomeConstructor = {
  new (s: string): SomeObject
}
function fn(ctor: SomeConstructor) {
  return new ctor('hello')
}
```

某些对象（如 JavaScript 的 `Date` 对象）可以带或不带 `new` 进行调用。你可以在同一个类型中任意组合调用签名和构造签名：

```ts twoslash
interface CallOrConstruct {
  (n?: number): string
  new (s: string): Date
}

function fn(ctor: CallOrConstruct) {
  // 传入 number 类型的参数给 `ctor` 将匹配
  // `CallOrConstruct` 接口中的第一个定义。
  console.log(ctor(10))
  // ^?

  // 同理，传入 string 类型的参数给 `ctor` 将匹配
  // `CallOrConstruct` 接口中的第二个定义。
  console.log(new ctor('10'))
  // ^?
}

fn(Date)
```

## 泛型函数

我们经常需要编写输入类型与输出类型相关，或者两个输入类型之间存在某种关联的函数。让我们先来看一个返回数组第一个元素的函数：

```ts twoslash
function firstElement(arr: any[]) {
  return arr[0]
}
```

这个函数完成了它的工作，但遗憾的是其返回值类型为 `any`。如果函数能返回数组元素的具体类型会更好。

在 TypeScript 中，当需要描述两个值之间的对应关系时，我们会使用 _泛型_。为此，我们在函数签名中声明一个类型参数：

```ts twoslash
function firstElement<Type>(arr: Type[]): Type | undefined {
  return arr[0]
}
```

通过在函数中添加类型参数 `Type` 并在两处使用它，我们在函数的输入（数组）和输出（返回值）之间建立了联系。现在调用它时，就能得出一个更具体的类型：

```ts twoslash
declare function firstElement<Type>(arr: Type[]): Type | undefined
// ---cut---
// s 的类型是 'string'
const s = firstElement(['a', 'b', 'c'])
// n 的类型是 'number'
const n = firstElement([1, 2, 3])
// u 的类型是 undefined
const u = firstElement([])
```

### 类型推断

注意，在这个示例中我们不必显式指定 `Type`。TypeScript _推断_（自动选择）了该类型。

我们也可以使用多个类型参数。例如，独立版本的 `map` 函数如下所示：

```ts twoslash
// prettier-ignore
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func);
}

// 参数 'n' 的类型是 'string'
// 'parsed' 的类型是 'number[]'
const parsed = map(['1', '2', '3'], (n) => parseInt(n))
```

注意，在这个示例中，TypeScript 不仅能根据给定的 `string` 数组推断出 `Input` 类型参数，还能根据函数表达式的返回值（`number`）推断出 `Output` 类型参数。

### 约束

我们前面编写的泛型函数可以处理 _任意_ 种类的值。有时我们需要关联两个值，但只能对某一部分特定的值进行操作。在这种情况下，可以使用 _约束_ 来限制类型参数可接受的类型范围。

让我们编写一个返回两个值中较长者的函数。为此，我们需要一个类型为数字的 `length` 属性。我们通过编写 `extends` 子句将类型参数 _约束_ 为该类型：

```ts twoslash
// @errors: 2345 2322
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a
  } else {
    return b
  }
}

// longerArray 的类型是 'number[]'
const longerArray = longest([1, 2], [1, 2, 3])
// longerString 的类型是 'alice' | 'bob'
const longerString = longest('alice', 'bob')
// 报错！数字没有 'length' 属性
const notOK = longest(10, 100)
```

这个例子中有几点值得注意的地方。我们允许 TypeScript _推断_ `longest` 的返回值类型。返回值类型推断同样适用于泛型函数。

因为我们将 `Type` 约束为 `{ length: number }`，所以我们被允许访问参数 `a` 和 `b` 的 `.length` 属性。如果没有类型约束，我们将无法访问这些属性，因为传入的值可能是没有 length 属性的其他类型。

`longerArray` 和 `longerString` 的类型是根据参数推断出来的。记住，泛型的本质就是将两个或多个值与相同的类型关联起来！

最后，正如预期，调用 `longest(10, 100)` 被拒绝了，因为 `number` 类型没有 `.length` 属性。

### 使用受限值

以下是在使用泛型约束时常见的错误：

```ts twoslash
// @errors: 2322
function minimumLength<Type extends { length: number }>(
  obj: Type,
  minimum: number,
): Type {
  if (obj.length >= minimum) {
    return obj
  } else {
    return { length: minimum }
  }
}
```

这个函数看起来似乎没问题：`Type` 被约束为 `{ length: number }`，函数要么返回 `Type`，要么返回匹配该约束的值。问题在于，函数承诺返回与传入参数完全相同类型的对象，而不仅仅是某个匹配约束的对象。如果这段代码是合法的，那你写出的以下代码绝对会报错：

```ts twoslash
declare function minimumLength<Type extends { length: number }>(
  obj: Type,
  minimum: number,
): Type
// ---cut---
// 'arr' 获得了值 { length: 6 }
const arr = minimumLength([1, 2, 3], 6)
// 在此处会崩溃，因为数组有 'slice' 方法，但返回的对象没有！
console.log(arr.slice(0))
```

### 指定类型参数

TypeScript 通常能够推断出泛型调用中预期的类型参数，但并非总是如此。例如，假设你编写了一个合并两个数组的函数：

```ts twoslash
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
  return arr1.concat(arr2)
}
```

通常情况下，使用不匹配的数组调用此函数会报错：

```ts twoslash
// @errors: 2322
declare function combine<Type>(arr1: Type[], arr2: Type[]): Type[]
// ---cut---
const arr = combine([1, 2, 3], ['hello'])
```

不过，如果你确实想这样做，可以手动指定 `Type`：

```ts twoslash
declare function combine<Type>(arr1: Type[], arr2: Type[]): Type[]
// ---cut---
const arr = combine<string | number>([1, 2, 3], ['hello'])
```

### 编写优秀泛型函数的准则

编写泛型函数很有趣，但也容易让人过度使用类型参数。拥有过多类型参数或在不需要的地方使用约束会降低类型推断的成功率，从而让函数调用者感到沮丧。

#### 下放类型参数

这里有两种看起来很相似的函数写法：

```ts twoslash
function firstElement1<Type>(arr: Type[]) {
  return arr[0]
}

function firstElement2<Type extends any[]>(arr: Type) {
  return arr[0]
}

// a: number (good)
const a = firstElement1([1, 2, 3])
// b: any (bad)
const b = firstElement2([1, 2, 3])
```

乍看之下它们似乎完全相同，但 `firstElement1` 是更好的写法。它的推断返回值类型是 `Type`，而 `firstElement2` 的推断返回值类型是 `any`，因为 TypeScript 必须使用约束类型来解析 `arr[0]` 表达式，而不是“等待”在调用时去解析元素。

> **准则**：只要可能，直接使用类型参数，而不是约束它。

#### 使用更少的类型参数

以下是两个相似的函数：

```ts twoslash
function filter1<Type>(arr: Type[], func: (arg: Type) => boolean): Type[] {
  return arr.filter(func)
}

function filter2<Type, Func extends (arg: Type) => boolean>(
  arr: Type[],
  func: Func,
): Type[] {
  return arr.filter(func)
}
```

我们创建了一个 _并不关联两个值_ 的类型参数 `Func`。这通常是一个危险信号，意味着如果调用者想要指定类型参数，就必须无缘无故地手动多传一个。`Func` 除了让函数更难阅读和推论外，没有任何作用！

> **准则**：总是尽可能少地使用类型参数。

#### 类型参数应出现两次

有时我们会忘记函数其实并不需要是泛型：

```ts twoslash
function greet<Str extends string>(s: Str) {
  console.log('Hello, ' + s)
}

greet('world')
```

我们完全可以写一个更简单的版本：

```ts twoslash
function greet(s: string) {
  console.log('Hello, ' + s)
}
```

记住，类型参数的作用是 _关联多个值的类型_。如果一个类型参数在函数签名中只使用了一次，那它没有关联任何东西。这也包括推断出的返回值类型；例如，如果 `Str` 是 `greet` 推断返回值类型的一部分，那么它就关联了参数和返回值类型，因此即使在代码中只写了一次，实际上也使用了 _两次_。

> **准则**：如果一个类型参数只出现了一次，请强烈重新考虑你是否真的需要它。

## 可选参数

JavaScript 函数通常接受可变数量的参数。例如，`number` 的 `toFixed` 方法接受一个可选的小数位数：

```ts twoslash
function f(n: number) {
  console.log(n.toFixed()) // 0 个参数
  console.log(n.toFixed(3)) // 1 个参数
}
```

在 TypeScript 中，我们可以通过添加 `?` 将参数标记为 _可选_ 来建模这一行为：

```ts twoslash
function f(x?: number) {
  // ...
}
f() // OK
f(10) // OK
```

虽然该参数被指定为 `number` 类型，但 `x` 实际上会是 `number | undefined` 类型，因为在 JavaScript 中未指定的参数会获得 `undefined` 值。

你也可以提供参数 _默认值_：

```ts twoslash
function f(x = 10) {
  // ...
}
```

现在在 `f` 的函数体中，`x` 的类型将变为 `number`，因为任何传入的 `undefined` 都会被替换为 `10`。注意，当参数是可选的时，调用者始终可以传入 `undefined`，这只是模拟了“缺少”该参数的情况：

```ts twoslash
declare function f(x?: number): void
// ---cut---
// All OK
f()
f(10)
f(undefined)
```

### 回调函数中的可选参数

在学习了可选参数和函数类型表达式后，编写调用回调函数的函数时很容易犯以下错误：

```ts twoslash
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    callback(arr[i], i)
  }
}
```

人们把 `index?` 写成可选参数时的初衷，往往是希望下面这两种调用都是合法的：

```ts twoslash
// @errors: 2532 18048
declare function myForEach(
  arr: any[],
  callback: (arg: any, index?: number) => void,
): void
// ---cut---
myForEach([1, 2, 3], (a) => console.log(a))
myForEach([1, 2, 3], (a, i) => console.log(a, i))
```

但实际上这表达的意思是：_`callback` 可能会只用一个参数被调用_。换言之，该函数定义表明其实现可能如下所示：

```ts twoslash
// @errors: 2532 18048
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    // 我今天不想传 index 参数
    callback(arr[i])
  }
}
```

因此，TypeScript 会强制执行这一语义，并报出实际上不可能发生的错误：

<!-- prettier-ignore -->
```ts twoslash
// @errors: 2532 18048
declare function myForEach(
  arr: any[],
  callback: (arg: any, index?: number) => void
): void;
// ---cut---
myForEach([1, 2, 3], (a, i) => {
  console.log(i.toFixed());
});
```

在 JavaScript 中，如果调用函数时提供的参数多于其定义的参数，多出的参数会被直接忽略。
TypeScript 的行为也是如此。只要类型一致，参数较少的函数始终可以替代参数较多的函数。

> **准则**：在为回调函数编写函数类型时，_绝不要_ 写可选参数，除非你真的打算在不传入该参数的情况下 _调用_ 它。

## 函数重载

有些 JavaScript 函数支持使用不同数量和类型的参数来调用。例如，你可能会写一个生成 `Date` 的函数，它既可以接受时间戳（一个参数），也可以接受 月/日/年 格式（三个参数）。

在 TypeScript 中，我们可以通过编写 _重载签名_ 来指定一个能以不同方式调用的函数。为此，你需要编写多个函数签名（通常是两个或更多），紧接着是函数体：

```ts twoslash
// @errors: 2575
function makeDate(timestamp: number): Date
function makeDate(m: number, d: number, y: number): Date
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d)
  } else {
    return new Date(mOrTimestamp)
  }
}
const d1 = makeDate(12345678)
const d2 = makeDate(5, 5, 5)
const d3 = makeDate(1, 3)
```

在这个例子中，我们编写了两个重载：一个接受单参数，另一个接受三个参数。前两个签名被称为 _重载签名_。

随后，我们写了一个具有兼容签名的函数实现。函数拥有一个 _实现_ 签名，但这个签名不能被直接调用。尽管我们在必填参数后面写了两个可选参数，但你不能只用两个参数来调用它！

### 重载签名与实现签名

这通常是困惑的源泉。人们经常会写出这样的代码，却不明白为什么会报错：

```ts twoslash
// @errors: 2554
function fn(x: string): void
function fn() {
  // ...
}
// 预期能够在没有参数的情况下调用
fn()
```

强调一下，用于编写函数体的签名从外部是“不可见”的。

> _实现_ 签名在外部是不可见的。编写重载函数时，应该始终在函数实现上方提供 _两个_ 或多个签名。

实现签名还必须与重载签名 _兼容_。例如，以下函数之所以会报错，是因为实现签名未能正确匹配重载签名：

```ts twoslash
// @errors: 2394
function fn(x: boolean): void
// 参数类型不正确
function fn(x: string): void
function fn(x: boolean) {}
```

```ts twoslash
// @errors: 2394
function fn(x: string): string
// 返回值类型不正确
function fn(x: number): boolean
function fn(x: string | number) {
  return 'oops'
}
```

### 编写优秀的重载

与泛型类似，在使用函数重载时你也应遵循一些准则。遵循这些原则能让你的函数更易于调用、理解和实现。

让我们来看一个返回字符串或数组长度的函数：

```ts twoslash
function len(s: string): number
function len(arr: any[]): number
function len(x: any) {
  return x.length
}
```

这个函数本身没问题；我们可以向它传入字符串或数组进行调用。然而，如果传入的值可能是字符串 _或者_ 数组，就无法调用成功，因为 TypeScript 只能将函数调用解析为单个重载：

```ts twoslash
// @errors: 2769
declare function len(s: string): number
declare function len(arr: any[]): number
// ---cut---
len('') // OK
len([0]) // OK
len(Math.random() > 0.5 ? 'hello' : [0])
```

因为这两个重载具有相同的参数数量和相同的返回值类型，我们可以改写一个非重载版本的函数：

```ts twoslash
function len(x: any[] | string) {
  return x.length
}
```

这样好多了！调用者可以使用任一类型的值调用它，并且作为额外的好处，我们无需再去弄清楚正确的实现签名了。

> 只要可能，始终优先使用联合类型参数而不是函数重载。

## 在函数中声明 `this`

TypeScript 会通过代码流分析推断出函数中的 `this` 是什么，例如：

```ts twoslash
const user = {
  id: 123,

  admin: false,
  becomeAdmin: function () {
    this.admin = true
  },
}
```

TypeScript 明白函数 `user.becomeAdmin` 对应的 `this` 是外部对象 `user`。
_这种_ 自动推断已经足以应对很多场景，但在许多情况下你需要对 `this` 所代表的对象有更多控制权。JavaScript 规范指明，你不能拥有名为 `this` 的参数。因此，TypeScript 利用了这一语法空间，让你可以在函数体中声明 `this` 的类型。

```ts twoslash
interface User {
  id: number
  admin: boolean
}
declare const getDB: () => DB
// ---cut---
interface DB {
  filterUsers(filter: (this: User) => boolean): User[]
}

const db = getDB()
const admins = db.filterUsers(function (this: User) {
  return this.admin
})
```

这种模式常见于回调风格的 API，通常由另一个对象来控制函数的调用时机。注意，你需要使用 `function` 关键字而非箭头函数才能实现这种行为：

```ts twoslash
// @errors: 7041 7017
interface User {
  id: number
  admin: boolean
}
declare const getDB: () => DB
// ---cut---
interface DB {
  filterUsers(filter: (this: User) => boolean): User[]
}

const db = getDB()
const admins = db.filterUsers(() => this.admin)
```

## 其他需要了解的类型

在处理函数类型时，有一些经常出现的附加类型你也应该认识。你可以在任何地方使用它们，但在函数的上下文中，它们显得尤为重要。

### `void`

`void` 表示不返回值函数的返回值。只要函数没有 `return` 语句，或者在 return 语句中没有返回任何显式值，它的推断类型就是 `void`：

```ts twoslash
// 推断出的返回值类型为 void
function noop() {
  return
}
```

在 JavaScript 中，不返回任何值的函数会隐式返回 `undefined` 值。然而在 TypeScript 中，`void` 和 `undefined` 是两码事。本章末尾有更详细的解释。

> `void` 和 `undefined` 并不相同。

### `object`

特殊类型 `object` 指的是除基本类型（`string`、`number`、`bigint`、`boolean`、`symbol`、`null` 或 `undefined`）之外的任何值。它有别于 _空对象类型_ `{ }`，也不同于全局类型 `Object`。你很可能永远都不会用到`Object`。

> `object` 不是 `Object`。**务必**使用 `object`！

注意，在 JavaScript 中，函数值就是对象：它们具有属性，原型链中有 `Object.prototype`，属于 `instanceof Object`，你还可以对它们调用 `Object.keys`，等等。因此，函数类型在 TypeScript 中被认为是 `object`。

### `unknown`

`unknown` 类型代表 _任何_ 值。它与 `any` 类型类似，但更为安全，因为对 `unknown` 值做任何操作都是不合法的：

```ts twoslash
// @errors: 2571 18046
function f1(a: any) {
  a.b() // OK
}
function f2(a: unknown) {
  a.b()
}
```

这在描述函数类型时很有用，因为你可以描述一个接受任意值的函数，而无需在函数体内引入 `any` 类型的值。

反之，你也可以描述一个返回 unknown 类型值的函数：

```ts twoslash
declare const someRandomString: string
// ---cut---
function safeParse(s: string): unknown {
  return JSON.parse(s)
}

// 必须谨慎处理 'obj'！
const obj = safeParse(someRandomString)
```

### `never`

某些函数 _永远不会_ 返回值：

```ts twoslash
function fail(msg: string): never {
  throw new Error(msg)
}
```

`never` 类型表示 _从未_ 被观测到的值。在返回值类型中，这意味着该函数抛出了异常，或终止了程序的执行。

当 TypeScript 判断一个联合类型中不再剩余任何选项时，也会出现 `never`。

```ts twoslash
function fn(x: string | number) {
  if (typeof x === 'string') {
    // 处理字符串
  } else if (typeof x === 'number') {
    // 处理数字
  } else {
    x // 此时类型为 'never'！
  }
}
```

### `Function`

全局类型 `Function` 描述了 JavaScript 中所有函数值上存在的诸如 `bind`、`call`、`apply` 等属性。它还有一个特殊属性，即 `Function` 类型的值总是可调用的；这些调用的返回值为 `any`：

```ts twoslash
function doSomething(f: Function) {
  return f(1, 2, 3)
}
```

这是 _无类型的函数调用_，由于其不安全的 `any` 返回值类型，通常最好避免使用。

如果你需要接受任意函数但并不打算调用它，使用 `() => void` 类型通常更安全。

## 剩余参数与展开语法

<blockquote class='bg-reading'>
   <p>背景阅读:<br />
   <a href='https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/rest_parameters'>剩余参数</a><br/>
   <a href='https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Spread_syntax'>展开语法</a><br/>
   </p>
</blockquote>

### 剩余参数

除了使用可选参数或函数重载让函数能接受各种固定数量的参数外，我们还可以使用 _剩余参数_ 来定义接受 _无限制_ 数量参数的函数。

剩余参数出现在所有其他参数之后，并使用 `...` 语法：

```ts twoslash
function multiply(n: number, ...m: number[]) {
  return m.map((x) => n * x)
}
// 'a' 获得了值 [10, 20, 30, 40]
const a = multiply(10, 1, 2, 3, 4)
```

在 TypeScript 中，这类参数的类型注解隐式地为 `any[]` 而非 `any`。任何显式提供的类型注解必须形如 `Array<T>` 或 `T[]`，抑或是元组类型（我们将在稍后学习）。

### 展开语法

反之，我们可以使用展开语法，从可迭代对象（例如数组）中 _提供_ 数量可变的参数。例如，数组的 `push` 方法可接受任意数量的参数：

```ts twoslash
const arr1 = [1, 2, 3]
const arr2 = [4, 5, 6]
arr1.push(...arr2)
```

注意，通常 TypeScript 并不假定数组是不可变的。这会导致一些令人意外的行为：

```ts twoslash
// @errors: 2556
// 推断类型为 number[] —— 即“包含零个或多个数字的数组”，
// 而非特指两个数字
const args = [8, 5]
const angle = Math.atan2(...args)
```

这种情况的最佳修复方案视具体代码而定，但一般来说，使用 `const` 上下文是最直接的解决方法：

```ts twoslash
// 被推断为长度为 2 的元组
const args = [8, 5] as const
// OK
const angle = Math.atan2(...args)
```

如果目标是较旧的运行时环境，使用展开参数可能需要开启 [`downlevelIteration`](/tsconfig#downlevelIteration)。

<!-- TODO link to downlevel iteration -->

## 参数解构

<blockquote class='bg-reading'>
   <p>背景阅读:<br />
   <a href='https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment'>解构</a><br/>
   </p>
</blockquote>

你可以使用参数解构，方便地将作为参数传入的对象解包为一个或多个函数体内的局部变量。在 JavaScript 中，代码如下：

```js
function sum({ a, b, c }) {
  console.log(a + b + c)
}
sum({ a: 10, b: 3, c: 9 })
```

对象的类型注解紧跟在解构语法之后：

```ts twoslash
function sum({ a, b, c }: { a: number; b: number; c: number }) {
  console.log(a + b + c)
}
```

这看起来可能有些冗长，但你也可以在此处使用具名类型：

```ts twoslash
// Same as prior example
type ABC = { a: number; b: number; c: number }
function sum({ a, b, c }: ABC) {
  console.log(a + b + c)
}
```

## 函数的可赋值性

### 返回值类型 `void`

函数的 `void` 返回值类型可能会产生一些不寻常但符合预期的行为。

返回值类型为 `void` 的上下文类型推断，并 _不_ 强制函数 _不_ 返回任何内容。换句话说，具有 `void` 返回值的上下文函数类型（`type voidFunc = () => void`）在实现时，可以返回任意其他值，但该返回值会被忽略。

因此，类型 `() => void` 的以下实现都是合法的：

```ts twoslash
type voidFunc = () => void

const f1: voidFunc = () => {
  return true
}

const f2: voidFunc = () => true

const f3: voidFunc = function () {
  return true
}
```

并且当这些函数之一的返回值赋给另一个变量时，它将保留 void 类型：

```ts twoslash
type voidFunc = () => void

const f1: voidFunc = () => {
  return true
}

const f2: voidFunc = () => true

const f3: voidFunc = function () {
  return true
}
// ---cut---
const v1 = f1()

const v2 = f2()

const v3 = f3()
```

之所以存在这种行为，是为了确保以下代码能够有效运行 —— 尽管 `Array.prototype.push` 返回的是一个数字，而 `Array.prototype.forEach` 方法需要的是一个返回类型为 `void` 的函数。

```ts twoslash
const src = [1, 2, 3]
const dst = [0]

src.forEach((el) => dst.push(el))
```

还有另一种需要注意的特殊情况：如果字面量函数定义明确指定了 `void` 返回值类型，那么该函数绝对 _不能_ 返回任何内容。

```ts twoslash
function f2(): void {
  // @ts-expect-error
  return true
}

const f3 = function (): void {
  // @ts-expect-error
  return true
}
```

关于 `void` 的更多信息，请参考其他相关文档：

- [常见问题 - “为什么返回非 void 的函数能赋值给返回 void 的函数？”](https://github.com/Microsoft/TypeScript/wiki/FAQ#why-are-functions-returning-non-void-assignable-to-function-returning-void)
