---
display: 'Max Node Module JS Depth'
oneline: '指定在 `node_modules` 中检查 JavaScript 文件时所允许的最大文件夹深度。仅适用于 [`allowJs`](#allowJs)。'
---

在 `node_modules` 下搜索并加载 JavaScript 文件的最大依赖深度。

该标志仅在启用 [`allowJs`](#allowJs) 时可用；如果你希望 TypeScript 推断 `node_modules` 内部所有 JavaScript 代码的类型，可以使用该选项。

理想情况下，此选项应保持为 0（默认值），并使用 `.d.ts` 文件显式定义模块的结构。
然而，在某些场景下你可能希望牺牲编译速度和潜在的准确性来开启此选项。
