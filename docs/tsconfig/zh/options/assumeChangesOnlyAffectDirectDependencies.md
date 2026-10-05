---
display: 'Assume Changes Only Affect Direct Dependencies'
oneline: '在使用 [`incremental`](#incremental) 和 `watch` 模式的项目中，假定文件内部的变更只会影响直接依赖该文件的文件。'
---

启用此选项后，TypeScript 将避免重新检查/重新构建所有可能受影响的文件，而只重新检查/重新构建发生变更的文件以及直接导入它们的文件。

这可以被看作是监视算法的一种“求快但不严谨（fast & loose）”的实现方式，它能大幅缩短增量重新构建的时间，但代价是偶尔需要运行一次完整构建以获取全部的编译器错误信息。
