---
display: 'Downlevel Iteration'
oneline: '为迭代操作生成更符合规范，但代码更繁琐且性能较低的 JavaScript。'
---

“降级（Downleveling）”是 TypeScript 中将代码转译为更低版本 JavaScript 的术语。
该标志用于在旧版 JavaScript 运行时中，更准确地实现现代 JavaScript 对新特性的迭代方式。

ECMAScript 6 增加了多种新的迭代原语：`for / of` 循环（`for (el of arr)`）、数组展开（`[a, ...b]`）、参数展开（`fn(...args)`）以及 `Symbol.iterator`。
如果运行时存在 `Symbol.iterator` 的实现，`downlevelIteration` 可以在 ES5 环境中更准确地使用这些迭代原语。

#### 示例：对 `for / of` 的影响

对于以下 TypeScript 代码：

```ts twoslash
const str = 'Hello!'
for (const s of str) {
  console.log(s)
}
```

在未启用 `downlevelIteration` 时，对任何对象使用的 `for / of` 循环都会被降级为传统的 `for` 循环：

```ts twoslash
// @target: ES5
// @showEmit
const str = 'Hello!'
for (const s of str) {
  console.log(s)
}
```

这通常符合人们的预期，但并不 100% 符合 ECMAScript 的迭代协议。
某些字符串（例如 emoji 表情 😜）的 `.length` 为 2（甚至更长！），但在 `for-of` 循环中应该作为一个单独的单元进行迭代。
详细解释请参阅 [Jonathan New 的这篇博文](https://blog.jonnew.com/posts/poo-dot-length-equals-two)。

启用 `downlevelIteration` 后，TypeScript 将使用一个辅助函数来检查是否存在 `Symbol.iterator` 实现（无论是原生还是 polyfill）。
如果缺少该实现，将回退到基于索引的迭代。

```ts twoslash
// @target: ES5
// @downlevelIteration
// @showEmit
const str = 'Hello!'
for (const s of str) {
  console.log(s)
}
```

你还可以配合 [`importHelpers`](#importHelpers) 使用 [tslib](https://www.npmjs.com/package/tslib)，以减少内联生成的 JavaScript 代码量：

```ts twoslash
// @target: ES5
// @downlevelIteration
// @importHelpers
// @showEmit
const str = 'Hello!'
for (const s of str) {
  console.log(s)
}
```

**注意：** 如果运行时环境中不存在 `Symbol.iterator`，启用 `downlevelIteration` 并不会提高规范符合性。

#### 示例：对数组展开的影响

这是一个数组展开：

```js
// 创建一个新数组，其元素为 1，后跟 arr2 的元素
const arr = [1, ...arr2]
```

从描述上看，降级到 ES5 似乎很简单：

```js
// 效果相同，对吧？
const arr = [1].concat(arr2)
```

然而，在某些罕见情况下，两者会表现出明显的差异。

例如，如果源数组缺少一个或多个元素（即存在空洞），展开语法会将每个空元素替换为 `undefined`，而 `.concat` 则会保留空位。

```js
// 创建一个索引 1 处缺失元素的数组
let arrayWithHole = ['a', , 'c']
let spread = [...arrayWithHole]
let concatenated = [].concat(arrayWithHole)

console.log(arrayWithHole)
// [ 'a', <1 个空项>, 'c' ]
console.log(spread)
// [ 'a', undefined, 'c' ]
console.log(concatenated)
// [ 'a', <1 个空项>, 'c' ]
```

与 `for / of` 一样，`downlevelIteration` 会使用 `Symbol.iterator`（如果存在）来更准确地模拟 ES 6 行为。
