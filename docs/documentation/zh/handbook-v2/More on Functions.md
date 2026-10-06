---
title: 深入理解函数
layout: docs
permalink: /zh/docs/handbook/2/functions.html
oneline: '深入了解函数在 TypeScript 中的工作机制。'
---

无论是在本地定义的局部函数、从其他模块导入的函数，还是类中的方法，函数都是构建任何应用程序的基石。
同时，函数本身也是一种值（value）。就像处理其他值一样，TypeScript 提供了多种方式来描述函数的调用方式。
接下来，我们将学习如何编写用于描述函数的类型。

## 函数类型表达式

描述函数最简单的方式就是使用*函数类型表达式*（function type expression）。
这类类型在语法上与箭头函数非常相似：

```ts twoslash
function greeter(fn: (a: string) => void) {
  fn('Hello, World')
}

function printToConsole(s: string) {
  console.log(s)
}

greeter(printToConsole)
```

语法 `(a: string) => void` 表示“一个名为 `a`、类型为 `string` 的单参数函数，且该函数没有返回值”。
与函数声明一样，如果未指定参数类型，它将被隐式推断为 `any`。

> 注意，参数名称是**必须**填写的。函数类型 `(string) => void` 实际上表示的是“一个名为 `string`、类型为 `any` 的参数的函数”！

当然，我们也可以使用类型别名来为函数类型命名：

```ts twoslash
type GreetFunction = (a: string) => void
function greeter(fn: GreetFunction) {
  // ...
}
```

## 调用签名

在 JavaScript 中，函数除了可以被调用之外，还可以拥有自己的属性。
然而，函数类型表达式语法并不支持声明属性。
如果我们想要描述一个既可调用又带有属性的对象，可以在对象类型中编写*调用签名*（call signature）：

```ts twoslash
type DescribableFunction = {
  description: string
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

注意，与函数类型表达式相比，这里的语法略有不同——在参数列表与返回类型之间使用的是 `:`，而不是 `=>`。

## 构造签名

JavaScript 函数还可以通过 `new` 操作符进行调用。
TypeScript 将这些函数称为*构造函数*（constructor），因为它们通常用于创建一个新对象。
通过在调用签名前加上 `new` 关键字，即可编写*构造签名*（construct signature）：

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

某些对象（例如 JavaScript 的 `Date` 对象）既可以在带 `new` 的情况下调用，也可以在不带 `new` 的情况下调用。
你可以在同一个类型中任意组合调用签名与构造签名：

```ts twoslash
interface CallOrConstruct {
  (n?: number): string
  new (s: string): Date
}

function fn(ctor: CallOrConstruct) {
  // 向 `ctor` 传递 `number` 类型的参数会将其与
  // `CallOrConstruct` 接口中的第一个定义相匹配。
  console.log(ctor(10))
  // ^?

  // 同样，向 `ctor` 传递 `string` 类型的参数会将其
  // 与 `CallOrConstruct` 接口中的第二个定义相匹配。
  console.log(new ctor('10'))
  // ^?
}

fn(Date)
```

## 泛型函数

在编写函数时，输入的类型往往与输出的类型存在关联，或者两个输入的类型之间以某种方式相互关联。
让我们先来看一个返回数组首个元素的函数：

```ts twoslash
function firstElement(arr: any[]) {
  return arr[0]
}
```

这个函数可以完成它的工作，但遗憾的是它的返回类型是 `any`。
如果该函数能够返回数组元素的具体类型，那就更好了。

在 TypeScript 中，当我们希望描述两个值之间的对应关系时，就会使用*泛型*（generic）。
具体做法是在函数签名中声明一个*类型参数*（type parameter）：

```ts twoslash
function firstElement<Type>(arr: Type[]): Type | undefined {
  return arr[0]
}
```

通过向该函数添加类型参数 `Type` 并在两处使用它，我们在函数的输入（数组）与输出（返回值）之间建立了关联。
现在调用它时，就能得到一个更具体的类型：

```ts twoslash
declare function firstElement<Type>(arr: Type[]): Type | undefined
// ---cut---
// s 的类型为 'string'
const s = firstElement(['a', 'b', 'c'])
// n 的类型为 'number'
const n = firstElement([1, 2, 3])
// u 的类型为 undefined
const u = firstElement([])
```

### 类型推断

注意，在这个示例中我们并不需要显式指定 `Type`。
该类型是由 TypeScript 自动*推断*（infer）出来的。

我们也可以使用多个类型参数。
例如，一个独立版本的 `map` 函数可以写成这样：

```ts twoslash
// prettier-ignore
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func);
}

