---
title: 变量声明
layout: docs
permalink: /zh/docs/handbook/variable-declarations.html
oneline: TypeScript 处理变量声明的方式
translatable: true
---

在 JavaScript 中，`let` 和 `const` 是两个相对较新的变量声明概念。
[正如我们之前提到的](/docs/handbook/basic-types.html#a-note-about-let)，`let` 在某些方面与 `var` 类似，但它能帮助开发者避开在 JavaScript 中经常遇到的一些常见“陷阱”（gotchas）。

`const` 则是对 `let` 的增强，它禁止对变量进行重新赋值。

TypeScript 作为 JavaScript 的超集，自然原生支持 `let` 和 `const`。
本节我们将详细阐述这些新的声明方式，以及为什么它们比 `var` 更值得推荐。

如果你只是随手用过 JavaScript，下一节可以帮你重温相关知识。
如果你已经对 JavaScript 中 `var` 声明的所有怪异行为（quirks）了如指掌，也可以直接跳过该部分。

## `var` 声明

传统上，在 JavaScript 中声明变量一直都是使用 `var` 关键字。

```ts
var a = 10
```

显然，我们刚刚声明了一个名为 `a` 且值为 `10` 的变量。

我们也可以在函数内部声明变量：

```ts
function f() {
  var message = 'Hello, world!'

  return message
}
```

并且还可以在其他函数中访问这些相同的变量：

```ts
function f() {
  var a = 10
  return function g() {
    var b = a + 1
    return b
  }
}

var g = f()
g() // 返回 '11'
```

在上面的例子中，`g` 捕获了在 `f` 中声明的变量 `a`。
无论在何时调用 `g`，`a` 的值都将与 `f` 中的 `a` 关联在一起。
即使 `g` 在 `f` 执行完毕后才被调用，它依然能够访问并修改 `a`。

```ts
function f() {
  var a = 1

  a = 2
  var b = g()
  a = 3

  return b

  function g() {
    return a
  }
}

f() // 返回 '2'
```

### 作用域规则

对于习惯了其他语言的人来说，`var` 声明的作用域规则有些古怪。
来看下面的例子：

```ts
function f(shouldInitialize: boolean) {
  if (shouldInitialize) {
    var x = 10
  }

  return x
}

f(true) // 返回 '10'
f(false) // 返回 'undefined'
```

有些读者看到这个例子可能会感到诧异。
变量 `x` 是在 *`if` 块内部*声明的，但我们却可以在该代码块外部访问它。
这是因为 `var` 声明在其包含它的函数、模块、命名空间或全局作用域中的任何位置都是可访问的（这些概念我们稍后都会讲到），不论它被包含在哪个代码块中。
有人称之为 *`var` 作用域*或*函数作用域*（function-scoping）。
函数参数同样也是函数作用域的。

这些作用域规则可能会引发好几类错误。
其中尤为严重的一个问题是：多次声明同一个变量并不会报错：

```ts
function sumMatrix(matrix: number[][]) {
  var sum = 0
  for (var i = 0; i < matrix.length; i++) {
    var currentRow = matrix[i]
    for (var i = 0; i < currentRow.length; i++) {
      sum += currentRow[i]
    }
  }

  return sum
}
```

也许经验丰富的 JavaScript 开发者很容易看出来，但内层的 `for` 循环会意外覆盖变量 `i`，因为两个 `i` 引用的都是同一个函数作用域变量。
经验丰富的开发者都知道，类似的 bug 很容易逃过代码审查，带来无尽的烦恼。

### 变量捕获的怪异行为

花几秒钟猜猜下面代码片段的输出是什么：

```ts
for (var i = 0; i < 10; i++) {
  setTimeout(function () {
    console.log(i)
  }, 100 * i)
}
```

如果你对此还不熟悉，`setTimeout` 会在指定的毫秒数后尝试执行一个函数（不过需要等待当前正在运行的其他代码结束）。

准备好了吗？来看结果：

```
10
10
10
10
10
10
10
10
10
10
```

很多 JavaScript 开发者对这种行为非常熟悉，但如果你感到意外，也绝不是只有你一个人这样想。
大多数人预期的输出通常是：

```
0
1
2
3
4
5
6
7
8
9
```

还记得我们前面提到的变量捕获吗？
我们传递给 `setTimeout` 的每个函数表达式，实际上引用的都是来自同一个作用域中的同一个 `i`。

让我们花点时间思考一下这意味着什么。
`setTimeout` 会在若干毫秒后运行函数，_但前提是_ `for` 循环已经停止执行；
当 `for` 循环执行结束时，`i` 的值已经是 `10` 了。
因此，每次调用传入的函数时，打印出的都是 `10`！

一种常见的变通解决方案是使用 IIFE（立即调用函数表达式，Immediately Invoked Function Expression）在每次迭代时捕获 `i`：

```ts
for (var i = 0; i < 10; i++) {
  // 通过使用 'i' 的当前值调用函数
  // 来捕获 'i' 的当前状态
  ;(function (i) {
    setTimeout(function () {
      console.log(i)
    }, 100 * i)
  })(i)
}
```

这种看起来有些奇特的写法其实相当常见。
参数列表中的 `i` 实际上遮蔽（shadow）了 `for` 循环中声明的 `i`，但因为我们将它们命名相同，所以不需要对循环体做太多修改。

## `let` 声明

现在你已经了解到 `var` 存在的一些问题，而这正是引入 `let` 语句的原因。
除了使用的关键字不同外，`let` 语句的书写方式与 `var` 语句完全相同。

```ts
let hello = 'Hello!'
```

关键的区别不在于语法，而在于语义，我们现在就来深入探讨这一点。

### 块级作用域

当使用 `let` 声明变量时，它使用的是人们常说的*词法作用域*（lexical-scoping）或*块级作用域*（block-scoping）。
与使用 `var` 声明的变量（其作用域会泄漏到外层函数）不同，块级作用域变量在包含它们的最近代码块或 `for` 循环之外是不可见的。

```ts
function f(input: boolean) {
  let a = 100

  if (input) {
    // 依然可以引用 'a'
    let b = a + 1
    return b
  }

  // 错误：此处不存在 'b'
  return b
}
```

在这里，我们有两个局部变量 `a` 和 `b`。
`a` 的作用域限定在 `f` 的函数体内，而 `b` 的作用域则限定在包含它的 `if` 语句块中。

在 `catch` 子句中声明的变量也具有类似的作用域规则。

```ts
try {
  throw 'oh no!'
} catch (e) {
  console.log('Oh well.')
}

// 错误：此处不存在 'e'
console.log(e)
```

块级作用域变量的另一个特性是：在它们被正式声明之前，不能对其进行读写操作。
尽管这些变量在其整个作用域内都是“存在”的，但在其实际声明之前的所有位置都属于它们的*暂存死区*（temporal dead zone，TDZ）。
这只是一种专业说法，用来表明你不能在 `let` 语句之前访问它们；幸运的是，TypeScript 会明确提示这一点。

```ts
a++ // 在声明之前使用 'a' 是非法的；
let a
```

需要注意的是，你仍然可以在块级作用域变量声明之前*捕获*它。
唯一的限制是不能在声明之前调用那个函数。
如果编译目标是 ES2015，现代运行时环境将会抛出错误；不过目前 TypeScript 比较宽容，不会将其报告为错误。

```ts
function foo() {
  // 可以捕获 'a'
  return a
}

// 在声明 'a' 之前调用 'foo' 是非法的
// 运行时应在此处抛出错误
foo()

let a
```

有关暂存死区的更多信息，请参阅 [Mozilla Developer Network](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/let#Temporal_dead_zone_and_errors_with_let) 上的相关内容。

### 重复声明与变量遮蔽

对于 `var` 声明，我们之前提到过，声明变量多少次都无所谓；最终都只会得到一个变量。

```ts
function f(x) {
  var x
  var x

  if (true) {
    var x
  }
}
```

在上面的示例中，对 `x` 的所有声明实际上都指向*同一个* `x`，这在语法上是完全合法的。
但这种行为往往会成为 bug 的温床。
值得庆幸的是，`let` 声明就不那么宽容了。

```ts
let x = 10
let x = 20 // 错误：不能在同一作用域内重复声明 'x'
```

两个变量并不一定都需要是块级作用域变量，TypeScript 才会报错。

```ts
function f(x) {
  let x = 100 // 错误：与参数声明冲突
}

function g() {
  let x = 100
  var x = 100 // 错误：不能同时存在两个 'x' 的声明
}
```

这并不是说块级作用域变量永远不能与函数作用域变量同名。
块级作用域变量只需要声明在截然不同的代码块中即可。

```ts
function f(condition, x) {
  if (condition) {
    let x = 100
    return x
  }

  return x
}

f(false, 0) // 返回 '0'
f(true, 0) // 返回 '100'
```

在更深层嵌套的作用域中引入新名称的行为称为*变量遮蔽*（shadowing）。
这是一把双刃剑：如果发生意外遮蔽，可能会引发某些 bug；但与此同时，它也能防止某些 bug 的发生。
例如，设想一下我们用 `let` 变量重写之前的 `sumMatrix` 函数。

```ts
function sumMatrix(matrix: number[][]) {
  let sum = 0
  for (let i = 0; i < matrix.length; i++) {
    var currentRow = matrix[i]
    for (let i = 0; i < currentRow.length; i++) {
      sum += currentRow[i]
    }
  }

  return sum
}
```

这个版本的循环实际上能正确计算求和，因为内层循环的 `i` 遮蔽了外层循环的 `i`。

为了编写更清晰的代码，*通常*应该避免变量遮蔽。
虽然在某些场景下利用它可以带来便利，但你应当根据实际情况权衡决断。

### 块级作用域变量捕获

当我们最初探讨 `var` 声明中的变量捕获概念时，曾简要说明了变量被捕获后的行为表现。
为了建立更直观的理解：每次执行一个作用域时，都会创建一个变量“环境”（environment）。
即使该作用域内的所有内容都已执行完毕，该环境及其捕获的变量依然可以继续存在。

```ts
function theCityThatAlwaysSleeps() {
  let getCity

  if (true) {
    let city = 'Seattle'
    getCity = function () {
      return city
    }
  }

  return getCity()
}
```

因为我们从其环境内部捕获了 `city`，所以尽管 `if` 块已经执行完毕，我们依然能够访问它。

回想一下之前的 `setTimeout` 示例，我们最终不得不使用 IIFE 在 `for` 循环的每次迭代中捕获变量的状态。
实际上，我们当时做的是为被捕获的变量创建了一个新的变量环境。
那确实有点麻烦，不过幸运的是，在 TypeScript 中你再也不需要这样做了。

当 `let` 声明作为循环的一部分时，其行为有着截然不同的表现。
这些声明不仅仅是为循环本身引入一个新的环境，而是在*每次迭代*时都会创建一个新的作用域。
既然这正是我们之前通过 IIFE 所做的事，那么我们只需将旧的 `setTimeout` 示例改成使用 `let` 声明即可。

```ts
for (let i = 0; i < 10; i++) {
  setTimeout(function () {
    console.log(i)
  }, 100 * i)
}
```

正如预期的那样，这段代码将打印出：

```
0
1
2
3
4
5
6
7
8
9
```

## `const` 声明

`const` 声明是另一种声明变量的方式。

```ts
const numLivesForCat = 9
```

它们与 `let` 声明类似，但顾名思义，它们的值一旦绑定就不能被修改。
换句话说，它们拥有与 `let` 相同的作用域规则，但不能对它们重新赋值。

这不应与它们所引用的值是*不可变*（immutable）的概念混为一谈。

```ts
const numLivesForCat = 9
const kitty = {
  name: 'Aurora',
  numLives: numLivesForCat,
}

// 错误
kitty = {
  name: 'Danielle',
  numLives: numLivesForCat,
}

// 都“正常”
kitty.name = 'Rory'
kitty.name = 'Kitty'
kitty.name = 'Cat'
kitty.numLives--
```

除非你采取特定手段来避免，否则 `const` 变量的内部状态仍然是可修改的。
幸运的是，TypeScript 允许你将对象的成员指定为 `readonly`。
详情请参阅[接口章节](/docs/handbook/interfaces.html)。

## `let` 与 `const`

既然有两种具有类似作用域语义的声明方式，我们很自然会问到底该用哪一个。
像大多数宽泛的问题一样，答案是：视具体情况而定。

根据[最小特权原则](https://wikipedia.org/wiki/Principle_of_least_privilege)，除了那些你计划修改的变量外，所有声明都应使用 `const`。
其基本理念是：如果一个变量不需要被写入，那么在同一个代码库中协作的其他人就不应该能随意写入该对象，并且需要仔细权衡是否确实需要对该变量重新赋值。
此外，使用 `const` 在推导数据流时也能让代码更具可预测性。

请根据实际情况自行决断；在适用的情况下，也可以与团队其他成员协商确定。

本手册的大部分内容都使用 `let` 声明。

## 解构（Destructuring）

TypeScript 支持的另一个 ECMAScript 2015 特性是解构。
完整的参考信息请参阅 [Mozilla Developer Network 上的文章](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)。
在本节中，我们将对此进行简要概述。

### 数组解构

最简单的解构形式是数组解构赋值：

```ts
let input = [1, 2]
let [first, second] = input
console.log(first) // 输出 1
console.log(second) // 输出 2
```

这创建了两个名为 `first` 和 `second` 的新变量。
这等价于使用索引访问，但要方便得多：

```ts
first = input[0]
second = input[1]
```

解构同样适用于已声明的变量：

```ts
// 交换变量
;[first, second] = [second, first]
```

也适用于函数参数：

```ts
function f([first, second]: [number, number]) {
  console.log(first)
  console.log(second)
}
f([1, 2])
```

你可以使用 `...` 语法为列表中剩余的项创建变量：

```ts
let [first, ...rest] = [1, 2, 3, 4]
console.log(first) // 输出 1
console.log(rest) // 输出 [ 2, 3, 4 ]
```

当然，既然这是 JavaScript，你完全可以忽略后面不关心的尾部元素：

```ts
let [first] = [1, 2, 3, 4]
console.log(first) // 输出 1
```

或者忽略其他位置的元素：

```ts
let [, second, , fourth] = [1, 2, 3, 4]
console.log(second) // 输出 2
console.log(fourth) // 输出 4
```

### 元组解构

元组可以像数组一样被解构；解构出的变量将获得对应元组元素的类型：

```ts
let tuple: [number, string, boolean] = [7, 'hello', true]

let [a, b, c] = tuple // a: number, b: string, c: boolean
```

超出元组元素范围进行解构会导致错误：

```ts
let [a, b, c, d] = tuple // 错误，索引 3 处没有元素
```

与数组一样，你可以使用 `...` 解构元组的剩余部分，从而得到一个更短的元组：

```ts
let [a, ...bc] = tuple // bc: [string, boolean]
let [a, b, c, ...d] = tuple // d: [], 空元组
```

或者忽略尾部元素，或其他位置的元素：

```ts
let [a] = tuple // a: number
let [, b] = tuple // b: string
```

### 对象解构

你也可以解构对象：

```ts
let o = {
  a: 'foo',
  b: 12,
  c: 'bar',
}
let { a, b } = o
```

这将从 `o.a` 和 `o.b` 创建新的变量 `a` 和 `b`。
注意，如果不需要 `c`，你可以直接略过它。

与数组解构一样，你也可以在不声明新变量的情况下进行解构赋值：

```ts
;({ a, b } = { a: 'baz', b: 101 })
```

注意，我们必须用括号将该语句包裹起来。
因为 JavaScript 通常会将 `{` 解析为代码块的开始。

你可以使用 `...` 语法为对象中剩余的属性创建变量：

```ts
let { a, ...passthrough } = o
let total = passthrough.b + passthrough.c.length
```

#### 属性重命名

你还可以为属性赋予不同的名称：

```ts
let { a: newName1, b: newName2 } = o
```

这里的语法开始变得容易令人混淆。
你可以将 `a: newName1` 理解为“将 `a` 重命名为 `newName1`”。
其方向是从左到右，就好像你写成了：

```ts
let newName1 = o.a
let newName2 = o.b
```

容易让人困惑的是，这里的冒号并*不*表示类型。
如果指定类型，仍然需要写在整个解构的后面：

```ts
let { a: newName1, b: newName2 }: { a: string; b: number } = o
```

#### 默认值

默认值允许你在属性为 `undefined` 时指定一个默认值：

```ts
function keepWholeObject(wholeObject: { a: string; b?: number }) {
  let { a, b = 1001 } = wholeObject
}
```

在这个示例中，`b?` 表示 `b` 是可选的，因此它可能是 `undefined`。
现在 `keepWholeObject` 拥有变量 `wholeObject` 以及属性 `a` 和 `b`，即使 `b` 是 `undefined` 也是如此。

## 函数声明

解构同样适用于函数声明。
对于简单的场景，这非常直观：

```ts
type C = { a: string; b?: number }
function f({ a, b }: C): void {
  // ...
}
```

但在参数中指定默认值更为常见，而在解构中正确设置默认值可能会有些棘手。
首先，你需要记住将解构模式放在默认值之前：

```ts
function f({ a = '', b = 0 } = {}): void {
  // ...
}
f()
```

> 上面的代码片段是类型推断的一个示例，本手册前面已做过说明。

其次，你需要记住为可选属性在解构属性上指定默认值，而不是在主初始化表达式中指定。
请记住，`C` 定义中的 `b` 是可选的：

```ts
function f({ a, b = 0 } = { a: '' }): void {
  // ...
}
f({ a: 'yes' }) // 正常，默认 b = 0
f() // 正常，默认为 { a: "" }，进而默认 b = 0
f({}) // 错误，如果提供了参数，则 'a' 是必填的
```

请谨慎使用解构。
如上面的示例所示，除了最简单的解构表达式外，其他任何形式都容易令人困惑。
对于深层嵌套的解构更是如此，即使不加上重命名、默认值和类型注解，它也*极度*难以理解。
尽量保持解构表达式简短明了。
你完全可以亲手编写解构本会生成的赋值语句。

## 展开运算符（Spread）

展开运算符与解构相反。
它允许你将一个数组展开到另一个数组中，或者将一个对象展开到另一个对象中。
例如：

```ts
let first = [1, 2]
let second = [3, 4]
let bothPlus = [0, ...first, ...second, 5]
```

这使得 `bothPlus` 的值为 `[0, 1, 2, 3, 4, 5]`。
展开操作会创建 `first` 和 `second` 的浅拷贝。
它们本身不会被展开操作改变。

你也可以展开对象：

```ts
let defaults = { food: 'spicy', price: '$$', ambiance: 'noisy' }
let search = { ...defaults, food: 'rich' }
```

现在 `search` 的值为 `{ food: "rich", price: "$$", ambiance: "noisy" }`。
对象展开比数组展开更复杂。
与数组展开一样，它是从左到右进行的，但结果仍然是一个对象。
这意味着在展开对象中靠后的属性会覆盖靠前的属性。
因此，如果我们将前面的示例修改为在末尾展开：

```ts
let defaults = { food: 'spicy', price: '$$', ambiance: 'noisy' }
let search = { food: 'rich', ...defaults }
```

那么 `defaults` 中的 `food` 属性就会覆盖 `food: "rich"`，在当前场景下这并不是我们想要的结果。

对象展开还有其他几个令人意外的限制。
首先，它仅包含对象的[自身可枚举属性](https://developer.mozilla.org/docs/Web/JavaScript/Enumerability_and_ownership_of_properties)。
基本上，这意味着当你展开一个对象的实例时，会丢失其方法：

```ts
class C {
  p = 12
  m() {}
}
let c = new C()
let clone = { ...c }
clone.p // 正常
clone.m() // 错误！
```

其次，TypeScript 编译器不允许展开泛型函数中的类型参数。
该特性预计将在未来的语言版本中提供。

## `using` 声明

`using` 声明是 JavaScript 中即将推出的新特性，属于 [Stage 3 显式资源管理（Explicit Resource Management）](https://github.com/tc39/proposal-explicit-resource-management)提案的一部分。
`using` 声明与 `const` 声明非常相似，区别在于它将绑定到声明的值的*生命周期*与变量的*作用域*关联在一起。

当控制流离开包含 `using` 声明的代码块时，声明的值的 `[Symbol.dispose]()` 方法会被执行，从而允许该值执行清理操作：

```ts
function f() {
  using x = new C()
  doSomethingWith(x)
} // 调用了 `x[Symbol.dispose]()`
```

在运行时，这*大致*等价于以下代码的效果：

```ts
function f() {
  const x = new C()
  try {
    doSomethingWith(x)
  } finally {
    x[Symbol.dispose]()
  }
}
```

在处理持有文件句柄等原生引用的 JavaScript 对象时，`using` 声明在避免内存泄漏方面非常有用：

```ts
{
  using file = await openFile()
  file.write(text)
  doSomethingThatMayThrow()
} // 即使抛出错误，`file` 也会被释放
```

或者用于追踪（tracing）等有界操作：

```ts
function f() {
  using activity = new TraceActivity('f') // 追踪进入函数
  // ...
} // 追踪退出函数
```

与 `var`、`let` 和 `const` 不同，`using` 声明不支持解构。

### `null` 和 `undefined`

需要注意的是，该值可以是 `null` 或 `undefined`，在这种情况下，在代码块结束时不会执行任何释放操作：

```ts
{
  using x = b ? new C() : null
  // ...
}
```

这*大致*等价于：

```ts
{
  const x = b ? new C() : null
  try {
    // ...
  } finally {
    x?.[Symbol.dispose]()
  }
}
```

这允许你在使用 `using` 声明时有条件地获取资源，而无需进行复杂的分支或重复操作。

### 定义可释放资源

你可以通过实现 `Disposable` 接口来表明你生成的类或对象是可释放的：

```ts
// 来自默认库：
interface Disposable {
  [Symbol.dispose](): void
}

// 用法：
class TraceActivity implements Disposable {
  readonly name: string
  constructor(name: string) {
    this.name = name
    console.log(`Entering: ${name}`)
  }

  [Symbol.dispose](): void {
    console.log(`Exiting: ${name}`)
  }
}

function f() {
  using _activity = new TraceActivity('f')
  console.log('Hello world!')
}

f()
// 打印：
//   Entering: f
//   Hello world!
//   Exiting: f
```

## `await using` 声明

某些资源或操作的清理工作可能需要以异步方式执行。为了满足这一需求，[显式资源管理（Explicit Resource Management）](https://github.com/tc39/proposal-explicit-resource-management)提案还引入了 `await using` 声明：

```ts
async function f() {
  await using x = new C()
} // 调用了 `await x[Symbol.asyncDispose]()`
```

当控制流离开包含 `await using` 声明的代码块时，它会调用并 _await_ 其值的 `[Symbol.asyncDispose]()` 方法。这支持异步清理操作，例如数据库事务执行回滚或提交，或者文件流在关闭前将所有待写入数据刷新（flush）到存储中。

与 `await` 一样，`await using` 只能在 `async` 函数或方法内部，或者在模块顶层使用。

### 定义异步可释放资源

正如 `using` 依赖于实现 `Disposable` 的对象一样，`await using` 依赖于实现 `AsyncDisposable` 的对象：

```ts
// 来自默认库：
interface AsyncDisposable {
  [Symbol.asyncDispose]: PromiseLike<void>
}

// 用法：
class DatabaseTransaction implements AsyncDisposable {
  public success = false
  private db: Database | undefined

  private constructor(db: Database) {
    this.db = db
  }

  static async create(db: Database) {
    await db.execAsync('BEGIN TRANSACTION')
    return new DatabaseTransaction(db)
  }

  async [Symbol.asyncDispose]() {
    if (this.db) {
      const db = this.db
      this.db = undefined
      if (this.success) {
        await db.execAsync('COMMIT TRANSACTION')
      } else {
        await db.execAsync('ROLLBACK TRANSACTION')
      }
    }
  }
}

async function transfer(
  db: Database,
  account1: Account,
  account2: Account,
  amount: number,
) {
  await using tx = await DatabaseTransaction.create(db)
  if (await debitAccount(db, account1, amount)) {
    await creditAccount(db, account2, amount)
  }
  // 如果在此行之前抛出异常，事务将回滚
  tx.success = true
  // 现在事务将被提交
}
```

### `await using` 与 `await`

`await using` 声明中的 `await` 关键字仅表示资源的*释放*会被 `await`。它并*不*代表 `await` 该值本身：

```ts
{
  await using x = getResourceSynchronously()
} // 执行 `await x[Symbol.asyncDispose]()`

{
  await using y = await getResourceAsynchronously()
} // 执行 `await y[Symbol.asyncDispose]()`
```

### `await using` 与 `return`

需要特别注意的是，如果你在返回 `Promise` 的 `async` 函数中使用 `await using` 声明，但返回时没有先对其进行 `await`，就会存在一个小小的注意事项：

```ts
function g() {
  return Promise.reject('error!')
}

async function f() {
  await using x = new C()
  return g() // 缺少 `await`
}
```

因为返回的 Promise 没有被 `await`，当执行暂停以 `await` 异步释放 `x` 时，由于尚未订阅返回的 Promise，JavaScript 运行时可能会报告未处理的拒绝（unhandled rejection）。不过这并不是 `await using` 独有的问题，在使用 `try..finally` 的 `async` 函数中同样可能发生这种情况：

```ts
async function f() {
  try {
    return g() // 同样会报告未处理的 rejection
  } finally {
    await somethingElse()
  }
}
```

为了避免这种情况，如果返回值可能是一个 `Promise`，建议对其进行 `await`：

```ts
async function f() {
  await using x = new C()
  return await g()
}
```

## 在 `for` 和 `for..of` 语句中使用 `using` 与 `await using`

`using` 和 `await using` 都可以用于 `for` 语句中：

```ts
for (using x = getReader(); !x.eof; x.next()) {
  // ...
}
```

在这种情况下，`x` 的生命周期限定在整个 `for` 语句范围内，并且仅在控制流因 `break`、`return`、`throw` 或循环条件为 false 而离开循环时才会被释放。

除了 `for` 语句外，这两种声明也可以用于 `for..of` 语句：

```ts
function* g() {
  yield createResource1()
  yield createResource2()
}

for (using x of g()) {
  // ...
}
```

在这里，`x` 会在*循环的每次迭代*结束时被释放，然后用下一个值重新初始化。当依次消费生成器逐个产出的资源时，这一点尤其有用。

## 在旧版运行时中使用 `using` 和 `await using`

当以较旧的 ECMAScript 版本为编译目标时，只要使用了兼容的 `Symbol.dispose`/`Symbol.asyncDispose` polyfill（例如最近版本的 NodeJS 默认提供的 polyfill），就可以使用 `using` 和 `await using` 声明。
