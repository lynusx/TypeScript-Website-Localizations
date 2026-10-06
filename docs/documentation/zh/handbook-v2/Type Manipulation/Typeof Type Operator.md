---
title: typeof 类型运算符
layout: docs
permalink: /zh/docs/handbook/2/typeof-types.html
oneline: '在类型上下文中使用 typeof 运算符。'
---

## `typeof` 类型运算符

JavaScript 本身就包含可以在*表达式*上下文中使用的 `typeof` 运算符：

```ts twoslash
// Prints "string"
console.log(typeof 'Hello world')
```

TypeScript 则新增了一个可以在*类型*上下文中使用的 `typeof` 运算符，用于引用变量或属性的*类型*：

```ts twoslash
let s = 'hello'
let n: typeof s
//  ^?
```

这对于基础类型来说用处不大，但若与其他类型运算符结合使用，你可以通过 `typeof` 方便地表达许多模式。
例如，我们先来看预定义类型 `ReturnType<T>`。
它接收一个*函数类型*并生成其返回值类型：

```ts twoslash
type Predicate = (x: unknown) => boolean
type K = ReturnType<Predicate>
//   ^?
```

如果我们尝试在函数名上直接使用 `ReturnType`，就会看到一个具有启发性的错误提示：

```ts twoslash
// @errors: 2749
function f() {
  return { x: 10, y: 3 }
}
type P = ReturnType<f>
```

请记住，*值*和*类型*并不是同一回事。
要引用*值 `f`* 所拥有的*类型*，我们需要使用 `typeof`：

```ts twoslash
function f() {
  return { x: 10, y: 3 }
}
type P = ReturnType<typeof f>
//   ^?
```

### 限制

TypeScript 刻意限制了可以对其使用 `typeof` 的表达式种类。

具体而言，只允许在标识符（即变量名）或其属性上使用 `typeof`。
这有助于避免开发者陷入误以为代码会执行的困惑陷阱：

```ts twoslash
// @errors: 1005
declare const msgbox: (prompt: string) => boolean;
// type msgbox = any;
// ---cut---
// Meant to use = ReturnType<typeof msgbox>
let shouldContinue: typeof msgbox("Are you sure you want to continue?");
```
