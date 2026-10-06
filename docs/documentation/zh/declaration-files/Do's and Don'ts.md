---
title: 最佳实践（Do's and Don'ts）
layout: docs
permalink: /zh/docs/handbook/declaration-files/do-s-and-don-ts.html
oneline: '编写 d.ts 声明文件的建议'
---

## 通用类型

### `Number`、`String`、`Boolean`、`Symbol` 和 `Object`

❌ **切勿**使用 `Number`、`String`、`Boolean`、`Symbol` 或 `Object` 类型。
这些类型指的是非原始的装箱对象（boxed objects），在 JavaScript 代码中几乎从不会被恰当使用。

```ts
/* 错误 */
function reverse(s: String): String
```

✅ **应当**使用 `number`、`string`、`boolean` 和 `symbol` 类型。

```ts
/* 正确 */
function reverse(s: string): string
```

应该使用非原始的 `object` 类型（[TypeScript 2.2 中新增](../release-notes/typescript-2-2.html#object-type)）来代替 `Object`。

### 泛型

❌ **切勿**定义从未使用其类型参数的泛型类型。
详情请参阅 [TypeScript 常见问题解答页面](https://github.com/Microsoft/TypeScript/wiki/FAQ#why-doesnt-type-inference-work-on-this-interface-interface-foot--)。

### any

❌ **切勿**将 `any` 用作类型，除非你正在将 JavaScript 项目迁移到 TypeScript 的过程中。编译器在*实质上*会将 `any` 视为“请对该内容关闭类型检查”。这类似于在变量的每次使用处都加上 `@ts-ignore` 注释。当你最初将 JavaScript 项目迁移到 TypeScript 时，这可能会很有帮助，因为你可以将尚未迁移的内容类型设置为 `any`；但在一个纯 TypeScript 项目中，你这样做会禁用程序中所有使用它的部分的类型检查。

如果你不知道希望接收什么类型，或者希望接收任意内容因为你只是盲目地传递它而不与其交互，可以使用 [`unknown`](/play/#example/unknown-and-never)。

<!-- TODO: More -->

## 回调函数类型

### 回调函数的返回类型

<!-- TODO: Reword; these examples make no sense in the context of a declaration file -->

❌ **切勿**对返回值会被忽略的回调函数使用 `any` 返回类型：

```ts
/* 错误 */
function fn(x: () => any) {
  x()
}
```

✅ **应当**对返回值会被忽略的回调函数使用 `void` 返回类型：

```ts
/* 正确 */
function fn(x: () => void) {
  x()
}
```

❔ **原因：** 使用 `void` 更安全，因为它可以防止你不经检查就意外使用 `x` 的返回值：

```ts
function fn(x: () => void) {
  var k = x() // 糟糕！本意是做其他事情
  k.doSomething() // 错误，但如果返回类型是 'any' 则不会报错
}
```

### 回调函数中的可选参数

❌ **切勿**在回调函数中使用可选参数，除非你确有此意：

```ts
/* 错误 */
interface Fetcher {
  getObject(done: (data: unknown, elapsedTime?: number) => void): void
}
```

这具有非常明确的含义：`done` 回调可能以 1 个参数调用，也可能以 2 个参数调用。
作者的本意可能是想表达回调函数可以不关心 `elapsedTime` 参数，
但要实现这一点并不需要将该参数设为可选——
在任何情况下，传入一个接受更少参数的回调函数在语法上始终是合法的。

✅ **应当**将回调函数的参数编写为非可选（必选）参数：

```ts
/* 正确 */
interface Fetcher {
  getObject(done: (data: unknown, elapsedTime: number) => void): void
}
```

### 重载与回调函数

❌ **切勿**编写仅在回调函数参数个数（arity）上有所差异的独立重载：

```ts
/* 错误 */
declare function beforeAll(action: () => void, timeout?: number): void
declare function beforeAll(
  action: (done: DoneFn) => void,
  timeout?: number,
): void
```

✅ **应当**使用最大参数个数编写单个重载：

```ts
/* 正确 */
declare function beforeAll(
  action: (done: DoneFn) => void,
  timeout?: number,
): void
```

❔ **原因：** 回调函数忽略某个参数始终是合法的，因此不需要较短的重载。
优先提供参数较短的回调重载，可能会导致类型错误的函数被成功传入，因为它们命中了第一个重载。

## 函数重载

### 排序

❌ **切勿**将更通用的重载放在更具体的重载之前：

```ts
/* 错误 */
declare function fn(x: unknown): unknown
declare function fn(x: HTMLElement): number
declare function fn(x: HTMLDivElement): string

var myElem: HTMLDivElement
var x = fn(myElem) // x: unknown，怎么回事？
```

✅ **应当**对重载进行排序，将更通用的签名放在更具体的签名之后：

```ts
/* 正确 */
declare function fn(x: HTMLDivElement): string
declare function fn(x: HTMLElement): number
declare function fn(x: unknown): unknown

var myElem: HTMLDivElement
var x = fn(myElem) // x: string，:)
```

❔ **原因：** TypeScript 在解析函数调用时会选择*首个匹配的重载*。
当较早的重载比后面的重载“更通用”时，后面的重载实际上会被隐藏且无法被调用。

### 使用可选参数

❌ **切勿**编写仅在末尾参数上有所差异的多个重载：

```ts
/* 错误 */
interface Example {
  diff(one: string): number
  diff(one: string, two: string): number
  diff(one: string, two: string, three: boolean): number
}
```

✅ **应当**尽可能使用可选参数：

```ts
/* 正确 */
interface Example {
  diff(one: string, two?: string, three?: boolean): number
}
```

请注意，只有当所有重载具有相同的返回类型时，才应该进行这种合并。

❔ **原因：** 这有两个重要的原因。

TypeScript 解析签名兼容性时，会检查源签名的参数是否可以调用目标签名的任意签名，
_且允许传入多余的参数_。
例如，只有当签名正确使用可选参数编写时，以下代码才会暴露出 bug：

```ts
function fn(x: (a: string, b: number, c: number) => void) {}
var x: Example
// 使用重载编写时正常 —— 使用了第一个重载
// 使用可选参数编写时正确报错
fn(x.diff)
```

第二个原因是当使用者开启 TypeScript 的“严格空值检查”（strict null checking）特性时。
因为未指定的参数在 JavaScript 中表现为 `undefined`，所以向带有可选参数的函数显式传递 `undefined` 通常是完全可行的。
例如，在 strict nulls 下，以下代码应当正常工作：

```ts
var x: Example
// 使用重载编写时会错误地报错，因为将 'undefined' 传递给了 'string'
// 使用可选参数编写时完全正常
x.diff('something', true ? undefined : 'hour')
```

### 使用联合类型

❌ **切勿**编写仅在某一参数位置上类型不同的重载：

```ts
/* 错误 */
interface Moment {
  utcOffset(): number
  utcOffset(b: number): Moment
  utcOffset(b: string): Moment
}
```

✅ **应当**尽可能使用联合类型：

```ts
/* 正确 */
interface Moment {
  utcOffset(): number
  utcOffset(b: number | string): Moment
}
```

请注意，这里我们没有将 `b` 设为可选，因为各签名的返回类型并不相同。

❔ **原因：** 这对于向你的函数“透传”（passing through）值的使用者来说非常重要：

```ts
function fn(x: string): Moment
function fn(x: number): Moment
function fn(x: number | string) {
  // 使用独立重载编写时会错误地报错
  // 使用联合类型编写时完全正常
  return moment().utcOffset(x)
}
```