// 参数 'n' 的类型为 'string'
// 'parsed' 的类型为 'number[]'
const parsed = map(['1', '2', '3'], (n) => parseInt(n))
```

注意在此示例中，TypeScript 不仅能根据传入的 `string` 数组推断出 `Input` 类型参数的类型，还能根据函数表达式的返回值（`number`）推断出 `Output` 类型参数的类型。

### 约束

前面我们编写的一些泛型函数可以作用于*任何*类型的值。
但有时我们虽然想要关联两个值，却只能对这些值的某个特定子集进行操作。
在这种情况下，我们可以使用*约束*（constraint）来限制类型参数所能接受的类型范围。

我们来编写一个返回两个值中较长者的函数。
为此，我们需要传入的值具有一个数字类型的 `length` 属性。
我们可以通过编写 `extends` 子句来将类型参数*约束*为该类型：

```ts twoslash
// @errors: 2345 2322
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a
  } else {
    return b
  }
}

// longerArray 的类型为 'number[]'
const longerArray = longest([1, 2], [1, 2, 3])
// longerString 的类型为 'alice' | 'bob'
const longerString = longest('alice', 'bob')
// 错误！数字没有 'length' 属性
const notOK = longest(10, 100)
```

这个示例中有几个值得注意的要点。
我们让 TypeScript 自动*推断*了 `longest` 的返回类型。
返回类型推断在泛型函数中同样适用。

因为我们将 `Type` 约束为 `{ length: number }`，所以允许访问参数 `a` 和 `b` 的 `.length` 属性。
如果没有此类型约束，我们将无法访问这些属性，因为传入的值可能是其他没有 `length` 属性的类型。

`longerArray` 和 `longerString` 的类型是根据实参推断出来的。
请记住，泛型的核心目的就是让两个或多个值具有相同类型的关联！

最后，正如我们所期望的那样，对 `longest(10, 100)` 的调用会被拒绝，因为 `number` 类型没有 `.length` 属性。

### 使用受约束的值

在使用泛型约束时，经常会出现以下常见错误：

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

这段代码表面上看起来没有问题——`Type` 被约束为 `{ length: number }`，并且函数要么返回 `Type`，要么返回一个符合该约束的值。
但问题在于，该函数承诺返回的是与传入对象*完全相同类型*的对象，而不仅仅是*某种*符合约束的对象。
如果这段代码合法，你就可以写出必然会引发运行时错误的代码：

```ts twoslash
declare function minimumLength<Type extends { length: number }>(
  obj: Type,
  minimum: number,
): Type
// ---cut---
// 'arr' 获取值 { length: 6 }
const arr = minimumLength([1, 2, 3], 6)
// 并且在这里崩溃，因为数组具有
// 'slice' 方法，但返回的对象没有！
console.log(arr.slice(0))
```

### 指定类型参数

TypeScript 通常能够推断出泛型调用中预期的类型实参，但并非总是如此。
例如，假设你编写了一个用于合并两个数组的函数：

```ts twoslash
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
  return arr1.concat(arr2)
}
```

通常情况下，如果使用类型不匹配的数组调用该函数，将会报错：

```ts twoslash
// @errors: 2322
declare function combine<Type>(arr1: Type[], arr2: Type[]): Type[]
// ---cut---
const arr = combine([1, 2, 3], ['hello'])
```

然而，如果你确实打算这样做，可以手动指定 `Type`：

```ts twoslash
declare function combine<Type>(arr1: Type[], arr2: Type[]): Type[]
// ---cut---
const arr = combine<string | number>([1, 2, 3], ['hello'])
```

### 编写良好泛型函数的准则

编写泛型函数很有趣，但也容易让人过度使用类型参数。
添加过多类型参数或在不必要的地方使用约束，会导致类型推断难以发挥作用，从而给函数的调用方带来困扰。

#### 下推类型参数

下面是两种看似相似的函数编写方式：

```ts twoslash
function firstElement1<Type>(arr: Type[]) {
  return arr[0]
}

