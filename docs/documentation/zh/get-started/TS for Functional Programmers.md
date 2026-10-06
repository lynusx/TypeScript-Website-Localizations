---
title: 针对函数式程序员的 TypeScript 指南
short: 针对函数式程序员的 TS 指南
layout: docs
permalink: /zh/docs/handbook/typescript-in-5-minutes-func.html
oneline: 针对具备函数式编程背景的开发者学习 TypeScript
---

TypeScript 最初诞生于将传统的面向对象类型引入 JavaScript 的一次尝试，以便微软的程序员能够将传统的面向对象程序搬到 Web 上。随着自身的发展，TypeScript 的类型系统不断演进，以对原生 JavaScript 开发者编写的代码进行建模。由此形成的系统功能强大、妙趣横生，但也颇具复杂性。

本指南专为希望学习 TypeScript 的在职 Haskell 或 ML 程序员设计。它阐述了 TypeScript 的类型系统与 Haskell 类型系统的不同之处，并介绍了 TypeScript 类型系统中那些因对 JavaScript 代码建模而产生的独有特性。

本指南不涉及面向对象编程。在实践中，TypeScript 中的面向对象程序与其它具有 OO 特性的主流语言中的程序非常相似。

## 前提条件

在本指南中，假定你已经了解以下内容：

- 如何编写 JavaScript 代码（精华部分）。
- C 衍生语言的类型语法。

