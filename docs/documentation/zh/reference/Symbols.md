---
title: Symbol
layout: docs
permalink: /zh/docs/handbook/symbols.html
oneline: 在 TypeScript 中使用 JavaScript 的 Symbol 原始数据类型
translatable: true
---

自 ECMAScript 2015 起，`symbol` 成为了一种原始数据类型，与 `number` 和 `string` 一样。

`symbol` 的值通过调用 `Symbol` 构造函数创建。

```ts
let sym1 = Symbol()

let sym2 = Symbol('key') // optional string key
```

Symbol 是不可变且唯一的。

```ts
let sym2 = Symbol('key')
let sym3 = Symbol('key')

sym2 === sym3 // false, symbols are unique
```

与字符串一样，Symbol 也可以用作对象属性的键。

```ts
const sym = Symbol()

let obj = {
  [sym]: 'value',
}

console.log(obj[sym]) // "value"
```

Symbol 还可以与计算属性声明结合使用，以声明对象属性和类成员。

```ts
const getClassNameSymbol = Symbol()

class C {
  [getClassNameSymbol]() {
    return 'C'
  }
}

let c = new C()
let className = c[getClassNameSymbol]() // "C"
```

## `unique symbol`

为了能够将 Symbol 视为唯一的字面量，TypeScript 提供了一种特殊的类型 `unique symbol`。`unique symbol` 是 `symbol` 的子类型，并且只能通过调用 `Symbol()` 或 `Symbol.for()`，或者通过显式类型注解产生。该类型只允许在 `const` 声明和 `readonly static` 属性上使用；若要引用特定的唯一 Symbol，必须使用 `typeof` 运算符。对某个唯一 Symbol 的每次引用，都代表着绑定到该特定声明的完全唯一的身份标识。

```ts twoslash
// @errors: 1332
declare const sym1: unique symbol

// sym2 can only be a constant reference.
let sym2: unique symbol = Symbol()

// Works - refers to a unique symbol, but its identity is tied to 'sym1'.
let sym3: typeof sym1 = sym1

// Also works.
class C {
  static readonly StaticSymbol: unique symbol = Symbol()
}
```

因为每个 `unique symbol` 都具有完全独立的身份标识，所以任意两个 `unique symbol` 类型之间既不能互相赋值，也不能互相比较。

```ts twoslash
// @errors: 2367
const sym2 = Symbol()
const sym3 = Symbol()

if (sym2 === sym3) {
  // ...
}
```

## 内置 Symbol（Well-known Symbols）

除了用户自定义的 Symbol，JavaScript 还内置了一系列熟知的 Symbol。
内置 Symbol 用于表示语言内部的行为。

以下是常用内置 Symbol 的列表：

### `Symbol.asyncIterator`

一个返回对象异步迭代器的方法，可与 for await..of 循环配合使用。

### `Symbol.hasInstance`

一个用于判断构造函数对象是否将某个对象识别为其自身实例的方法。由 instanceof 运算符的语义调用。

### `Symbol.isConcatSpreadable`

一个布尔值，表示对象在由 Array.prototype.concat 拼接时，是否应展开为其数组元素。

### `Symbol.iterator`

一个返回对象默认迭代器的方法。由 for-of 语句的语义调用。

### `Symbol.match`

一个正则表达式方法，用于将正则表达式与字符串进行匹配。由 `String.prototype.match` 方法调用。

### `Symbol.replace`

一个正则表达式方法，用于替换字符串中匹配的子串。由 `String.prototype.replace` 方法调用。

### `Symbol.search`

一个正则表达式方法，用于返回字符串中与正则表达式匹配的索引位置。由 `String.prototype.search` 方法调用。

### `Symbol.species`

一个函数值属性，该函数即用于创建派生对象的构造函数。

### `Symbol.split`

一个正则表达式方法，用于在匹配正则表达式的索引处拆分字符串。
由 `String.prototype.split` 方法调用。

### `Symbol.toPrimitive`

一个将对象转换为相应原始值的方法。
由 `ToPrimitive` 抽象操作调用。

### `Symbol.toStringTag`

一个字符串值，用于创建对象的默认字符串描述。
由内置方法 `Object.prototype.toString` 调用。

### `Symbol.unscopables`

一个对象，其自身属性名将被关联对象的 'with' 环境绑定排除在外。
