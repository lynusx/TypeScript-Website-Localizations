---
display: 'No Emit'
oneline: '禁用在编译中输出文件。'
---

不输出编译器生成的文件，例如 JavaScript 源代码、Source Map 或声明文件。

这为 [Babel](https://babeljs.io) 或 [swc](https://github.com/swc-project/swc) 等其他工具腾出空间，由它们负责将 TypeScript 文件转换为可以在 JavaScript 环境中运行的文件。

此时你可以将 TypeScript 用作提供编辑器集成以及源码类型检查的工具。
