---
title: TypeScript 1.4
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-1-4.html
oneline: TypeScript 1.4 发布说明
---

## 联合类型（Union types）

### 概述

联合类型是表达一个值可以是多种类型之一的强大方式。例如，你可能有一个用于运行程序的 API，它接受的命令行参数可以是 `string`、`string[]` 或返回 `string` 的函数。现在你可以这样编写：

```ts
interface RunOptions {
  program: string
  commandline: string[] | string | (() => string)
}
```

对联合类型的赋值非常直观——凡是可以赋值给联合类型中任一成员的值，都可以赋值给该联合类型：

```ts
var opts: RunOptions = /* ... */;
opts.commandline = '-hello world'; // OK
opts.commandline = ['-hello', 'world']; // OK
opts.commandline = [42]; // Error, number is not string or string[]
```

当读取联合类型的值时，你可以访问它们之间共享的所有属性：

```ts
if (opts.commandline.length === 0) {
  // OK, string and string[] both have 'length' property
  console.log("it's empty")
}
```

借助类型保护（Type Guards），你可以轻松处理联合类型的变量：

```ts
function formatCommandline(c: string | string[]) {
  if (typeof c === 'string') {
    return c.trim()
  } else {
    return c.join(' ')
  }
}
```

### 更严格的泛型调用

由于联合类型能够表示广泛的类型场景，我们决定提高某些泛型调用的严格程度。在此之前，像下面这样的代码会出人意料地编译通过且不报错：

```ts
function equal<T>(lhs: T, rhs: T): boolean {
  return lhs === rhs
}

// Previously: No error
// New behavior: Error, no best common type between 'string' and 'number'
var e = equal(42, 'hello')
```

有了联合类型之后，你现在可以在函数声明处和调用处分别指定所期望的行为：

```ts
// 'choose' function where types must match
function choose1<T>(a: T, b: T): T {
  return Math.random() > 0.5 ? a : b
}
var a = choose1('hello', 42) // Error
var b = choose1<string | number>('hello', 42) // OK

// 'choose' function where types need not match
function choose2<T, U>(a: T, b: U): T | U {
  return Math.random() > 0.5 ? a : b
}
var c = choose2('bar', 'foo') // OK, c: string
var d = choose2('hello', 42) // OK, d: string|number
```

### 更好的类型推断

在数组以及集合中可能包含多种类型值的其他场景下，联合类型也能带来更好的类型推断：

```ts
var x = [1, 'hello'] // x: Array<string|number>
x[0] = 'world' // OK
x[0] = false // Error, boolean is not string or number
```

## `let` 声明

在 JavaScript 中，`var` 声明会被“提升（hoist）”到所在封闭作用域的顶部。这可能会导致令人困惑的 bug：

```ts
console.log(x) // meant to write 'y' here
/* later in the same block */
var x = 'hello'
```

TypeScript 现已支持 ES6 新增的 `let` 关键字，它声明的变量具有更符合直觉的“块级（block）”语义。`let` 变量只能在其声明之后被引用，且其作用域限定在定义它的语法块内：

```ts
if (foo) {
  console.log(x) // Error, cannot refer to x before its declaration
  let x = 'hello'
} else {
  console.log(x) // Error, x is not declared in this block
}
```

`let` 仅在目标为 ECMAScript 6（`--target ES6`）时可用。

## `const` 声明

TypeScript 支持的另一种新的 ES6 声明类型是 `const`。`const` 变量不可被重新赋值，并且必须在声明处进行初始化。这对于在初始化后不希望其值发生改变的声明非常有用：

```ts
const halfPi = Math.PI / 2
halfPi = 2 // Error, can't assign to a `const`
```

`const` 仅在目标为 ECMAScript 6（`--target ES6`）时可用。

## 模板字符串

TypeScript 现已支持 ES6 模板字符串。这是一种在字符串中嵌入任意表达式的简便方式：

```ts
var name = 'TypeScript'
var greeting = `Hello, ${name}! Your name has ${name.length} characters`
```

当编译到低于 ES6 的目标版本时，字符串会被分解转换：

```js
var name = 'TypeScript!'
var greeting =
  'Hello, ' + name + '! Your name has ' + name.length + ' characters'
```

## 类型保护（Type Guards）

在 JavaScript 中，一种常见的模式是使用 `typeof` 或 `instanceof` 在运行时检查表达式的类型。TypeScript 现在能够理解这些条件，并在 `if` 块中相应地改变类型推断。

使用 `typeof` 检查变量：

```ts
var x: any = /* ... */;
if(typeof x === 'string') {
    console.log(x.subtr(1)); // Error, 'subtr' does not exist on 'string'
}
// x is still any here
x.unknown(); // OK
```

在联合类型和 `else` 分支中使用 `typeof`：

```ts
var x: string | HTMLElement = /* ... */;
if(typeof x === 'string') {
    // x is string here, as shown above
}
else {
    // x is HTMLElement here
    console.log(x.innerHTML);
}
```

在类和联合类型中使用 `instanceof`：

```ts
class Dog { woof() { } }
class Cat { meow() { } }
var pet: Dog|Cat = /* ... */;
if (pet instanceof Dog) {
    pet.woof(); // OK
}
else {
    pet.woof(); // Error
}
```

## 类型别名

现在可以使用 `type` 关键字为类型定义一个*别名*：

```ts
type PrimitiveArray = Array<string | number | boolean>
type MyNumber = number
type NgScope = ng.IScope
type Callback = () => void
```

类型别名与其原始类型完全相同；它们只是替代名称。

## `const enum`（完全内联的枚举）

枚举非常有用，但某些程序实际上并不需要生成的代码，而是能从将所有枚举成员实例直接内联为其数值对等项中获益。新增的 `const enum` 声明在类型安全方面的表现与常规 `enum` 相同，但在编译时会被完全擦除。

```ts
const enum Suit {
  Clubs,
  Diamonds,
  Hearts,
  Spades,
}
var d = Suit.Diamonds
```

编译后的代码恰好为：

```js
var d = 1
```

TypeScript 现在还会在可能的情况下计算枚举值：

```ts
enum MyFlags {
  None = 0,
  Neat = 1,
  Cool = 2,
  Awesome = 4,
  Best = Neat | Cool | Awesome,
}
var b = MyFlags.Best // emits var b = 7;
```

## `-noEmitOnError` 命令行选项

TypeScript 编译器的默认行为是：即使存在类型错误（例如尝试将 `string` 赋值给 `number`），仍然会生成 .js 文件。在构建服务器或其他只希望得到“无报错”构建产物的场景中，这种行为可能并非所需。新增的 [`noEmitOnError`](/tsconfig#noEmitOnError) 标志可以阻止编译器在出现任何错误时生成 .js 代码。

现在该选项已成为 MSBuild 项目的默认配置；由于产物仅在无报错的构建中生成，这使得 MSBuild 增量构建能够按预期工作。

## AMD 模块名称

默认情况下，生成的 AMD 模块是匿名的。当使用其他工具（如打包工具 `r.js`）处理生成的模块时，这可能会引发问题。

新增的 `amd-module name` 标签允许向编译器传递一个可选的模块名称：

```ts
//// [amdModule.ts]
///<amd-module name='NamedModule'/>
export class C {}
```

这会在调用 AMD `define` 时将模块命名为 `NamedModule`：

```js
//// [amdModule.js]
define('NamedModule', ['require', 'exports'], function (require, exports) {
  var C = (function () {
    function C() {}
    return C
  })()
  exports.C = C
})
```
