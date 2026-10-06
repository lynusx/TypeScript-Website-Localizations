---
title: 在 JavaScript 文件中进行类型检查
layout: docs
permalink: /zh/docs/handbook/type-checking-javascript-files.html
oneline: 如何使用 TypeScript 为 JavaScript 文件添加类型检查
---

以下是 `.js` 文件与 `.ts` 文件在类型检查机制上的一些显著差异。

## 类属性推断自类体内的赋值语句

ES2015 没有在类上声明属性的语法。属性就像对象字面量一样是动态赋值的。

在 `.js` 文件中，编译器会根据类体内的属性赋值语句来推断属性。
属性的类型是在构造函数中赋予的类型，除非它未在构造函数中定义，或者构造函数中的类型为 `undefined` 或 `null`。
在这种情况下，属性的类型是这些赋值语句中所有右值类型的联合类型。
在构造函数中定义的属性总是被假定为存在，而仅在方法、getter 或 setter 中定义的属性则被视为可选属性。

```js twoslash
// @checkJs
// @errors: 2322
class C {
  constructor() {
    this.constructorOnly = 0
    this.constructorUnknown = undefined
  }
  method() {
    this.constructorOnly = false
    this.constructorUnknown = 'plunkbat' // 正常，constructorUnknown 的类型为 string | undefined
    this.methodOnly = 'ok' // 正常，但 methodOnly 也可以是 undefined
  }
  method2() {
    this.methodOnly = true // 同样正常，methodOnly 的类型为 string | boolean | undefined
  }
}
```

如果属性从未在类体内设置过，则它们被视为 `unknown`。
如果你的类中有只读的属性，可以在构造函数中添加声明并使用 JSDoc 进行类型注解来指定其类型。
如果属性稍后才会被初始化，你甚至不需要为其赋初始值：

```js twoslash
// @checkJs
// @errors: 2322
class C {
  constructor() {
    /** @type {number | undefined} */
    this.prop = undefined
    /** @type {number | undefined} */
    this.count
  }
}

let c = new C()
c.prop = 0 // 正常
c.count = 'string'
```

## 构造函数等价于类

在 ES2015 之前，JavaScript 使用构造函数代替类。
编译器支持这种模式，并将构造函数视为等价于 ES2015 的类。
上面介绍的属性推断规则完全同样适用。

```js twoslash
// @checkJs
// @errors: 2683 2322
function C() {
  this.constructorOnly = 0
  this.constructorUnknown = undefined
}
C.prototype.method = function () {
  this.constructorOnly = false
  this.constructorUnknown = 'plunkbat' // 正常，类型为 string | undefined
}
```

## 支持 CommonJS 模块

在 `.js` 文件中，TypeScript 支持 CommonJS 模块格式。
对 `exports` 和 `module.exports` 的赋值会被识别为导出声明。
同样，`require` 函数调用会被识别为模块导入。例如：

```js
// 等同于 `import module "fs"`
const fs = require('fs')

// 等同于 `export function readFile`
module.exports.readFile = function (f) {
  return fs.readFileSync(f)
}
```

JavaScript 中的模块支持在语法上比 TypeScript 的模块支持宽容得多。
支持大多数赋值与声明的组合方式。

## 类、函数和对象字面量都是命名空间

在 `.js` 文件中，类充当着命名空间的作用。
这可用于嵌套类，例如：

```js twoslash
class C {}
C.D = class {}
```

对于 ES2015 之前的代码，它也可以用于模拟静态方法：

```js twoslash
function Outer() {
  this.y = 2
}

Outer.Inner = function () {
  this.yy = 2
}

Outer.Inner()
```

它还可以用来创建简单的命名空间：

```js twoslash
var ns = {}
ns.C = class {}
ns.func = function () {}

ns
```

也允许其他变体形式：

```js twoslash
// IIFE
var ns = (function (n) {
  return n || {}
})()
ns.CONST = 1

// 默认指向 global
var assign =
  assign ||
  function () {
    // 代码写在这里
  }
assign.extra = 1
```

## 对象字面量是开放式的

在 `.ts` 文件中，用于初始化变量声明的对象字面量会将其类型赋予该声明。
不能添加未在原始字面量中指定的任何新成员。
在 `.js` 文件中，该规则被放宽了；对象字面量具有开放式类型（索引签名），允许添加和查找最初未定义的属性。
例如：

