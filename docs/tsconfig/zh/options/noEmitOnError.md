---
display: 'No Emit On Error'
oneline: '报告任何类型检查错误时不生成文件。'
---

如果报告了任何错误，则不生成编译器输出文件，如 JavaScript 源代码、Source Map 或声明文件。

默认值为 `false`，这使得在类似 watch 的环境中配合 TypeScript 开发更为方便 —— 在确保所有错误都得到解决之前，你可能希望先在其他环境中查看代码更改的结果。
