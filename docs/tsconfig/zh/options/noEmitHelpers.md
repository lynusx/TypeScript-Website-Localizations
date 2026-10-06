---
display: 'No Emit Helpers'
oneline: '禁用在编译输出中生成类似 `__extends` 的自定义辅助函数。'
---

除了使用 [`importHelpers`](#importHelpers) 导入辅助函数外，你还可以在全局作用域中为你所使用的辅助函数提供实现，并彻底关闭辅助函数的生成。

例如，在 ES5 中使用如下 `async` 函数需要类似 `await` 和类似 `generator` 的辅助函数才能运行：

```ts twoslash
const getAPI = async (url: string) => {
  // 请求 API
  return {}
}
```

这会生成相当多的 JavaScript 代码：

```ts twoslash
// @showEmit
// @target: ES5
const getAPI = async (url: string) => {
  // 请求 API
  return {}
}
```

通过此标志，可以将它们替换为你自定义的全局实现：

```ts twoslash
// @showEmit
// @target: ES5
// @noEmitHelpers
const getAPI = async (url: string) => {
  // 请求 API
  return {}
}
```