```js twoslash
var obj = { a: 1 }
obj.b = 2 // 允许
```

对象字面量的行为就像它们具有索引签名 `[x:string]: any` 一样，这使得它们可以被视为开放的映射（open map）而非封闭的对象。

与其他特殊的 JS 检查行为一样，可以通过为变量指定 JSDoc 类型来改变此行为。例如：

```js twoslash
// @checkJs
// @errors: 2339
/** @type {{a: number}} */
var obj = { a: 1 }
obj.b = 2
```

## null、undefined 和空数组初始化器的类型为 any 或 any[]

任何使用 `null` 或 `undefined` 进行初始化的变量、参数或属性，其类型都将为 `any`，即使开启了严格空值检查（strict null checks）也是如此。
任何使用 `[]` 进行初始化的变量、参数或属性，其类型都将为 `any[]`，即使开启了严格空值检查也是如此。
唯一的例外是前面介绍的具有多个初始化器的属性。

```js twoslash
function Foo(i = null) {
  if (!i) i = 1
  var j = undefined
  j = 2
  this.l = []
}

var foo = new Foo()
foo.l.push(foo.i)
foo.l.push('end')
```

## 函数参数默认是可选的

由于在 ES2015 之前的 JavaScript 中无法指定参数的可选性，因此 `.js` 文件中的所有函数参数都被视为可选参数。
允许在调用时传入少于声明数量的参数。

需要注意的是，调用函数时传入过多参数属于错误。

例如：

```js twoslash
// @checkJs
// @strict: false
// @errors: 7006 7006 2554
function bar(a, b) {
  console.log(a + ' ' + b)
}

bar(1) // 正常，第二个参数被视为可选参数
bar(1, 2)
bar(1, 2, 3) // 错误，参数过多
```

带有 JSDoc 注解的函数不受此规则限制。
可以使用 JSDoc 的可选参数语法（`[` `]`）来表示可选性。例如：

```js twoslash
/**
 * @param {string} [somebody] - 某人的姓名。
 */
function sayHello(somebody) {
  if (!somebody) {
    somebody = 'John Doe'
  }
  console.log('Hello ' + somebody)
}

sayHello()
```

## 根据 `arguments` 的使用推断变长参数声明

如果函数体内引用了 `arguments`，则隐式认为该函数具有变长参数（即 `(...arg: any[]) => any`）。可以使用 JSDoc 变长参数语法来指定参数的类型。

```js twoslash
/** @param {...number} args */
function sum(/* 数字 */) {
  var total = 0
  for (var i = 0; i < arguments.length; i++) {
    total += arguments[i]
  }
  return total
}
```

## 未指定的类型参数默认为 `any`

由于 JavaScript 中没有指定泛型类型参数的原生语法，未指定的类型参数默认为 `any`。

### 在 extends 子句中

例如，`React.Component` 被定义为具有两个类型参数：`Props` 和 `State`。
在 `.js` 文件中，无法直接在 extends 子句中合法地指定它们。默认情况下，类型参数将为 `any`：

```js
import { Component } from 'react'

class MyComponent extends Component {
  render() {
    this.props.b // 允许，因为 this.props 的类型为 any
  }
}
```

可以使用 JSDoc 的 `@augments` 来显式指定类型。例如：

```js
import { Component } from 'react'

/**
 * @augments {Component<{a: number}, State>}
 */
class MyComponent extends Component {
  render() {
    this.props.b // 错误：属性 b 不存在于 {a:number} 上
  }
}
```

### 在 JSDoc 引用中

JSDoc 中未指定的类型参数默认为 `any`：

```js twoslash
/** @type{Array} */
var x = []

x.push(1) // 正常
x.push('string') // 正常，x 的类型为 Array<any>

/** @type{Array.<number>} */
var y = []

y.push(1) // 正常
y.push('string') // 错误，string 不能赋值给 number
```

### 在函数调用中

对泛型函数的调用会使用传入的参数来推断类型参数。有时该过程无法推断出任何类型（主要是由于缺少推断来源）；在这些情况下，类型参数将默认为 `any`。例如：

```js
var p = new Promise((resolve, reject) => {
  reject()
})

p // Promise<any>;
```

要了解 JSDoc 中提供的全部功能，请参阅[参考文档](/docs/handbook/jsdoc-supported-types.html)。
