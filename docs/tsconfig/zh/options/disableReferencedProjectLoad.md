---
display: 'Disable Referenced Project Load'
oneline: '减少 TypeScript 自动加载的项目数量。'
---

在多项目的 TypeScript 程序中，TypeScript 会将所有可用项目加载到内存中，以便为需要完整知识图谱的编辑器操作（例如“查找所有引用”）提供准确的结果。

如果你的项目规模较大，可以使用标志 `disableReferencedProjectLoad` 来禁用所有项目的自动加载。这样，项目只有在你在编辑器中打开文件时才会动态加载。
