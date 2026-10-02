---
title: 常见类型
layout: docs
permalink: /zh/docs/handbook/2/everyday-types.html
oneline: 'The language primitives.'
---

在本章中，我们将介绍一些在 JavaScript 代码中最常见的值的类型，并说明在 TypeScript 中描述这些类型相应的方法。
这不是一个详尽的列表，后续章节将描述命名和使用其他类型的更多方法。

类型还可以出现在许多 _地方_ ，而不仅仅是类型注释。
在我们了解类型本身的同时，我们还将了解在哪些地方可以引用这些类型来形成新的结构。

我们将首先回顾一下你在编写 JavaScript 或 TypeScript 代码时可能遇到的最基本和最常见的类型。
这些将在稍后形成更复杂类型的核心构建块。

## 基本类型：`string`，`number`，和 `boolean`

JavaScript 有三个非常常用的 [基本类型](https://developer.mozilla.org/zh-CN/docs/Glossary/Primitive)：`string`、`number` 和 `boolean`。
它们在 TypeScript 中都有对应的类型。正如你预期的那样，如果对这些类型的值使用 JavaScript 的 `typeof` 运算符，你会看到相同的名称：

- `string` 表示字符串值，如 `"Hello, world"`。
- `number` 表示数字，如 `42`。JavaScript 没有针对整数的特殊运行时值，因此没有 `int` 或 `float` 的等价物 —— 一切都只是 `number`。
- `boolean` 表示 `true` 和 `false` 这两个值。

> 类型名称 `String`、`Number` 和 `Boolean`（以大写字母开头）是合法的，但它们指的是一些极少出现在代码中的特殊内置类型。_始终_ 使用 `string`、`number` 或 `boolean` 作为类型。

## 数组

要指定像 `[1, 2, 3]` 这样的数组类型，可以使用 `number[]` 语法；此语法适用于任何类型（例如，`string[]` 表示字符串数组，依此类推）。你可能也会看到它被写成 `Array<number>`，含义相同。我们将在介绍 _泛型_ 时，进一步学习 `T<U>` 语法。

注意 `[number]` 是完全不同的概念；请参考 _元组类型 (tuple types)_ 部分。

## `any`

TypeScript 还有一个特殊类型 `any`。当你不想让某个特定值引发类型检查错误时，可以使用它。

当一个值的类型为 `any` 时，你可以访问它的任何属性（返回的类型也将是 `any`），可以像函数一样调用它，将它赋值给任何类型的值（或反向赋值），几乎可以进行任何语法上合法的操作：

```ts twoslash
let obj: any = { x: 0 }
// 下面的代码都不会抛出编译器错误。
// 使用 `any` 会禁用所有后续的类型检查，编译器会假定你比 TypeScript 更了解当前环境。
obj.foo()
obj()
obj.bar = 100
obj = 'hello'
const n: number = obj
```

当你不想仅仅为了让 TypeScript 接受某行代码而编写冗长的类型时，`any` 类型非常有用。

### `noImplicitAny`

如果没有指定类型，且 TypeScript 无法从上下文中推断出来，编译器通常会默认使用 `any` 类型。

但通常应避免这种情况，因为 `any` 不会进行类型检查。使用编译器标志 [`noImplicitAny`](/tsconfig#noImplicitAny) 可以将任何隐式的 `any` 标记为错误。

## 变量上的类型注解

使用 `const`、`var` 或 `let` 声明变量时，可以选择添加类型注解，以显式指定变量的类型：

```ts twoslash
let myName: string = 'Alice'
//        ^^^^^^^^ 类型注解
```

> TypeScript 不使用类似 `int x = 0;` 这种“类型在左边”的声明方式。
> 类型注解始终放在被限定内容的 _后面_。

不过在大多数情况下，并不需要这样做。TypeScript 会尽可能自动 _推断_ 代码中的类型。例如，变量的类型会根据其初始化器推断得出：

```ts twoslash
// 不需要类型注解 —— 'myName' 被推断为 'string' 类型
let myName = 'Alice'
```

通常不需要刻意学习这些推断规则。如果是初学者，可以尝试尽量少用类型注解 —— 你会惊讶地发现，只需极少的注解，TypeScript 就能完全理解代码意图。

## 函数

函数是 JavaScript 中传递数据的主要方式。TypeScript 允许指定函数的输入和输出值的类型。

### 参数类型注解

声明函数时，可以在每个参数后添加类型注解，以声明函数接受的参数类型。参数类型注解位于参数名称之后：

```ts twoslash
// 参数类型注解
function greet(name: string) {
  //                 ^^^^^^^^
  console.log('Hello, ' + name.toUpperCase() + '!!')
}
```

只要参数有类型注解，TypeScript 就会检查传递给该函数的实参：

```ts twoslash
// @errors: 2345
declare function greet(name: string): void
// ---cut---
// 如果执行，这里会产生运行时错误！
greet(42)
```

> 即便没有在参数上添加类型注解，TypeScript 依然会检查传入的实参个数是否正确。

### 返回类型注解

你也可以添加返回类型注解。
返回类型注解出现在参数列表之后：

```ts twoslash
function getFavoriteNumber(): number {
  //                        ^^^^^^^^
  return 26
}
```

与变量类型注解类似，通常不需要返回类型注解，因为 TypeScript 会根据 `return` 语句推断函数的返回类型。
上面示例中的类型注解并不会改变任何事情。有些代码库为了编写文档、防止意外修改，或者纯粹出于个人偏好，会显式指定返回类型。

### 匿名函数

匿名函数与函数声明略有不同。当函数出现在 TypeScript 可以确定其调用方式的地方时，该函数的参数会自动被赋予类型。

例如：

```ts twoslash
// @errors: 2551
// 这里没有类型注解，但 TypeScript 能发现 bug
const names = ['Alice', 'Bob', 'Eve']

// 函数的上下文类型
names.forEach(function (s) {
  console.log(s.toUppercase())
})

// 上下文类型同样适用于箭头函数
names.forEach((s) => {
  console.log(s.toUppercase())
})
```

尽管参数 `s` 没有类型注解，TypeScript 还是利用 `forEach` 函数的类型以及数组的推断类型，确定了 `s` 的类型。

这个过程称为 _上下文类型推断_ (contextual typing)，因为函数所处的 _上下文 (context)_ 决定了它应有的类型。与前面的推断规则一样，你不需要专门学习其背后的机制，但只要知道它的 _存在_，就能帮你判断何时可以省略类型注解。稍后，我们会看到更多关于值所处上下文如何影响其类型的示例。

## 对象类型

除了基本类型，最常见的类型就是 _对象类型_。这指的是任何带有属性的 JavaScript 值（几乎涵盖了所有值！）。定义对象类型只需列出其属性及对应类型。

例如，下面这个函数接受一个类似坐标点的对象：

```ts twoslash
// 参数的类型注解是一个对象类型
function printCoord(pt: { x: number; y: number }) {
  //                      ^^^^^^^^^^^^^^^^^^^^^^^^
  console.log("The coordinate's x value is " + pt.x)
  console.log("The coordinate's y value is " + pt.y)
}
printCoord({ x: 3, y: 7 })
```

此处，我们用一个包含 `x` 和 `y` 两个 `number` 类型属性的类型对参数进行了注解。你可以使用 `,` 或 `;` 来分隔属性，且最后一个属性后面的分隔符是可选的。

每个属性的类型部分也是可选的。如果不指定类型，默认会被视为 `any`。

### 可选属性

对象类型还可以指定部分或全部属性为 _可选的_。为此，只需在属性名后加上 `?`：

```ts twoslash
function printName(obj: { first: string; last?: string }) {
  // ...
}
// 都可以
printName({ first: 'Bob' })
printName({ first: 'Alice', last: 'Alisson' })
```

在 JavaScript 中，如果访问一个不存在的属性，会得到 `undefined` 而不是运行时错误。因此，在 _读取_ 可选属性时，必须先检查它是否为 `undefined` 然后再使用。

```ts twoslash
// @errors: 2532
function printName(obj: { first: string; last?: string }) {
  // 错误 - 如果没有提供 'obj.last'，可能会崩溃！
  console.log(obj.last.toUpperCase())
  if (obj.last !== undefined) {
    // OK
    console.log(obj.last.toUpperCase())
  }

  // 使用现代 JavaScript 语法的一种安全替代方案：
  console.log(obj.last?.toUpperCase())
}
```

## 联合类型

TypeScript 的类型系统允许你使用各种运算符，基于现有类型构建新类型。了解如何编写基本类型后，接下来学习如何以有趣的方式将它们 _组合_ 起来。

### 定义联合类型

最先接触的类型组合方式是 _联合_ 类型。联合类型由两个或多个其他类型组合而成，表示可能是这些类型中 _任意一个_ 的值。我们将这些基础类型称为联合类型的 _成员_。

我们来写一个可以操作字符串或数字的函数：

```ts twoslash
// @errors: 2345
function printId(id: number | string) {
  console.log('Your ID is: ' + id)
}
// OK
printId(101)
// OK
printId('202')
// Error
printId({ myID: 22342 })
```

### 使用联合类型

_提供_ 一个符合联合类型的值很简单 —— 只需提供匹配任意成员类型的值即可。但如果 _拥有_ 一个联合类型的值，应该如何使用它呢？

TypeScript 仅允许执行对联合类型中 _所有_ 成员都有效的操作。例如，对于 `string` | `number` 联合类型，不能使用仅 `string` 独有的方法：

```ts twoslash
// @errors: 2339
function printId(id: number | string) {
  console.log(id.toUpperCase())
}
```

解决方法是在代码中 _收窄_ 联合类型，就像在没有类型注解的 JavaScript 中一样。当 TypeScript 能够根据代码结构推导出该值具有更具体的类型时，就会发生 _收窄_。

例如，TypeScript 知道只有 `string` 类型的值，其 `typeof` 才会返回 `"string"`：

```ts twoslash
function printId(id: number | string) {
  if (typeof id === 'string') {
    // 在这个分支中，id 是 'string' 类型
    console.log(id.toUpperCase())
  } else {
    // 这里，id 是 'number' 类型
    console.log(id)
  }
}
```

另一个例子是使用像 `Array.isArray` 这样的函数：

```ts twoslash
function welcomePeople(x: string[] | string) {
  if (Array.isArray(x)) {
    // 这里：'x' 是 'string[]'
    console.log('Hello, ' + x.join(' and '))
  } else {
    // 这里：'x' 是 'string'
    console.log('Welcome lone traveler ' + x)
  }
}
```

注意在 `else` 分支中不需要做特殊处理 —— 如果 `x` 不是 `string[]`，那它必然是 `string`。

有时联合类型的所有成员会存在共性。例如，数组和字符串都有 `slice` 方法。如果联合类型的每个成员都有某个公共属性，则无需收窄就可以直接使用该属性：

```ts twoslash
// 返回类型被推断为 number[] | string
function getFirstThree(x: number[] | string) {
  return x.slice(0, 3)
}
```

> 类型的 _联合_ 似乎拥有这些类型属性的 _交集_，这可能让人有些困惑。
> 这并非偶然 —— 名称 _联合_ 来源于类型理论。
> _联合类型_ `number | string` 是由每种类型值的并集组成的。
> 请注意，假设有两个集合以及各自具备的一些特征，只有这些特征的 _交集_ 才必定适用于这两个集合 _合并_ 后的并集。
> 例如，如果一个房间全是戴帽子的高个子，另一个房间全是戴帽子的说西班牙语的人，把这两个房间的人合并后，关于 _每_ 个人我们唯一能确定的是：他们肯定都戴着帽子。

## 类型别名

我们通过直接在类型注解中编写对象类型和联合类型来使用它们。这很方便，但是常常会想要多次使用同一个类型，并且通过一个名称引用它。

_类型别名_ 正是如此 —— 任意 _类型_ 的一个 _名称_ 。类型别名的语法是：

```ts twoslash
type Point = {
  x: number
  y: number
}

// 与前面的示例完全相同
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x)
  console.log("The coordinate's y value is " + pt.y)
}

printCoord({ x: 100, y: 100 })
```

实际上，不只是对象类型，你可以使用类型别名为任何类型命名。
例如，类型别名可以命名联合类型：

```ts twoslash
type ID = number | string
```

请注意，别名 _只是_ 别名 - 你不能使用类型别名创建同一类型的不同“版本”。
当你使用别名时，它与您编写的别名类型完全一样。
换句话说，这段代码 _看起来_ 可能是非法的，但是对于 TypeScript 来说是正确的，因为这两种类型都是同一类型的别名：

```ts twoslash
declare function getInput(): string
declare function sanitize(str: string): string
// ---分割---
type UserInputSanitizedString = string

function sanitizeInput(str: string): UserInputSanitizedString {
  return sanitize(str)
}

// 创建一个经过清理的输入框
let userInput = sanitizeInput(getInput())

// 仍然可以使用字符串重新赋值
userInput = 'new input'
```

## 接口

_接口声明_ 是命名对象类型的另一种方式：

```ts twoslash
interface Point {
  x: number
  y: number
}

function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x)
  console.log("The coordinate's y value is " + pt.y)
}

printCoord({ x: 100, y: 100 })
```

就像我们上面使用类型别名时一样，这个示例的工作方式就像我们使用了匿名对象类型一样。
TypeScript 只关心我们传递给 `printCoord` 的值的结构 - 它只关心它是否具有预期的属性。
只关心类型的结构和功能，这就是为什么我们说 TypeScript 是一个 _结构化类型_ 的类型系统。

### 类型别名和接口之间的区别

类型别名和接口非常相似，在大多数情况下你可以在它们之间自由选择。
几乎所有的 `interface` 功能都可以在 `type` 中使用，关键区别在于不能重新开放类型以添加新的属性，而接口始终是可扩展的。

<table class='full-width-table'>
  <tbody>
    <tr>
      <th><code>Interface</code></th>
      <th><code>Type</code></th>
    </tr>
    <tr>
      <td>
        <p>扩展接口</p>
        <code><pre>
interface Animal {
  name: string
}<br/>
interface Bear extends Animal {
  honey: boolean
}<br/>
const bear = getBear() 
bear.name
bear.honey
        </pre></code>
      </td>
      <td>
        <p>通过 "&" 扩展类型</p>
        <code><pre>
type Animal = {
  name: string
}<br/>
type Bear = Animal & { 
  honey: Boolean 
}<br/>
const bear = getBear();
bear.name;
bear.honey;
        </pre></code>
      </td>
    </tr>
    <tr>
      <td>
        <p>向现有接口添加新字段</p>
        <code><pre>
interface Window {
  title: string
}<br/>
interface Window {
  ts: TypeScriptAPI
}<br/>
const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
        </pre></code>
      </td>
      <td>
        <p>类型创建后不能更改</p>
        <code><pre>
type Window = {
  title: string
}<br/>
type Window = {
  ts: TypeScriptAPI
}<br/>
<span style="color: #A31515"> // Error: Duplicate identifier 'Window'.</span><br/>
        </pre></code>
      </td>
    </tr>
    </tbody>
</table>

在后面的章节中你会学到更多关于这些概念的知识，所以如果你没有立即理解这些知识，请不要担心。

- 在 TypeScript 4.2 之前，类型别名命名 [_可能_ 会出现在错误消息中](/play?#code/PTAEGEHsFsAcEsA2BTATqNrLusgzngIYDm+oA7koqIYuYQJ56gCueyoAUCKAC4AWHAHaFcoSADMaQ0PCG80EwgGNkALk6c5C1EtWgAsqOi1QAb06groEbjWg8vVHOKcAvpokshy3vEgyyMr8kEbQJogAFND2YREAlOaW1soBeJAoAHSIkMTRmbbI8e6aPMiZxJmgACqCGKhY6ABGyDnkFFQ0dIzMbBwCwqIccabcYLyQoKjIEmh8kwN8DLAc5PzwwbLMyAAeK77IACYaQSEjUWZWhfYAjABMAMwALA+gbsVjoADqgjKESytQPxCHghAByXigYgBfr8LAsYj8aQMUASbDQcRSExCeCwFiIQh+AKfAYyBiQFgOPyIaikSGLQo0Zj-aazaY+dSaXjLDgAGXgAC9CKhDqAALxJaw2Ib2RzOISuDycLw+ImBYKQflCkWRRD2LXCw6JCxS1JCdJZHJ5RAFIbFJU8ADKC3WzEcnVZaGYE1ABpFnFOmsFhsil2uoHuzwArO9SmAAEIsSFrZB-GgAjjA5gtVN8VCEc1o1C4Q4AGlR2AwO1EsBQoAAbvB-gJ4HhPgB5aDwem-Ph1TCV3AEEirTp4ELtRbTPD4vwKjOfAuioSQHuDXBcnmgACC+eCONFEs73YAPGGZVT5cRyyhiHh7AAON7lsG3vBggB8XGV3l8-nVISOgghxoLq9i7io-AHsayRWGaFrlFauq2rg9qaIGQHwCBqChtKdgRo8TxRjeyB3o+7xAA)，有时代替等效的匿名类型（可能需要也可能不需要）。接口在错误消息中将始终被命名。
- 类型别名不能参与 [声明合并，但接口可以](/play?#code/PTAEEEDtQS0gXApgJwGYEMDGjSfdAIx2UQFoB7AB0UkQBMAoEUfO0Wgd1ADd0AbAK6IAzizp16ALgYM4SNFhwBZdAFtV-UAG8GoPaADmNAcMmhh8ZHAMMAvjLkoM2UCvWad+0ARL0A-GYWVpA29gyY5JAWLJAwGnxmbvGgALzauvpGkCZmAEQAjABMAMwALLkANBl6zABi6DB8okR4Jjg+iPSgABboovDk3jjo5pbW1d6+dGb5djLwAJ7UoABKiJTwjThpnpnGpqPBoTLMAJrkArj4kOTwYmycPOhW6AR8IrDQ8N04wmo4HHQCwYi2Waw2W1S6S8HX8gTGITsQA)。
- 接口只能用于 [声明对象的形状，不能重命名基本类型](/play?#code/PTAEAkFMCdIcgM6gC4HcD2pIA8CGBbABwBtIl0AzUAKBFAFcEBLAOwHMUBPQs0XFgCahWyGBVwBjMrTDJMAshOhMARpD4tQ6FQCtIE5DWoixk9QEEWAeV37kARlABvaqDegAbrmL1IALlAEZGV2agBfampkbgtrWwMAJlAAXmdXdy8ff0Dg1jZwyLoAVWZ2Lh5QVHUJflAlSFxROsY5fFAWAmk6CnRoLGwmILzQQmV8JmQmDzI-SOiKgGV+CaYAL0gBBdyy1KCQ-Pn1AFFplgA5enw1PtSWS+vCsAAVAAtB4QQWOEMKBuYVUiVCYvYQsUTQcRSBDGMGmKSgAAa-VEgiQe2GLgKQA).
- 接口名称将 [_始终_ 以其原始形式出现](/play?#code/PTAEGEHsFsAcEsA2BTATqNrLusgzngIYDm+oA7koqIYuYQJ56gCueyoAUCKAC4AWHAHaFcoSADMaQ0PCG80EwgGNkALk6c5C1EtWgAsqOi1QAb06groEbjWg8vVHOKcAvpokshy3vEgyyMr8kEbQJogAFND2YREAlOaW1soBeJAoAHSIkMTRmbbI8e6aPMiZxJmgACqCGKhY6ABGyDnkFFQ0dIzMbBwCwqIccabcYLyQoKjIEmh8kwN8DLAc5PzwwbLMyAAeK77IACYaQSEjUWY2Q-YAjABMAMwALA+gbsVjNXW8yxySoAADaAA0CCaZbPh1XYqXgOIY0ZgmcK0AA0nyaLFhhGY8F4AHJmEJILCWsgZId4NNfIgGFdcIcUTVfgBlZTOWC8T7kAJ42G4eT+GS42QyRaYbCgXAEEguTzeXyCjDBSAAQSE8Ai0Xsl0K9kcziExDeiQs1lAqSE6SyOTy0AKQ2KHk4p1V6s1OuuoHuzwArMagA) 在错误消息中，但 _只有_ 在按名称使用时才会出现。

在大多数情况下，你可以根据个人喜好进行选择，TypeScript 会告诉你它是否需要其他类型的声明。如果您想要启发式方法，可以使用 `interface` 直到你需要使用 `type` 中的功能。

## 类型断言

有时你会掌握一些 TypeScript 无法知道的关于某个值的类型信息。

例如，在使用 `document.getElementById` 时，TypeScript 只知道它会返回 _某_ 种 `HTMLElement`，但你可能清楚你的页面上必定有一个特定 ID 的 `HTMLCanvasElement`。

在这种情况下，可以使用 _类型断言_ 来指定一个更具体的类型：

```ts twoslash
const myCanvas = document.getElementById('main_canvas') as HTMLCanvasElement
```

与类型注解一样，类型断言会在编译时被移除，不会影响代码的运行时行为。

你也可以使用尖括号语法（除非代码在 `.tsx` 文件中），两者是等效的：

```ts twoslash
const myCanvas = <HTMLCanvasElement>document.getElementById('main_canvas')
```

> 提示：因为类型断言在编译时被移除，所以它不涉及任何运行时检查。
> 即使类型断言错误，也不会生成异常或 `null`。

TypeScript 只允许转换为 _更具体_ 或 _更宽泛_ 的类型。这条规则可以防止像下面这种“不可能”的强制转换：

```ts twoslash
// @errors: 2352
const x = 'hello' as number
```

有时这条规则过于保守，会阻止一些可能有效的复杂转换。若发生这种情况，可以使用两次断言，先断言为 `any`（或后续将介绍的 `unknown`），然后再断言为目标类型：

```ts twoslash
declare const expr: any
type T = { a: 1; b: 2; c: 3 }
// ---cut---
const a = expr as any as T
```

## 字面量类型

除了通用的 `string` 和 `number`，我们还可以在类型位置引用 _特定_ 的字符串和数字。

可以这样理解：JavaScript 提供了不同的变量声明方式。`var` 和 `let` 允许修改变量内部的值，而 `const` 不允许。这种特性也反映在 TypeScript 如何为字面量创建类型上。

```ts twoslash
let changingString = 'Hello World'
changingString = 'Olá Mundo'
// 因为 changingString 可以表示任何可能的字符串，所以
// TypeScript 在类型系统中就是这样描述它的
changingString
// ^?

const constantString = 'Hello World'
// 因为 constantString 只能表示唯一一种字符串，所以
// 它具有字面量类型表示
constantString
// ^?
```

仅凭自身，字面量类型并没有太大价值：

```ts twoslash
// @errors: 2322
let x: 'hello' = 'hello'
// OK
x = 'hello'
// ...
x = 'howdy'
```

一个只能拥有一种值的变量用处不大！

但是，通过将字面量 _组合_ 成联合类型，就能表达更有用的概念 —— 例如，编写只接受一组已知值的函数：

```ts twoslash
// @errors: 2345
function printText(s: string, alignment: 'left' | 'right' | 'center') {
  // ...
}
printText('Hello, world', 'left')
printText("G'day, mate", 'centre')
```

数字字面量类型的工作原理相同：

```ts twoslash
function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1
}
```

当然，也可以将它们与非字面量类型结合使用：

```ts twoslash
// @errors: 2345
interface Options {
  width: number
}
function configure(x: Options | 'auto') {
  // ...
}
configure({ width: 100 })
configure('auto')
configure('automatic')
```

还有一种字面量类型：布尔字面量。布尔字面量类型只有两个，正如你猜测的那样，它们是 `true` 和 `false`。`boolean` 类型本身实际上仅仅是联合类型 `true | false` 的别名。

### 字面量推断

当使用对象初始化变量时，TypeScript 假定该对象的属性值后续可能会改变。例如，对于下面这段代码：

```ts twoslash
declare const someCondition: boolean
// ---cut---
const obj = { counter: 0 }
if (someCondition) {
  obj.counter = 1
}
```

TypeScript 不认为将 `1` 赋给之前为 `0` 的字段是错误的。换言之，`obj.counter` 的类型必须被推断为 `number`，而不是 `0`，因为类型系统不仅要考量 _读取_ 行为，还要兼顾 _写入_ 行为。

同样的情况也适用于字符串：

```ts twoslash
// @errors: 2345
declare function handleRequest(url: string, method: 'GET' | 'POST'): void
// ---cut---
const req = { url: 'https://example.com', method: 'GET' }
handleRequest(req.url, req.method)
```

在上面的例子中，`req.method` 被推断为 `string`，而不是 `"GET"`。因为在创建 `req` 和调用 `handleRequest` 之间，代码可能会被求值，从而将 `"GUESS"` 之类的新字符串赋给 `req.method`，因此 TypeScript 认为这段代码存在错误。

有两种变通方法：

1. 可以在任意位置添加类型断言来改变推断结果：

   ```ts twoslash
   declare function handleRequest(url: string, method: 'GET' | 'POST'): void
   // ---cut---
   // 方式 1:
   const req = { url: 'https://example.com', method: 'GET' as 'GET' }
   // 方式 2:
   handleRequest(req.url, req.method as 'GET')
   ```

   - 方式 1 表示“我打算让 `req.method` 始终保持 _字面量类型_ `"GET"”`，从而防止随后可能向该字段赋予 `"GUESS"` 的行为。
   - 方式 2 表示“我出于其他原因确知 `req.method` 的值为 `"GET"`”。

2. 可以使用 `as const` 将整个对象转换为类型字面量：

   ```ts twoslash
   declare function handleRequest(url: string, method: 'GET' | 'POST'): void
   // ---cut---
   const req = { url: 'https://example.com', method: 'GET' } as const
   handleRequest(req.url, req.method)
   ```

`as const` 后缀对类型系统的作用类似于 `const` 对于 JavaScript，它确保所有属性都被赋予相应的字面量类型，而非更泛用的 `string` 或 `number` 版本。

## `null` 和 `undefined`

JavaScript 具有两个用于表示缺失或未初始化值的基本类型：`null` 和 `undefined`。

TypeScript 有两个同名的对应 _类型_。这些类型的行为取决于是否开启了 `strictNullChecks` 选项。

### 关闭 `strictNullChecks`

在 _关闭_ `strictNullChecks` 时，仍可正常访问可能为 `null` 或 `undefined` 的值，并且可以将 `null` 和 `undefined` 赋给任何类型的属性。这与没有空值检查的语言（如 C#、Java）的行为类似。缺乏对此类值的检查往往是 bug 的主要来源；我们始终建议在条件允许的情况下，在代码库中开启 `strictNullChecks`。

### 开启 `strictNullChecks`

在 _开启_ `strictNullChecks` 时，如果值为 `null` 或 `undefined`，必须先对其进行测试，然后才能调用该值的方法或属性。就像在使用可选属性前需要检查 `undefined` 一样，我们可以使用收窄来检查可能为 `null` 的值：

```ts twoslash
function doSomething(x: string | null) {
  if (x === null) {
    // 什么都不做
  } else {
    console.log('Hello, ' + x.toUpperCase())
  }
}
```

### 非空断言操作符 (后缀 `!`)

TypeScript 还提供了一种特殊语法，可以在不进行显式检查的情况下将 `null` 和 `undefined` 从类型中移除。在任何表达式之后写上 `!`，实际上就是一种类型断言，断言该值不为 `null` 或 `undefined`：

```ts twoslash
function liveDangerously(x?: number | null) {
  // 没有报错
  console.log(x!.toFixed())
}
```

与其他类型断言一样，它 _不会_ 改变代码的运行时行为，因此，只有在确知该值绝对不可能为 `null` 或 `undefined` 时，才应该使用 `!`。

## 枚举

枚举是 TypeScript 引入到 JavaScript 的一项功能，它允许描述一个可能是一组命名常量之一的值。与大多数 TypeScript 功能不同，这 _不是_ 在类型级别对 JavaScript 的补充，而是真正添加到了语言和运行时层面的特性。正因如此，你应该知道该功能的存在，但在完全确定之前，最好暂缓使用。有关枚举的更多信息，请参阅 [枚举参考页](/docs/handbook/enums.html)。

## 不常见的基本类型

值得一提的是，JavaScript 中其余的基本类型在类型系统中也有所体现。尽管我们在此不再深入探讨。

##### `bigint`

从 ES2020 开始，JavaScript 提供了一个用于处理极大整数的基本类型 `BigInt`：

```ts twoslash
// @target: es2020

// 通过 BigInt 函数创建 bigint
const oneHundred: bigint = BigInt(100)

// 通过字面量语法创建 BigInt
const anotherHundred: bigint = 100n
```

可以在 [TypeScript 3.2 发布说明](/docs/handbook/release-notes/typescript-3-2.html#bigint) 中了解有关 BigInt 的更多信息。

##### `symbol`

JavaScript 中有一个基本类型，用于通过 `Symbol()` 函数创建全局唯一的引用：

```ts twoslash
// @errors: 2367
const firstName = Symbol('name')
const secondName = Symbol('name')

if (firstName === secondName) {
  // 永远不可能发生
}
```

可以在 [Symbol 参考页](/docs/handbook/symbols.html) 中了解有关它们的更多信息。
