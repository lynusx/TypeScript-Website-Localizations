---
display: 'No Resolve'
oneline: '禁止通过 `import`、`require` 或 `<reference>` 增加 TypeScript 应添加到项目中的文件数量。'
---

默认情况下，TypeScript 会检查初始文件集中的 `import` 和 `<reference` 指令，并将解析出的这些文件添加到程序中。

如果设置了 `noResolve`，则不会执行该过程。
不过，系统仍会检查 `import` 语句是否能解析为有效的模块，因此你需要确保通过其他方式满足该条件。