如果你需要学习 JavaScript 的精华部分，建议阅读 [JavaScript: The Good Parts](https://shop.oreilly.com/product/9780596517748.do)（《JavaScript 语言精粹》）。如果你懂得如何在一门具备传值调用、词法作用域、充斥着大量可变性而几乎没有其他特性的语言中编写程序，也可以略过这本书。[R<sup>4</sup>RS Scheme](https://people.csail.mit.edu/jaffer/r4rs.pdf) 就是一个很好的例子。

[The C++ Programming Language](http://www.stroustrup.com/4th.html) 是了解 C 风格类型语法的好资料。与 C++ 不同的是，TypeScript 使用后置类型语法，例如：`x: string` 而不是 `string x`。

## Haskell 中不存在的概念

### 内置类型

JavaScript 定义了 8 种内置类型：

| 类型        | 说明                          |
| ----------- | ----------------------------- |
| `Number`    | 双精度 IEEE 754 浮点数。      |
| `String`    | 不可变的 UTF-16 字符串。      |
| `BigInt`    | 任意精度格式的整数。          |
| `Boolean`   | `true` 和 `false`。           |
| `Symbol`    | 通常用作键的唯一值。          |
| `Null`      | 等价于单元类型（unit type）。 |
| `Undefined` | 同样等价于单元类型。          |
| `Object`    | 类似于记录（record）。        |

[有关更多详细信息，请参阅 MDN 页面](https://developer.mozilla.org/docs/Web/JavaScript/Data_structures)。

TypeScript 为这些内置类型提供了对应的原始类型：

- `number`
- `string`
- `bigint`
- `boolean`
- `symbol`
- `null`
- `undefined`
- `object`

#### 其他重要的 TypeScript 类型

| 类型          | 说明                                 |
| ------------- | ------------------------------------ |
| `unknown`     | 顶类型（top type）。                 |
| `never`       | 底类型（bottom type）。              |
| 对象字面量    | 例如 `{ property: Type }`            |
| `void`        | 用于未声明返回值（或无返回值）的函数 |
| `T[]`         | 可变数组，亦可写作 `Array<T>`        |
| `[T, T]`      | 元组（tuple），长度固定但内容可变    |
| `(t: T) => U` | 函数                                 |

注意：

1. 函数类型语法包含参数名称。这一点相当难以适应！

   ```ts
   let fst: (a: any, b: any) => any = (a, b) => a

   // 或者更准确地说：

   let fst: <T, U>(a: T, b: U) => T = (a, b) => a
   ```

2. 对象字面量类型语法与对象字面量值的语法高度相似：

   ```ts
   let o: { n: number; xs: object[] } = { n: 1, xs: [] }
   ```

3. `[T, T]` 是 `T[]` 的子类型。这与 Haskell 不同，在 Haskell 中元组与列表毫无关联。

#### 装箱类型

JavaScript 拥有与原始类型对应的装箱等价物，其中包含了程序员通常与这些类型相关联的方法。TypeScript 同样反映了这一点，例如原始类型 `number` 与装箱类型 `Number` 之间的区别。我们极少需要使用装箱类型，因为它们的方法返回的都是原始值。

```ts
;(1).toExponential()
// equivalent to
Number.prototype.toExponential.call(1)
```

注意，在数字字面量上调用方法时，需要将其放在圆括号中以辅助解析器进行解析。

### 渐进式类型

每当无法确定某个表达式应该是什么类型时，TypeScript 就会使用类型 `any`。与 `Dynamic` 相比，把 `any` 称为一种类型有些言过其实。它仅仅是在出现的地方关闭了类型检查器。例如，你可以将任何值推入 `any[]` 中，而无需对该值进行任何标记：

```ts twoslash
// with "noImplicitAny": false in tsconfig.json, anys: any[]
const anys = []
anys.push(1)
anys.push('oh no')
anys.push({ anything: 'goes' })
```

并且你可以在任何地方使用 `any` 类型的表达式：

```ts
anys.map(anys[1]) // oh no, "oh no" is not a function
```

`any` 还具有传染性——如果你使用类型为 `any` 的表达式来初始化一个变量，该变量的类型也同样会变成 `any`。

```ts
let sepsis = anys[0] + anys[1] // this could mean anything
```

若希望在 TypeScript 产生 `any` 时报错，请在 `tsconfig.json` 中配置 `"noImplicitAny": true` 或 `"strict": true`。

### 结构化类型

大多数函数式程序员对结构化类型（structural typing）这一概念并不陌生，尽管 Haskell 和大多数 ML 方言并非结构化类型的。它的基本形式相当简单：

```ts
// @strict: false
let o = { x: 'hi', extra: 1 } // ok
let o2: { x: string } = o // ok
```

在这里，对象字面量 `{ x: "hi", extra: 1 }` 具有对应的字面量类型 `{ x: string, extra: number }`。该类型可以赋值给 `{ x: string }`，因为它具备所有必需的属性，且这些属性的类型都是可赋值的。额外的属性并不会阻碍赋值，它只是使其成为了 `{ x: string }` 的子类型。

命名类型只是给类型赋予一个名称；出于可赋值性的考虑，下方的类型别名 `One` 与接口类型 `Two` 之间没有任何区别。它们都具有一个属性 `p: string`。（不过，在递归定义和类型参数方面，类型别名与接口的行为有所不同。）

```ts twoslash
// @errors: 2322
type One = { p: string }
interface Two {
  p: string
}
class Three {
  p = 'Hello'
}

let x: One = { p: 'hi' }
let two: Two = x
two = new Three()
```

### 联合类型

在 TypeScript 中，联合类型是无标签的（untagged unions）。换言之，它们不像 Haskell 中的 `data` 那样是可辨识联合（discriminated unions）。不过，你通常可以使用内置标签或其他属性来区分联合中的各个类型。

```ts twoslash
function start(
  arg: string | string[] | (() => string) | { s: string },
): string {
  // this is super common in JavaScript
  if (typeof arg === 'string') {
    return commonCase(arg)
  } else if (Array.isArray(arg)) {
    return arg.map(commonCase).join(',')
  } else if (typeof arg === 'function') {
    return commonCase(arg())
  } else {
    return commonCase(arg.s)
  }

  function commonCase(s: string): string {
    // finally, just convert a string to another string
    return s
  }
}
```

`string`、`Array` 和 `Function` 具有内置类型谓词（type predicates），从而巧妙地将对象类型留给了 `else` 分支。然而，有时也可能会产生在运行时难以区分的联合。对于新编写的代码，最好只构建可辨识联合。

以下类型具有内置谓词：

| 类型      | 谓词                               |
| --------- | ---------------------------------- |
| string    | `typeof s === "string"`            |
| number    | `typeof n === "number"`            |
| bigint    | `typeof m === "bigint"`            |
| boolean   | `typeof b === "boolean"`           |
| symbol    | `typeof g === "symbol"`            |
| undefined | `typeof undefined === "undefined"` |
| function  | `typeof f === "function"`          |
| array     | `Array.isArray(a)`                 |
| object    | `typeof o === "object"`            |

请注意，函数和数组在运行时虽然都是对象，但它们拥有专属的谓词。

#### 交叉类型

除了联合类型，TypeScript 还支持交叉类型（intersections）：

```ts twoslash
type Combined = { a: number } & { b: string }
type Conflicting = { a: number } & { a: string }
```

`Combined` 拥有 `a` 和 `b` 两个属性，就如同它们被写在一个对象字面量类型中一样。当发生冲突时，交叉和联合是递归处理的，因此 `Conflicting.a: number & string`。

### 单元类型

单元类型（unit types）是仅包含单个特定原始值的原始类型的子类型。例如，字符串 `"foo"` 具有类型 `"foo"`。由于 JavaScript 没有内置的枚举，因此通常使用一组预定义的已知字符串来代替。字符串字面量类型的联合让 TypeScript 能够对此类模式进行类型化：

```ts twoslash
declare function pad(s: string, n: number, direction: 'left' | 'right'): string
pad('hi', 10, 'left')
```

必要时，编译器会**拓宽**（_widen_，即转换为超类型）单元类型为原始类型，例如将 `"foo"` 拓宽为 `string`。这种情况会在使用可变性时发生，并可能对可变变量的某些用法造成阻碍：

```ts twoslash
// @errors: 2345
declare function pad(s: string, n: number, direction: 'left' | 'right'): string
// ---cut---
let s = 'right'
pad('hi', 10, s) // error: 'string' is not assignable to '"left" | "right"'
```

错误发生的具体原因如下：

- `"right": "right"`
- `s: string`，因为在赋值给可变变量时，`"right"` 会被拓宽为 `string`。
- `string` 无法赋值给 `"left" | "right"`

你可以通过为 `s` 添加类型注解来解决此问题，但这反过来又会阻止将非 `"left" | "right"` 类型的值赋给 `s`。

```ts twoslash
declare function pad(s: string, n: number, direction: 'left' | 'right'): string
// ---cut---
let s: 'left' | 'right' = 'right'
pad('hi', 10, s)
```

## 与 Haskell 相似的概念

### 上下文类型化

TypeScript 在一些显而易见的地方能够推断类型，例如变量声明：

```ts twoslash
let s = "I'm a string!"
```

但如果你曾经使用过其他 C 语法家族的语言，它在某些地方推断类型的方式可能会超出你的预期：

```ts twoslash
declare function map<T, U>(f: (t: T) => U, ts: T[]): U[]
let sns = map((n) => n.toString(), [1, 2, 3])
```

在此示例中，即使在调用前尚未推断出 `T` 和 `U`，`n` 也同样被推断为 `number`。事实上，在使用 `[1, 2, 3]` 推断出 `T=number` 之后，`n => n.toString()` 的返回值类型被用于推断 `U=string`，从而使得 `sns` 具有了类型 `string[]`。

需要注意的是，虽然推断可以在任意顺序下工作，但智能提示（IntelliSense）只能自左向右工作，因此 TypeScript 更推荐在声明 `map` 时将数组放在前面：

```ts twoslash
declare function map<T, U>(ts: T[], f: (t: T) => U): U[]
```

上下文类型化还可以递归地作用于对象字面量，以及通常会被推断为 `string` 或 `number` 的单元类型上。此外，它还能根据上下文推断返回值类型：

```ts twoslash
declare function run<T>(thunk: (t: T) => void): T
let i: { inference: string } = run((o) => {
  o.inference = 'INSERT STATE HERE'
})
```

`o` 的类型被确定为 `{ inference: string }`，因为：

1. 声明的初始化器受到声明类型 `{ inference: string }` 的上下文类型化约束。
2. 调用的返回类型会使用上下文类型进行推断，因此编译器推断出 `T={ inference: string }`。
3. 箭头函数使用上下文类型来为其参数提供类型，因此编译器推断出 `o: { inference: string }`。

而且这一推断是在你编写代码的同时实时进行的，因此在输入 `o.` 后，你就能获得属性 `inference` 的自动补全，以及真实程序中可能具有的其他属性补全。
综合来看，这一特性让 TypeScript 的类型推断机制看起来有点像合一类型推断（unifying type inference）引擎，但实际上并非如此。

### 类型别名

类型别名仅仅是别名，就像 Haskell 中的 `type` 一样。编译器会尝试在源代码中使用该别名的地方尽量复用这个别名名称，但并非总能成功。

```ts twoslash
type Size = [number, number]
let x: Size = [101.1, 999.9]
```

与 `newtype` 最接近的等价物是**带标签的交叉类型**（_tagged intersection_）：

```ts
type FString = string & { __compileTimeOnly: any }
```

`FString` 与普通字符串几乎无异，只是编译器会认为它包含一个实际上并不存在的名为 `__compileTimeOnly` 的属性。这意味着 `FString` 仍然可以赋值给 `string`，但反过来不行。

### 可辨识联合

与 `data` 最接近的等价物是具有辨识属性的类型联合，在 TypeScript 中通常被称为可辨识联合（discriminated unions）：

```ts
type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'square'; x: number }
  | { kind: 'triangle'; x: number; y: number }
```

与 Haskell 不同的是，这里的标签或辨识属性只是每个对象类型中的普通属性。每个变体都有一个同名属性，但具有不同的单元类型。这仍然是一个普通的联合类型；开头的 `|` 只是联合类型语法中的可选部分。你可以使用常规的 JavaScript 代码来区分联合中的成员：

```ts twoslash
type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'square'; x: number }
  | { kind: 'triangle'; x: number; y: number }

function area(s: Shape) {
  if (s.kind === 'circle') {
    return Math.PI * s.radius * s.radius
  } else if (s.kind === 'square') {
    return s.x * s.x
  } else {
    return (s.x * s.y) / 2
  }
}
```

注意，`area` 的返回类型被推断为 `number`，因为 TypeScript 知道该函数是全函数的（total）。如果某些变体未被覆盖，`area` 的返回类型将会变成 `number | undefined`。

此外，与 Haskell 不同的是，公共属性在任何联合中都会显现，因此你可以有效地同时区分联合中的多个成员：

```ts twoslash
type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'square'; x: number }
  | { kind: 'triangle'; x: number; y: number }
// ---cut---
function height(s: Shape) {
  if (s.kind === 'circle') {
    return 2 * s.radius
  } else {
    // s.kind: "square" | "triangle"
    return s.x
  }
}
```

### 类型参数

与大多数 C 衍生语言一样，TypeScript 要求声明类型参数：

```ts
function liftArray<T>(t: T): Array<T> {
  return [t]
}
```

类型参数没有大小写强制要求，但按照惯例通常使用单个大写字母。类型参数也可以被约束到某种类型，其行为有点类似于类型类约束（type class constraints）：

```ts
function firstish<T extends { length: number }>(t1: T, t2: T): T {
  return t1.length > t2.length ? t1 : t2
}
```

TypeScript 通常可以根据调用时的实参类型推断出类型实参，因此通常无需显式指定类型实参。

由于 TypeScript 采用结构化类型系统，它不像标称（nominal）系统那样需要频繁使用类型参数。具体来说，并不需要为了让函数具备多态性而引入类型参数。类型参数应该只用于**传递**（_propagate_）类型信息，例如约束多个参数为相同类型：

```ts
function length<T extends ArrayLike<unknown>>(t: T): number {}

function length(t: ArrayLike<unknown>): number {}
```

在第一个 `length` 中，T 是没有必要的；注意它只被引用了一次，并未用于约束返回值或其他参数的类型。

#### 高阶类型

TypeScript 不支持高阶类型（higher-kinded types），因此以下写法是不合法的：

```ts
function length<T extends ArrayLike<unknown>, U>(m: T<U>) {}
```

#### Point-free 编程

Point-free 编程（无点风格编程）——大量使用柯里化和函数组合——在 JavaScript 中是可行的，但可能会比较繁琐冗长。在 TypeScript 中，类型推断在 point-free 程序中往往容易失效，因此你最终不得不去指定类型参数而不是值参数。这样做的结果是代码过于冗长，以至于通常最好避免使用 point-free 编程风格。

### 模块系统

JavaScript 的现代模块语法与 Haskell 有点相似，不同之处在于任何包含 `import` 或 `export` 的文件都会被隐式视为一个模块：

```ts
import { value, Type } from 'npm-package'
import { other, Types } from './local-package'
import * as prefix from '../lib/third-package'
```

你也可以导入 CommonJS 模块——即使用 Node.js 模块系统编写的模块：

```ts
import f = require('single-function-package')
```

你可以使用导出列表进行导出：

```ts
export { f }

function f() {
  return g()
}
function g() {} // g is not exported
```

或者单独标记每个导出：

```ts
export function f() {
  return g()
}
function g() {}
```

后一种风格更常见，但这两种风格都受支持，甚至可以在同一个文件中混用。

### `readonly` 与 `const`

在 JavaScript 中，可变性（mutability）是默认行为，尽管它允许使用 `const` 声明变量来表明其**引用**（_reference_）是不可变的。但被引用的实体本身仍然是可变的：

```js
const a = [1, 2, 3]
a.push(102) // ):
a[0] = 101 // D:
```

TypeScript 还额外为属性提供了 `readonly` 修饰符：

```ts
interface Rx {
  readonly x: number
}
let rx: Rx = { x: 1 }
rx.x = 12 // error
```

它还内置了一个映射类型 `Readonly<T>`，可以将所有属性都转换为 `readonly`：

```ts
interface X {
  x: number
}
let rx: Readonly<X> = { x: 1 }
rx.x = 12 // error
```

此外，它还提供了专用的 `ReadonlyArray<T>` 类型，该类型移除了产生副作用的方法并禁止向数组索引写入内容，同时还为该类型提供了专门的语法简写：

```ts
let a: ReadonlyArray<number> = [1, 2, 3]
let b: readonly number[] = [1, 2, 3]
a.push(102) // error
b[0] = 101 // error
```

你还可以使用 const 断言（const-assertion），它作用于数组和对象字面量：

```ts
let a = [1, 2, 3] as const
a.push(102) // error
a[0] = 101 // error
```

然而，这些选项都不是默认行为，因此它们在 TypeScript 代码中并没有被始终如一地普遍使用。

### 下一步

本文档简要概述了你在日常代码中会用到的语法和类型。在此之后，你可以：

- [从头到尾](/docs/handbook/intro.html)阅读完整的手册
- 探索 [Playground 示例](/play#show-examples)