function firstElement2<Type extends any[]>(arr: Type) {
  return arr[0]
}

// a: number (推荐)
const a = firstElement1([1, 2, 3])
// b: any (不推荐)
const b = firstElement2([1, 2, 3])
```

乍看之下两者似乎完全相同，但 `firstElement1` 是好得多的写法。
它的推断返回类型是 `Type`，而 `firstElement2` 的推断返回类型却是 `any`，因为 TypeScript 必须使用约束类型来解析 `arr[0]` 表达式，而不是“等待”在函数调用期间去解析具体的元素类型。

> **规则**：尽可能直接使用类型参数本身，而不是对其添加不必要的约束。

#### 使用更少的类型参数

下面是另一对相似的函数：

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

我们创建了一个*并没有关联两个值*的类型参数 `Func`。
这通常是一个危险信号（red flag），因为这意味着如果调用方想要显式指定类型参数，就必须无端多指定一个类型参数。
`Func` 毫无用处，只会让函数更难阅读和推导！

> **规则**：始终尽可能少地使用类型参数。

#### 类型参数应当出现两次

有时我们可能会忘记，某些函数其实根本不需要使用泛型：

```ts twoslash
function greet<Str extends string>(s: Str) {
  console.log('Hello, ' + s)
}

greet('world')
```

我们完全可以写出一个更简洁的版本：

```ts twoslash
function greet(s: string) {
  console.log('Hello, ' + s)
}
```

请记住，类型参数的目的是*关联多个值的类型*。
如果一个类型参数在函数签名中只使用了一次，那它就没有起到任何关联作用。
这同样包括推断的返回类型；例如，如果 `Str` 是 `greet` 推断返回类型的一部分，那么它就关联了实参类型与返回类型，因此即使在书写的代码中只出现了一次，它实际上也被使用了*两次*。

> **规则**：如果一个类型参数只出现在一个位置，请认真重新考虑是否真的需要它。

## 可选参数

JavaScript 中的函数经常接受可变数量的参数。
例如，`number` 的 `toFixed` 方法接受一个可选的小数位数：

```ts twoslash
function f(n: number) {
  console.log(n.toFixed()) // 0 个参数
  console.log(n.toFixed(3)) // 1 个参数
}
```

在 TypeScript 中，我们可以通过使用 `?` 将参数标记为*可选*（optional）来表达这一特性：

```ts twoslash
function f(x?: number) {
  // ...
}
f() // 正常
f(10) // 正常
```

虽然该参数被指定为 `number` 类型，但参数 `x` 的实际类型为 `number | undefined`，因为在 JavaScript 中未传入的参数其值默认为 `undefined`。

你也可以为参数提供一个*默认值*（default）：

```ts twoslash
function f(x = 10) {
  // ...
}
```

此时在 `f` 的函数体内部，`x` 的类型将为 `number`，因为任何传入的 `undefined` 实参都会被替换为 `10`。
需要注意的是，当参数是可选时，调用方始终可以显式传入 `undefined`，因为这只是模拟了“未传实参”的情况：

```ts twoslash
declare function f(x?: number): void
// ---cut---
// 全部正常
f()
f(10)
f(undefined)
```

### 回调函数中的可选参数

在了解了可选参数和函数类型表达式之后，编写调用回调函数的函数时很容易犯下以下错误：

```ts twoslash
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    callback(arr[i], i)
  }
}
```

人们在将 `index?` 写作可选参数时，通常的本意是希望以下两种调用方式都是合法的：

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

但这*实际*所表达的意思是：_`callback` 在被调用时可能只传入一个实参_。
换言之，根据该函数定义，其实现在逻辑上可能是这样的：

```ts twoslash
// @errors: 2532 18048
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    // 今天我不想提供 index 参数
    callback(arr[i])
  }
}
```

反过来，TypeScript 会严格遵循这层含义，并在实际上不可能出错的地方报告错误：

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

在 JavaScript 中，如果调用函数时传入的实参数量多于其形参数量，多余的实参会被直接忽略。
TypeScript 的行为与此一致。
形参数量较少（且类型兼容）的函数始终可以替代形参数量较多的函数。

> **规则**：在为回调函数编写函数类型时，*千万不要*编写可选参数，除非你确实打算在*调用*该函数时不传递对应的实参。

## 函数重载

某些 JavaScript 函数可以在接收不同数量和类型的实参的情况下被调用。
例如，你可能会编写一个生成 `Date` 对象的函数，它既可以接收一个时间戳（1 个参数），也可以接收月/日/年规范（3 个参数）。

在 TypeScript 中，我们可以通过编写*重载签名*（overload signature）来指定一个可以以不同方式调用的函数。
具体做法是先编写若干个函数签名（通常是两个或更多），紧接着编写函数体：

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

在这个示例中，我们编写了两个重载：一个接收一个参数，另一个接收三个参数。
前面的这两个签名被称为*重载签名*。

随后，我们编写了一个具有兼容签名的函数实现。
函数包含一个*实现*签名（implementation signature），但该签名不能被直接调用。
尽管我们在必选参数之后编写了两个带有可选参数的函数实现，但外部并不能只传入两个参数来调用它！

### 重载签名与实现签名

这是一个经常令人困惑的地方。
人们经常会写出如下代码，却不理解为什么会报错：

```ts twoslash
// @errors: 2554
function fn(x: string): void
function fn() {
  // ...
}
// 期望能够以零个参数调用
fn()
```

再次强调，用于编写函数体的实现签名从外部是“不可见”的。

> *实现*签名从外部是不可见的。
> 编写重载函数时，应该始终在函数实现之上定义*两个*或更多的签名。

实现签名还必须与重载签名*兼容*。
例如，以下函数会报错，因为实现签名未能正确匹配重载签名：

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
// 返回类型不正确
function fn(x: number): boolean
function fn(x: string | number) {
  return 'oops'
}
```

