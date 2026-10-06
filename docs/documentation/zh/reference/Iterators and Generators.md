---
title: 迭代器和生成器
layout: docs
permalink: /zh/docs/handbook/iterators-and-generators.html
oneline: TypeScript 中迭代器和生成器的工作原理
translatable: true
---

## 可迭代对象（Iterables）

当一个对象实现了 [`Symbol.iterator`](symbols.html#symboliterator) 属性时，该对象就被视为可迭代的。
某些内置类型（如 `Array`、`Map`、`Set`、`String`、`Int32Array`、`Uint32Array` 等）已经实现了它们的 `Symbol.iterator` 属性。
对象上的 `Symbol.iterator` 函数负责返回用于迭代的值的列表。

### `Iterable` 接口

如果我们想要接收上述可迭代的类型，可以使用 `Iterable` 类型。示例如下：

```ts
function toArray<X>(xs: Iterable<X>): X[] {
  return [...xs]
}
```

### `for..of` 语句

`for..of` 循环遍历一个可迭代对象，并调用该对象上的 `Symbol.iterator` 属性。
以下是在数组上使用 `for..of` 循环的简单示例：

```ts
let someArray = [1, 'string', false]

for (let entry of someArray) {
  console.log(entry) // 1, "string", false
}
```

### `for..of` 与 `for..in` 语句

`for..of` 和 `for..in` 语句都可以遍历列表，但所遍历的值却有所不同：`for..in` 返回被迭代对象上的*键*（keys）的列表，而 `for..of` 则返回被迭代对象的数值属性的*值*（values）列表。

以下示例演示了这一区别：

```ts
let list = [4, 5, 6]

for (let i in list) {
  console.log(i) // "0", "1", "2",
}

for (let i of list) {
  console.log(i) // 4, 5, 6
}
```

另一个区别是 `for..in` 可以操作任何对象，它作为一种检查对象属性的方式；而 `for..of` 则主要关注可迭代对象的值。像 `Map` 和 `Set` 这样的内置对象实现了 `Symbol.iterator` 属性，从而允许访问其存储的值。

```ts
let pets = new Set(['Cat', 'Dog', 'Hamster'])
pets['species'] = 'mammals'

for (let pet in pets) {
  console.log(pet) // "species"
}

for (let pet of pets) {
  console.log(pet) // "Cat", "Dog", "Hamster"
}
```

### 代码生成

#### 目标为 ES5

当编译目标为兼容 ES5 的引擎时，迭代器仅允许用于 `Array` 类型的值。
在非 Array 值上使用 `for..of` 循环将会报错，即使这些非 Array 值实现了 `Symbol.iterator` 属性也是如此。

编译器会为 `for..of` 循环生成一个简单的 `for` 循环，例如：

```ts
let numbers = [1, 2, 3]
for (let num of numbers) {
  console.log(num)
}
```

将生成为：

```js
var numbers = [1, 2, 3]
for (var _i = 0; _i < numbers.length; _i++) {
  var num = numbers[_i]
  console.log(num)
}
```

#### 目标为 ECMAScript 2015 及更高版本

当编译目标为兼容 ECMAScript 2015 的引擎时，编译器将生成针对引擎内置迭代器实现的 `for..of` 循环。
