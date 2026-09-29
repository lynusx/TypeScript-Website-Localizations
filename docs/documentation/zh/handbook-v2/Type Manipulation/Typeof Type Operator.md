---
title: typeof 类型操作符
layout: docs
permalink: /zh/docs/handbook/2/typeof-types.html
oneline: '在类型上下文中使用 typeof 操作符。'
---

## `typeof` 类型操作符

JavaScript 已经提供了一个可以在表达式上下文中使用的 `typeof` 操作符：

```ts twoslash
// 打印 'string'
console.log(typeof 'Hello world')
```

TypeScript 增加了一个可以在*类型*上下文中使用的 `typeof` 运算符，用于引用变量或属性的 _类型_：

```ts twoslash
let s = 'hello'
let n: typeof s
//  ^?
```

这对基本类型来说用处不大，但与其他类型运算符结合使用时，你可以用 `typeof` 方便地表达许多模式。举个例子，我们先看看预定义类型 `ReturnType<T>`。它接受一个 _函数类型_ 并产生其返回类型：

```ts twoslash
type Predicate = (x: unknown) => boolean
type K = ReturnType<Predicate>
//   ^?
```

如果尝试将 `ReturnType` 直接用于函数名，会看到一个有启发性的错误：

```ts twoslash
// @errors: 2749
function f() {
  return { x: 10, y: 3 }
}
type P = ReturnType<f>
```

请记住，_值_ 和 _类型_ 不是一回事。要引用 _值 `f`_ 所具有的 _类型_，我们使用 `typeof`：

```ts twoslash
function f() {
  return { x: 10, y: 3 }
}
type P = ReturnType<typeof f>
//   ^?
```

### 限制

TypeScript 有意限制了可以对哪些表达式使用 `typeof`。

具体来说，只允许对标识符（即变量名）或其属性使用 `typeof`。这有助于避免一种容易混淆的陷阱：你以为写下的代码会执行，但实际上并没有：

```ts twoslash
// @errors: 1005
declare const msgbox: (prompt: string) => boolean;
// type msgbox = any;
// ---cut---
// 意图是使用 = ReturnType<typeof msgbox>
let shouldContinue: typeof msgbox("Are you sure you want to continue?");
```