### 编写良好重载的准则

与泛型一样，在使用函数重载时也有一些应当遵循的准则。
遵循这些原则能让你的函数更易于调用、更易于理解，也更易于实现。

我们来看一个返回字符串或数组长度的函数：

```ts twoslash
function len(s: string): number
function len(arr: any[]): number
function len(x: any) {
  return x.length
}
```

这个函数本身没有问题；我们可以传入字符串或数组来调用它。
但是，我们无法使用可能为字符串*或*数组的值来调用它，因为 TypeScript 只能将函数调用解析为单个重载：

```ts twoslash
// @errors: 2769
declare function len(s: string): number
declare function len(arr: any[]): number
// ---cut---
len('') // 正常
len([0]) // 正常
len(Math.random() > 0.5 ? 'hello' : [0])
```

由于这两个重载具有相同的参数个数和相同的返回类型，我们可以改写一个非重载版本的函数：

```ts twoslash
function len(x: any[] | string) {
  return x.length
}
```

这要好得多！
调用方可以使用任意一种类型的值来调用该函数；此外还有一个额外的好处，那就是我们不必费心去推敲如何编写正确的实现签名。

> 在可能的情况下，始终优先使用包含联合类型的参数，而不是函数重载。

## 在函数中声明 `this`

TypeScript 会通过控制流分析推断出函数中 `this` 的取值，例如在以下代码中：

```ts twoslash
const user = {
  id: 123,

  admin: false,
  becomeAdmin: function () {
    this.admin = true
  },
}
```

