---
display: 'Composite'
oneline: '启用相关约束，以便 TypeScript 项目可以配合项目引用（Project References）使用。'
---

`composite` 选项会强制施加若干约束，使构建工具（包括处于 `--build` 模式下的 TypeScript 自身）能够快速判断项目是否已经被构建。

启用此设置后：

- 如果未显式设置 [`rootDir`](#rootDir)，其默认值为包含 `tsconfig.json` 文件的目录。

- 所有实现文件必须匹配 [`include`](#include) 模式或列在 [`files`](#files) 数组中。如果违反此约束，`tsc` 会提示哪些文件未被指定。

- [`declaration`](#declaration) 默认值为 `true`。

你可以在[手册](https://www.typescriptlang.org/docs/handbook/project-references.html)中找到有关 TypeScript 项目的文档。
