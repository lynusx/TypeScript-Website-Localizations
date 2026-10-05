---
display: 'Import Helpers'
oneline: '允许在每个项目中从 tslib 统一导入辅助函数，而不是在每个文件中重复包含它们。'
---

对于某些降级操作，TypeScript 会使用一些辅助代码来处理类继承、数组或对象展开以及异步操作等。
默认情况下，这些辅助函数会被插入到使用它们的文件中。
如果同一个辅助函数在许多不同的模块中被使用，这可能会导致代码重复。

如果开启了 `importHelpers` 标志，这些辅助函数将改为从 [tslib](https://www.npmjs.com/package/tslib) 模块中导入。
你需要确保在运行时能够导入 `tslib` 模块。
这仅对模块生效；全局脚本文件不会尝试导入模块。

例如，对于以下 TypeScript 代码：

```ts
export function fn(arr: number[]) {
  const arr2 = [1, ...arr]
}
```

开启 [`downlevelIteration`](#downlevelIteration) 且 `importHelpers` 仍为 false 时：

```ts twoslash
// @showEmit
// @target: ES5
// @downleveliteration
export function fn(arr: number[]) {
  const arr2 = [1, ...arr]
}
```

接着同时开启 [`downlevelIteration`](#downlevelIteration) 和 `importHelpers`：

```ts twoslash
// @showEmit
// @target: ES5
// @downleveliteration
// @importhelpers
// @noErrors
export function fn(arr: number[]) {
  const arr2 = [1, ...arr]
}
```

当你自行提供这些函数的实现时，可以使用 [`noEmitHelpers`](#noEmitHelpers)。