TypeScript 能够理解函数 `user.becomeAdmin` 对应的 `this` 是外部对象 `user`。在很多情况下，这种推断机制已经足够用了，但也有很多场景需要对 `this` 所代表的对象进行更精确的控制。JavaScript 规范规定不能存在名为 `this` 的参数，因此 TypeScript 利用了这一语法空间，允许你在函数体中声明 `this` 的类型。

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

这种模式在回调风格的 API 中很常见，在这类 API 中通常由另一个对象来控制何时调用你的函数。注意，你需要使用 `function` 关键字而不能使用箭头函数来实现这一行为：

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

在处理函数类型时，有一些经常出现的附加类型值得关注。
与所有类型一样，你可以在任何地方使用它们，但这些类型在函数的上下文中尤为重要。

### `void`

`void` 表示不返回任何值的函数的返回值类型。
当一个函数没有任何 `return` 语句，或者 `return` 语句没有返回任何显式值时，它就是该函数推断出的返回类型：

```ts twoslash
// 推断的返回类型为 void
function noop() {
  return
}
```

在 JavaScript 中，不返回任何值的函数会隐式返回 `undefined`。
然而在 TypeScript 中，`void` 和 `undefined` 并不是一回事。
本章末尾将提供更多详细说明。

> `void` 与 `undefined` 并不相同。

### `object`

特殊类型 `object` 指的是任何非原始类型（primitive）的值（即不是 `string`、`number`、`bigint`、`boolean`、`symbol`、`null` 或 `undefined`）。
这不同于*空对象类型* `{ }`，也不同于全局类型 `Object`。
在绝大多数情况下，你可能永远不需要使用 `Object`。

> `object` 不是 `Object`。**始终**使用 `object`！

需要注意的是，在 JavaScript 中，函数值本身就是对象：它们拥有属性、原型链中包含 `Object.prototype`、是 `Object` 的实例（`instanceof Object`）、可以对其调用 `Object.keys` 等等。
因此，在 TypeScript 中，函数类型也被视为 `object`。

### `unknown`

`unknown` 类型表示*任何*值。
这与 `any` 类型相似，但更加安全，因为对 `unknown` 类型的值执行任何操作都是非法的：

```ts twoslash
// @errors: 2571 18046
function f1(a: any) {
  a.b() // 正常
}
function f2(a: unknown) {
  a.b()
}
```

这在描述函数类型时非常有用，因为你可以描述能够接受任何值的函数，而无需在函数体内使用 `any` 类型的值。

反之，你也可以描述一个返回未知类型值的函数：

```ts twoslash
declare const someRandomString: string
// ---cut---
function safeParse(s: string): unknown {
  return JSON.parse(s)
}

// 需要对 'obj' 保持谨慎！
const obj = safeParse(someRandomString)
```

### `never`

有些函数*从不*返回值：

```ts twoslash
function fail(msg: string): never {
  throw new Error(msg)
}
```

`never` 类型表示*永远不会被观察到*的值。
作为返回类型时，它表示该函数会抛出异常或终止程序的执行。

当 TypeScript 判断联合类型中已经没有任何可能的成员时，也会出现 `never`。

```ts twoslash
function fn(x: string | number) {
  if (typeof x === 'string') {
    // 执行某些操作
  } else if (typeof x === 'number') {
    // 执行其他操作
  } else {
    x // 类型为 'never'！
  }
}
```

### `Function`

全局类型 `Function` 描述了存在于 JavaScript 所有函数值上的属性，例如 `bind`、`call`、`apply` 等。
它还有一个特殊性质：`Function` 类型的值始终可以被调用，并且这些调用返回 `any`：

```ts twoslash
function doSomething(f: Function) {
  return f(1, 2, 3)
}
```

这属于*无类型函数调用*（untyped function call），通常应尽量避免，因为其不安全的 `any` 返回类型可能会引入风险。

如果你需要接收一个任意函数但并不打算调用它，使用类型 `() => void` 通常会更加安全。

## 剩余参数与展开参数

<blockquote class='bg-reading'>
   <p>Background Reading:<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters'>Rest Parameters</a><br/>
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax'>Spread Syntax</a><br/>
   </p>
</blockquote>

### 剩余参数

