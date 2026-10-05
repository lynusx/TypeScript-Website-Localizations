---
display: 'Exclude'
oneline: '过滤 [`include`](#include) 选项匹配到的结果。'
---

指定解析 [`include`](#include) 时应跳过的文件名或匹配模式数组。

**重要提示**：`exclude` *仅*改变因 [`include`](#include) 设置而包含的文件。
即使某个文件在 `exclude` 中指定，它仍可能因代码中的 `import` 语句、`types` 引入、`/// <reference` 指令，或被列在 [`files`](#files) 列表中而成为代码库的一部分。

它并不是一种**阻止**文件被包含进代码库的机制 —— 它仅仅改变了 [`include`](#include) 设置所查找到的内容。
