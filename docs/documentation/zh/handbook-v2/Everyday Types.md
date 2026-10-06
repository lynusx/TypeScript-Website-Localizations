---
title: 常见类型
layout: docs
permalink: /zh/docs/handbook/2/everyday-types.html
oneline: '语言的基本原始类型。'
---

在本章中，我们将介绍 JavaScript 代码中最常见的一些值类型，并说明在 TypeScript 中描述这些类型的相应方式。
这并非详尽无遗的清单，后续章节还会介绍更多命名和使用其他类型的方法。

类型不仅可以出现在类型注解中，还可以出现在许多其他*位置*。
在了解类型本身的同时，我们也会了解可以在哪些地方引用这些类型来构建新的结构。

首先，我们将回顾编写 JavaScript 或 TypeScript 代码时可能遇到的最基础、最常见的类型。
它们将构成后续更复杂类型的核心构建基石。

## 原始类型：`string`、`number` 和 `boolean`

JavaScript 拥有三种非常常用的[原始类型（primitives）](https://developer.mozilla.org/en-US/docs/Glossary/Primitive)：`string`、`number` 和 `boolean`。
每种原始类型在 TypeScript 中都有对应的类型。
正如你所预期的那样，这些类型名称与你对这些类型的值使用 JavaScript `typeof` 操作符时看到的结果一致：

- `string` 表示字符串值，例如 `"Hello, world"`
- `number` 表示数字，例如 `42`。JavaScript 在运行时没有专门的整数类型，因此不存在与 `int` 或 `float` 等价的类型——所有数字都统一为 `number`
- `boolean` 表示两个布尔值：`true` 和 `false`

> 类型名称 `String`、`Number` 和 `Boolean`（以大写字母开头）在语法上是合法的，但它们指的是某些特殊的内置类型，在你的代码中极少需要出现。在表示类型时，*始终*请使用 `string`、`number` 或 `boolean`。

## 数组

要指定形如 `[1, 2, 3]` 的数组类型，可以使用语法 `number[]`；该语法适用于任何类型（例如 `string[]` 表示字符串数组，依此类推）。
你可能还会看到写作 `Array<number>` 的形式，两者含义完全相同。
在后续介绍*泛型*时，我们会详细了解形如 `T<U>` 的语法。

> 请注意，`[number]` 表示另一种完全不同的概念；请参阅[元组（Tuples）](/docs/handbook/2/objects.html#tuple-types)章节。

## `any`

TypeScript 还提供了一个特殊类型 `any`，当你不希望某个特定值引发类型检查错误时，便可以使用它。

当一个值的类型为 `any` 时，你可以访问它的任意属性（这些属性的类型反过来也都是 `any`）、将其作为函数调用、将其赋值给任何类型的值（或接受任何类型的值赋值），以及进行几乎任何语法上合法的操作：

```ts twoslash
let obj: any = { x: 0 }
// 以下任何一行代码都不会抛出编译器错误。
// 使用 `any` 会禁用所有后续的类型检查，并且假设
// 你比 TypeScript 更了解所处的运行环境。
obj.foo()
obj()
obj.bar = 100
obj = 'hello'
const n: number = obj
```

当你不希望为了让 TypeScript 认可某一行代码而编写冗长的类型定义时，`any` 类型非常有用。

### `noImplicitAny`

当你未指定类型，且 TypeScript 无法从上下文中推断出类型时，编译器通常会默认其为 `any`。

然而，通常情况下你应该尽量避免这种情况，因为 `any` 不会进行类型检查。
可以使用编译器标志 [`noImplicitAny`](/tsconfig#noImplicitAny) 将任何隐式推断的 `any` 标记为错误。

## 变量上的类型注解

当你使用 `const`、`var` 或 `let` 声明变量时，可以选择添加类型注解来显式指定该变量的类型：

```ts twoslash
let myName: string = 'Alice'
//        ^^^^^^^^ 类型注解
```

> TypeScript 不使用类似 `int x = 0;` 这样“类型在左侧”的声明风格。
> 类型注解总是位于被标注对象的*后面*。

不过在大多数情况下，并不需要显式标注。
TypeScript 会尽可能尝试自动*推断*代码中的类型。
例如，变量的类型会根据其初始值的类型来进行推断：

```ts twoslash
// 无需类型注解——'myName' 会被推断为 'string' 类型
let myName = 'Alice'
```

通常情况下，你无需刻意去死记类型推断的具体规则。
如果你刚刚上手，不妨尝试少写一些类型注解——你可能会惊讶地发现，只需极少的显式注解，TypeScript 就能完全理解代码的类型意图。

## 函数

函数是 JavaScript 中传递数据的主要方式。
TypeScript 允许你指定函数输入值和输出值的类型。

### 参数类型注解

声明函数时，可以在每个参数后面添加类型注解，以声明该函数接受何种类型的参数。
参数类型注解放在参数名称之后：

```ts twoslash
// 参数类型注解
function greet(name: string) {
  //                 ^^^^^^^^
  console.log('Hello, ' + name.toUpperCase() + '!!')
}
```

当参数具有类型注解时，传入该函数的实参就会受到检查：

```ts twoslash
// @errors: 2345
declare function greet(name: string): void
// ---cut---
// 如果执行，将在运行时报错！
greet(42)
```

> 即使你的参数没有类型注解，TypeScript 依然会检查你传递的实参数量是否正确。

### 返回值类型注解

你也可以添加返回值类型注解。
返回值类型注解位于参数列表之后：

```ts twoslash
function getFavoriteNumber(): number {
  //                        ^^^^^^^^
  return 26
}
```

与变量类型注解十分相似，通常你不需要显式编写返回值类型注解，因为 TypeScript 会根据函数内部的 `return` 语句自动推断其返回值类型。
在上面的示例中，添加类型注解并不会改变任何行为。
不过在某些代码库中，开发者会出于编写文档、防止意外更改或个人偏好的目的，显式指定返回值类型。

#### 返回 Promise 的函数

如果要标注返回 Promise 的函数的返回值类型，应该使用 `Promise` 类型：

```ts twoslash
async function getFavoriteNumber(): Promise<number> {
  return 26
}
```

### 匿名函数

匿名函数与函数声明略有不同。
当函数出现在 TypeScript 能够确定其调用方式的位置时，该函数的参数会自动获得类型。

示例如下：

```ts twoslash
// @errors: 2551
const names = ['Alice', 'Bob', 'Eve']

// 函数的上下文类型推断——参数 s 被推断为 string 类型
names.forEach(function (s) {
  console.log(s.toUpperCase())
})

// 上下文类型推断同样适用于箭头函数
names.forEach((s) => {
  console.log(s.toUpperCase())
})
```

尽管参数 `s` 没有类型注解，TypeScript 还是结合了 `forEach` 函数的类型定义以及数组的推断类型，推导出了 `s` 的类型。

这个过程被称为*上下文类型推断*（contextual typing），因为函数所在的*上下文*决定了它应该具备的类型。

与类型推断规则类似，你无需刻意去记忆它是如何运作的，但理解它*确实在生效*有助于你识别何时无需编写冗余的类型注解。
稍后，我们将看到更多关于“值所在的上下文如何影响其类型”的示例。

## 对象类型

除了原始类型之外，最常见的类型就是*对象类型*（object type）。
它指的是任何带有属性的 JavaScript 值，而几乎所有值都属于此类！
要定义对象类型，只需列出其各个属性以及对应的类型。

例如，下面是一个接受类似坐标点对象的函数：

```ts twoslash
// 参数的类型注解是一个对象类型
function printCoord(pt: { x: number; y: number }) {
  //                      ^^^^^^^^^^^^^^^^^^^^^^^^
  console.log("The coordinate's x value is " + pt.x)
  console.log("The coordinate's y value is " + pt.y)
}
printCoord({ x: 3, y: 7 })
```

这里，我们给参数标注了一个具有两个属性（`x` 和 `y`）的类型，这两个属性的类型均为 `number`。
可以使用 `,` 或 `;` 来分隔各个属性，无论哪种方式，最后一个属性后的分隔符都是可选的。

每个属性的类型标注也是可选的。
如果不指定属性的类型，它将被视为 `any`。

### 可选属性

对象类型还可以指定其部分或全部属性为*可选*（optional）属性。
为此，只需在属性名后加上 `?`：

```ts twoslash
function printName(obj: { first: string; last?: string }) {
  // ...
}
// 两种方式均正常
printName({ first: 'Bob' })
printName({ first: 'Alice', last: 'Alisson' })
```

在 JavaScript 中，访问不存在的属性会得到 `undefined` 值，而不是引发运行时错误。
因此，当你*读取*可选属性时，在使用它之前必须先检查其是否为 `undefined`。

```ts twoslash
// @errors: 18048
function printName(obj: { first: string; last?: string }) {
  // 错误——如果未提供 'obj.last'，可能会崩溃！
  console.log(obj.last.toUpperCase())
  if (obj.last !== undefined) {
    // 正常
    console.log(obj.last.toUpperCase())
  }

  // 使用现代 JavaScript 语法的安全替代方案：
  console.log(obj.last?.toUpperCase())
}
```

## 联合类型

TypeScript 的类型系统允许你使用各种操作符基于现有类型构建新类型。
既然我们已经了解了如何编写几种基础类型，现在是时候开始以有趣的方式*组合*它们了。

### 定义联合类型

你可能见到的第一种类型组合方式是*联合类型*（union type）。
联合类型是由两个或多个其他类型组成的类型，表示可能属于其中*任何一个*类型的值。
我们把其中的每一个类型称为联合类型的*成员*（members）。

我们来编写一个既能处理字符串又能处理数字的函数：

```ts twoslash
// @errors: 2345
function printId(id: number | string) {
  console.log('Your ID is: ' + id)
}
// 正常
printId(101)
// 正常
printId('202')
// 错误
printId({ myID: 22342 })
```

> 联合类型成员之间的分隔符允许出现在第一个元素之前，因此你也可以这样写：
>
> ```ts twoslash
> function printTextOrNumberOrBool(
>   textOrNumberOrBool: string | number | boolean,
> ) {
>   console.log(textOrNumberOrBool)
> }
> ```

### 使用联合类型

*提供*匹配联合类型的值很容易——只需提供与联合类型的任意成员相匹配的类型即可。
但如果你手头*拥有*一个联合类型的值，该如何使用它呢？

TypeScript 仅允许对联合类型的*每个*成员都有效的操作。
例如，如果拥有联合类型 `string | number`，你将无法调用仅在 `string` 上存在的方法：

```ts twoslash
// @errors: 2339
function printId(id: number | string) {
  console.log(id.toUpperCase())
}
```

解决方案是通过代码来*缩小*（narrow）该联合类型的范围，就像你在没有类型注解的原生 JavaScript 中所做的那样。
当 TypeScript 能根据代码结构为某个值推导出更具体的类型时，就会发生*类型缩小*（narrowing）。

例如，TypeScript 知道只有 `string` 类型的值其 `typeof` 运算结果才会是 `"string"`：

```ts twoslash
function printId(id: number | string) {
  if (typeof id === 'string') {
    // 在此分支中，id 的类型为 'string'
    console.log(id.toUpperCase())
  } else {
    // 在这里，id 的类型为 'number'
    console.log(id)
  }
}
```

另一个例子是使用类似 `Array.isArray` 的函数：

```ts twoslash
function welcomePeople(x: string[] | string) {
  if (Array.isArray(x)) {
    // 在这里：'x' 为 'string[]'
    console.log('Hello, ' + x.join(' and '))
  } else {
    // 在这里：'x' 为 'string'
    console.log('Welcome lone traveler ' + x)
  }
}
```

请注意，在 `else` 分支中，我们不需要做任何特殊处理——如果 `x` 不是 `string[]`，那么它必定是 `string`。

有时，联合类型中的所有成员都具有某种共同特征。
例如，数组和字符串都具有 `slice` 方法。
如果联合类型中的每个成员都具有某个共同属性，则无需缩小范围即可直接使用该属性：

```ts twoslash
// 返回值类型被推断为 number[] | string
function getFirstThree(x: number[] | string) {
  return x.slice(0, 3)
}
```

> 类型的*并集*（union）表面上看起来只拥有这些类型属性的*交集*（intersection），这可能会让人感到困惑。
> 这一点并非巧合——“联合（union）”这个名称源于类型论。
> 联合类型 `number | string` 是通过对每个类型的*值*取并集构成的。
> 请注意，给定两个集合以及关于每个集合的已知事实，只有这些事实的*交集*才适用于集合本身的*并集*。
> 例如，如果一个房间里全都是戴帽子的个子高的人，另一个房间里全都是戴帽子的说西班牙语的人，把这两个房间的人合并在一起后，我们对*每个人*唯一确知的事实就是他们都戴着帽子。

## 类型别名

此前我们一直通过在类型注解中直接编写对象类型和联合类型来使用它们。
这很方便，但很多时候我们希望多次复用同一个类型，并通过一个单一的名称来引用它。

_类型别名_（type alias）正是为此而生——它是一个任何*类型*的*名称*。
类型别名的语法如下：

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

实际上，你可以使用类型别名给任何类型命名，而不仅仅是对象类型。
例如，类型别名可以为联合类型命名：

```ts twoslash
type ID = number | string
```

请注意，别名*仅仅*是别名——你无法使用类型别名来创建同一个类型的不同/独立“版本”。
当你使用别名时，其效果与直接写出被别名引用的类型完全相同。
换句话说，以下代码虽然*看起来*像是非法的，但在 TypeScript 眼中完全正常，因为这两个类型都是同一个类型的别名：

```ts twoslash
declare function getInput(): string
declare function sanitize(str: string): string
// ---cut---
type UserInputSanitizedString = string

function sanitizeInput(str: string): UserInputSanitizedString {
  return sanitize(str)
}

// 创建一个经过净化的输入
let userInput = sanitizeInput(getInput())

// 不过仍然可以被重新赋值为字符串
userInput = 'new input'
```

## 接口

_接口声明_（interface declaration）是为对象类型命名的另一种方式：

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

就像上面使用类型别名一样，该示例的运行效果与直接使用匿名对象类型完全相同。
TypeScript 只关心传递给 `printCoord` 的值的*结构*——只要它具有预期的属性即可。
这种只关注类型的结构与能力的特性，正是我们称 TypeScript 为*结构化类型*（structurally typed）系统的原因。

### 类型别名与接口的区别

类型别名与接口非常相似，在很多情况下你可以自由选择使用哪一个。
`interface` 的几乎所有特性都可以在 `type` 中使用，两者最核心的区别在于：类型别名创建后无法重新打开以添加新属性，而接口始终是可扩展的。

<div class='table-container'>
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
  name: string;
}<br/>
interface Bear extends Animal {
  honey: boolean;
}<br/>
const bear = getBear();
bear.name;
bear.honey;
        </pre></code>
      </td>
      <td>
        <p>通过交叉类型扩展类型</p>
        <code><pre>
type Animal = {
  name: string;
}<br/>
type Bear = Animal & { 
  honey: boolean;
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
  title: string;
}<br/>
interface Window {
  ts: TypeScriptAPI;
}<br/>
const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
        </pre></code>
      </td>
      <td>
        <p>类型在创建后无法更改</p>
        <code><pre>
type Window = {
  title: string;
}<br/>
type Window = {
  ts: TypeScriptAPI;
}<br/>
<span style="color: #A31515"> // 错误：标识符 'Window' 重复。</span><br/>
        </pre></code>
      </td>
    </tr>
    </tbody>
</table>
</div>

在后续章节中你将学到更多关于这些概念的内容，因此如果无法立即完全理解也不必担心。

- 在 TypeScript 4.2 版本之前，类型别名的名称[_可能_会出现在错误信息中](/play?#code/PTAEGEHsFsAcEsA2BTATqNrLusgzngIYDm+oA7koqIYuYQJ56gCueyoAUCKAC4AWHAHaFcoSADMaQ0PCG80EwgGNkALk6c5C1EtWgAsqOi1QAb06groEbjWg8vVHOKcAvpokshy3vEgyyMr8kEbQJogAFND2YREAlOaW1soBeJAoAHSIkMTRmbbI8e6aPMiZxJmgACqCGKhY6ABGyDnkFFQ0dIzMbBwCwqIccabcYLyQoKjIEmh8kwN8DLAc5PzwwbLMyAAeK77IACYaQSEjUWZWhfYAjABMAMwALA+gbsVjoADqgjKESytQPxCHghAByXigYgBfr8LAsYj8aQMUASbDQcRSExCeCwFiIQh+AKfAYyBiQFgOPyIaikSGLQo0Zj-aazaY+dSaXjLDgAGXgAC9CKhDqAALxJaw2Ib2RzOISuDycLw+ImBYKQflCkWRRD2LXCw6JCxS1JCdJZHJ5RAFIbFJU8ADKC3WzEcnVZaGYE1ABpFnFOmsFhsil2uoHuzwArO9SmAAEIsSFrZB-GgAjjA5gtVN8VCEc1o1C4Q4AGlR2AwO1EsBQoAAbvB-gJ4HhPgB5aDwem-Ph1TCV3AEEirTp4ELtRbTPD4vwKjOfAuioSQHuDXBcnmgACC+eCONFEs73YAPGGZVT5cRyyhiHh7AAON7lsG3vBggB8XGV3l8-nVISOgghxoLq9i7io-AHsayRWGaFrlFauq2rg9qaIGQHwCBqChtKdgRo8TxRjeyB3o+7xAA)，有时会代替等价的匿名类型出现（这可能符合也可能不符合预期）。而接口名称则始终会显示在错误信息中。
- 类型别名无法参与[声明合并，但接口可以](/play?#code/PTAEEEDtQS0gXApgJwGYEMDGjSfdAIx2UQFoB7AB0UkQBMAoEUfO0Wgd1ADd0AbAK6IAzizp16ALgYM4SNFhwBZdAFtV-UAG8GoPaADmNAcMmhh8ZHAMMAvjLkoM2UCvWad+0ARL0A-GYWVpA29gyY5JAWLJAwGnxmbvGgALzauvpGkCZmAEQAjABMAMwALLkANBl6zABi6DB8okR4Jjg+iPSgABboovDk3jjo5pbW1d6+dGb5djLwAJ7UoABKiJTwjThpnpnGpqPBoTLMAJrkArj4kOTwYmycPOhW6AR8IrDQ8N04wmo4HHQCwYi2Waw2W1S6S8HX8gTGITsQA)。
- 接口只能用于[声明对象的形状，无法重命名原始类型](/play?#code/PTAEAkFMCdIcgM6gC4HcD2pIA8CGBbABwBtIl0AzUAKBFAFcEBLAOwHMUBPQs0XFgCahWyGBVwBjMrTDJMAshOhMARpD4tQ6FQCtIE5DWoixk9QEEWAeV37kARlABvaqDegAbrmL1IALlAEZGV2agBfampkbgtrWwMAJlAAXmdXdy8ff0Dg1jZwyLoAVWZ2Lh5QVHUJflAlSFxROsY5fFAWAmk6CnRoLGwmILzQQmV8JmQmDzI-SOiKgGV+CaYAL0gBBdyy1KCQ-Pn1AFFplgA5enw1PtSWS+vCsAAVAAtB4QQWOEMKBuYVUiVCYvYQsUTQcRSBDGMGmKSgAAa-VEgiQe2GLgKQA)。
- 接口名称在错误信息中[_始终_会以其原始形式出现](/play?#code/PTAEGEHsFsAcEsA2BTATqNrLusgzngIYDm+oA7koqIYuYQJ56gCueyoAUCKAC4AWHAHaFcoSADMaQ0PCG80EwgGNkALk6c5C1EtWgAsqOi1QAb06groEbjWg8vVHOKcAvpokshy3vEgyyMr8kEbQJogAFND2YREAlOaW1soBeJAoAHSIkMTRmbbI8e6aPMiZxJmgACqCGKhY6ABGyDnkFFQ0dIzMbBwCwqIccabcYLyQoKjIEmh8kwN8DLAc5PzwwbLMyAAeK77IACYaQSEjUWZWhfYAjABMAMwALA+gbsVjNXW8yxySoAADaAA0CCaZbPh1XYqXgOIY0ZgmcK0AA0nyaLFhhGY8F4AHJmEJILCWsgZId4NNfIgGFdcIcUTVfgBlZTOWC8T7kAJ42G4eT+GS42QyRaYbCgXAEEguTzeXyCjDBSAAQSE8Ai0Xsl0K9kcziExDeiQs1lAqSE6SyOTy0AKQ2KHk4p1V6s1OuuoHuzwArMagA)，但*仅限*于它们被按名引用的情况。
- 带有 `extends` 的接口在编译器中的性能表现[通常优于](https://github.com/microsoft/TypeScript/wiki/Performance#preferring-interfaces-over-intersections)带有交叉类型的类型别名。

在大多数情况下，你可以根据个人喜好进行选择，当 TypeScript 需要特定的声明形式时会给出提示。如果你想要一个启发式原则，可以优先使用 `interface`，直到你需要使用 `type` 的特性为止。

## 类型断言

有时你会掌握一些 TypeScript 无法知晓的值的类型信息。

例如，如果你在使用 `document.getElementById`，TypeScript 只知道它会返回*某种* `HTMLElement`，但你可能清楚你的页面上具有该特定 ID 的元素始终是一个 `HTMLCanvasElement`。

在这种情况下，你可以使用*类型断言*（type assertion）来指定一个更具体的类型：

```ts twoslash
const myCanvas = document.getElementById('main_canvas') as HTMLCanvasElement
```

就像类型注解一样，类型断言会在编译阶段被移除，不会对代码的运行时行为产生任何影响。

你也可以使用尖括号语法（除非代码位于 `.tsx` 文件中），两者完全等价：

```ts twoslash
const myCanvas = <HTMLCanvasElement>document.getElementById('main_canvas')
```

> 提示：由于类型断言在编译时被移除，因此类型断言不会伴随任何运行时检查。
> 即使类型断言是错误的，也不会抛出异常或产生 `null`。

TypeScript 仅允许将类型断言转换为更*具体*或更*不具体*的版本。
这一规则可以防止“不可能”的强制转换，例如：

```ts twoslash
// @errors: 2352
const x = 'hello' as number
```

有时该规则可能过于保守，会阻止某些原本有效的更复杂的强制转换。
在这种情况下，你可以进行两次断言：首先断言为 `any`（或后续将介绍的 `unknown`），然后再断言为目标类型：

```ts twoslash
declare const expr: any
type T = { a: 1; b: 2; c: 3 }
// ---cut---
const a = expr as any as T
```

## 字面量类型

除了通用的 `string` 和 `number` 类型之外，我们还可以在类型位置引用*具体的*字符串和数字。

理解这一点的一种方式是，思考 JavaScript 提供的不同变量声明方式。`var` 和 `let` 都允许修改变量中保存的内容，而 `const` 则不允许。这种区别直接体现在 TypeScript 为字面量创建类型的方式上。

```ts twoslash
let changingString = 'Hello World'
changingString = 'Olá Mundo'
// 因为 `changingString` 可以表示任何可能的字符串，
// 所以 TypeScript 在类型系统中也是这样描述它的
changingString
// ^?

const constantString = 'Hello World'
// 因为 `constantString` 只能表示 1 种可能的字符串，
// 所以它具有字面量类型表示
constantString
// ^?
```

单就字面量类型本身而言，其用处并不大：

```ts twoslash
// @errors: 2322
let x: 'hello' = 'hello'
// 正常
x = 'hello'
// ...
x = 'howdy'
```

一个只能拥有唯一定值的变量并没有太大用处！

但是，通过将字面量*组合*成联合类型，你就可以表达出强大得多的概念——例如，只接受一组特定已知值的函数：

```ts twoslash
// @errors: 2345
function printText(s: string, alignment: 'left' | 'right' | 'center') {
  // ...
}
printText('Hello, world', 'left')
printText("G'day, mate", 'centre')
```

数字字面量类型的工作方式也是如此：

```ts twoslash
function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1
}
```

当然，你也可以将字面量类型与非字面量类型进行组合：

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

还有一种字面量类型：布尔字面量。
布尔字面量类型只有两种，正如你猜想的那样，它们就是类型 `true` 和 `false`。
`boolean` 类型本身实际上就是联合类型 `true | false` 的别名。

### 字面量推断

当你用一个对象初始化变量时，TypeScript 会假定该对象的属性值在之后可能会发生改变。
例如，如果你写了这样的代码：

```ts twoslash
declare const someCondition: boolean
// ---cut---
const obj = { counter: 0 }
if (someCondition) {
  obj.counter = 1
}
```

TypeScript 不会认为将 `1` 赋给之前为 `0` 的字段是一个错误。
换一种说法，`obj.counter` 的类型必须是 `number`，而不是 `0`，因为类型同时用于确定*读取*和*写入*行为。

同样的规则也适用于字符串：

```ts twoslash
// @errors: 2345
declare function handleRequest(url: string, method: 'GET' | 'POST'): void

const req = { url: 'https://example.com', method: 'GET' }
handleRequest(req.url, req.method)
```

在上面的示例中，`req.method` 被推断为 `string`，而不是 `"GET"`。因为在创建 `req` 和调用 `handleRequest` 之间可能会执行其他代码，可能会给 `req.method` 赋予一个新字符串（如 `"GUESS"`），所以 TypeScript 认为这段代码存在错误。

有两种方法可以解决此问题：

1. 你可以通过在任一位置添加类型断言来改变推断结果：

   ```ts twoslash
   declare function handleRequest(url: string, method: 'GET' | 'POST'): void
   // ---cut---
   // 修改 1：
   const req = { url: 'https://example.com', method: 'GET' as 'GET' }
   // 修改 2
   handleRequest(req.url, req.method as 'GET')
   ```

   修改 1 的含义是“我希望 `req.method` 始终具有*字面量类型* `"GET"`”，从而防止后续向该字段赋值 `"GUESS"` 的可能。
   修改 2 的含义是“由于其他原因，我确信 `req.method` 的值为 `"GET"`”。

2. 你可以使用 `as const` 将整个对象转换为字面量类型：

   ```ts twoslash
   declare function handleRequest(url: string, method: 'GET' | 'POST'): void
   // ---cut---
   const req = { url: 'https://example.com', method: 'GET' } as const
   handleRequest(req.url, req.method)
   ```

`as const` 后缀的作用类似于类型系统层面的 `const`，它确保所有属性都被赋予字面量类型，而不是更通用的 `string` 或 `number` 版本。

## `null` 与 `undefined`

JavaScript 具有两个用于表示值缺失或未初始化的原始值：`null` 和 `undefined`。

TypeScript 具有两个同名的对应*类型*。这两个类型的行为取决于你是否开启了 [`strictNullChecks`](/tsconfig#strictNullChecks) 选项。

### 关闭 `strictNullChecks`

在*关闭* [`strictNullChecks`](/tsconfig#strictNullChecks) 时，可能为 `null` 或 `undefined` 的值仍然可以正常访问，并且可以将 `null` 和 `undefined` 赋值给任何类型的属性。
这类似于没有空值检查的编程语言（例如旧版 C#、Java）的行为。
缺乏对这些值的检查往往是 bug 的主要来源；只要代码库允许，我们始终建议开启 [`strictNullChecks`](/tsconfig#strictNullChecks)。

### 开启 `strictNullChecks`

在*开启* [`strictNullChecks`](/tsconfig#strictNullChecks) 时，当一个值为 `null` 或 `undefined` 时，你在访问该值的方法或属性之前必须先对其进行检查。
就像在使用可选属性前检查 `undefined` 一样，我们可以使用*类型缩小*（narrowing）来检查可能为 `null` 的值：

```ts twoslash
function doSomething(x: string | null) {
  if (x === null) {
    // 什么也不做
  } else {
    console.log('Hello, ' + x.toUpperCase())
  }
}
```

### 非空断言操作符（后缀 `!`）

TypeScript 还提供了一种特殊的语法，可以在不进行显式检查的情况下从类型中移除 `null` 和 `undefined`。
在任何表达式后面写上 `!`，实际上就是在断言该值既不是 `null` 也不是 `undefined`：

```ts twoslash
function liveDangerously(x?: number | null) {
  // 没有错误
  console.log(x!.toFixed())
}
```

与其他类型断言一样，这不会改变代码的运行时行为，因此务必只在你确信该值*绝不可能是* `null` 或 `undefined` 时才使用 `!`。

## 枚举

枚举（Enums）是 TypeScript 为 JavaScript 添加的一项特性，允许描述一个可能是某组具名常量之一的值。与大多数 TypeScript 特性不同，这*不是*单纯在类型层面对 JavaScript 进行的扩展，而是向语言本身和运行时引入的新特性。正因如此，你应该了解它的存在，但在确有必要之前可以先暂缓使用。你可以在[枚举参考页面](/docs/handbook/enums.html)中了解更多关于枚举的内容。

## 较少使用的原始类型

值得一提的是，JavaScript 中的其余原始类型也在类型系统中得到了支持。
不过我们在此不展开深入讨论。

#### `bigint`

从 ES2020 开始，JavaScript 引入了一个用于表示非常大整数的原始类型 `BigInt`：

```ts twoslash
// @target: es2020

// 通过 BigInt 函数创建 bigint
const oneHundred: bigint = BigInt(100)

// 通过字面量语法创建 BigInt
const anotherHundred: bigint = 100n
```

你可以在 [TypeScript 3.2 发布说明](/docs/handbook/release-notes/typescript-3-2.html#bigint)中了解有关 BigInt 的更多信息。

#### `symbol`

JavaScript 中还有一个原始类型，用于通过 `Symbol()` 函数创建全局唯一的引用：

```ts twoslash
// @errors: 2367
const firstName = Symbol('name')
const secondName = Symbol('name')

if (firstName === secondName) {
  // 绝不可能发生
}
```

你可以在 [Symbols 参考页面](/docs/handbook/symbols.html)中了解关于它们的更多信息。