除了使用可选参数或重载来定义能够接收多种固定实参个数的函数外，我们还可以通过*剩余参数*（rest parameters）来定义接收*无限制*数量实参的函数。

剩余参数必须出现在所有其他参数之后，并使用 `...` 语法：

```ts twoslash
function multiply(n: number, ...m: number[]) {
  return m.map((x) => n * x)
}
// 'a' 获取值 [10, 20, 30, 40]
const a = multiply(10, 1, 2, 3, 4)
```

在 TypeScript 中，这些参数上的类型注解隐式为 `any[]` 而不是 `any`；若显式提供类型注解，其格式必须是 `Array<T>`、`T[]` 或元组类型（tuple type，我们将在后续内容中学习）。

### 展开实参

相反地，我们也可以通过展开语法，从可迭代对象（例如数组）中为函数*提供*可变数量的实参。
例如，数组的 `push` 方法可以接收任意数量的实参：

```ts twoslash
const arr1 = [1, 2, 3]
const arr2 = [4, 5, 6]
arr1.push(...arr2)
```

需要注意的是，TypeScript 通常不会假定数组是不可变的（immutable）。
这可能会导致一些出乎意料的行为：

```ts twoslash
// @errors: 2556
// 推断的类型为 number[] —— 即“包含零个或多个数字的数组”，
// 并非具体两个数字
const args = [8, 5]
const angle = Math.atan2(...args)
```

解决这种情况的最佳方式取决于具体代码，但通常来说，使用 `const` 上下文是最直接的解决方案：

```ts twoslash
// 推断为长度为 2 的元组
const args = [8, 5] as const
// OK
const angle = Math.atan2(...args)
```

当目标运行环境为较低版本时，使用展开实参可能需要启用 [`downlevelIteration`](/tsconfig#downlevelIteration)。

<!-- TODO link to downlevel iteration -->

## 参数解构

<blockquote class='bg-reading'>
   <p>Background Reading:<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment'>Destructuring Assignment</a><br/>
   </p>
</blockquote>

你可以使用参数解构，便捷地将实参对象解构为函数体内部的一个或多个局部变量。
在 JavaScript 中，形式如下：

```js
function sum({ a, b, c }) {
  console.log(a + b + c)
}
sum({ a: 10, b: 3, c: 9 })
```

对象的类型注解应写在解构语法之后：

```ts twoslash
function sum({ a, b, c }: { a: number; b: number; c: number }) {
  console.log(a + b + c)
}
```

这种写法可能略显冗长，但你也可以在这里使用命名类型：

```ts twoslash
// 与前面的示例相同
type ABC = { a: number; b: number; c: number }
function sum({ a, b, c }: ABC) {
  console.log(a + b + c)
}
```

## 函数的可赋值性

### 返回类型 `void`

函数的 `void` 返回类型可能会产生一些看似不寻常但符合预期的行为。

返回类型为 `void` 的上下文类型推断（Contextual typing）**并不会**强制函数**不能**返回值。换句话说，当实现一个带有 `void` 返回类型的上下文函数类型（`type voidFunc = () => void`）时，该实现可以返回*任何*值，但该返回值会被忽略。

因此，类型 `() => void` 的以下几种实现都是有效的：

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

并且当这些函数的返回值被赋给另一个变量时，该变量将保留 `void` 类型：

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

之所以存在这种行为，是为了确保以下代码能够合法运行——尽管 `Array.prototype.push` 返回一个数字，而 `Array.prototype.forEach` 方法期望接收一个返回类型为 `void` 的函数。

```ts twoslash
const src = [1, 2, 3]
const dst = [0]

src.forEach((el) => dst.push(el))
```

还有一种特殊情况需要注意：当字面量函数定义显式指定了 `void` 返回类型时，该函数就**不能**返回任何内容。

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

有关 `void` 的更多信息，请参阅以下其他文档：

- [FAQ - "Why are functions returning non-void assignable to function returning void?"](https://github.com/Microsoft/TypeScript/wiki/FAQ#why-are-functions-returning-non-void-assignable-to-function-returning-void)
