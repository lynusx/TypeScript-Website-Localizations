---
display: 'Imports Not Used As Values'
oneline: '指定仅用于类型的导入的生成与检查行为。'
---

已废弃，推荐使用 [`verbatimModuleSyntax`](#verbatimModuleSyntax)。

该标志控制 `import` 的工作方式，有 3 个不同的选项：

- `remove`：默认行为，丢弃仅引用类型的 `import` 语句。

- `preserve`：保留所有值或类型从未被使用的 `import` 语句。这会导致导入及其副作用被保留。

- `error`：保留所有导入（与 preserve 选项相同），但当某个值导入仅用作类型时报错。如果你希望确保不会意外导入任何值，同时明确显式声明副作用导入，这会很有用。

该标志之所以有效，是因为你可以使用 `import type` 显式创建绝不应生成到 JavaScript 中的 `import` 语句。
