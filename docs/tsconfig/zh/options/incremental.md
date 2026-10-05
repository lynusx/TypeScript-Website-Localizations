---
display: 'Incremental'
oneline: '保存 .tsbuildinfo 文件以实现项目的增量编译。'
---

指示 TypeScript 将上一次编译的项目图信息保存到存储在磁盘上的文件中。
这会在与编译输出相同的目录中生成一系列 `.tsbuildinfo` 文件。这些文件在 JavaScript 运行时不会被使用，可以安全删除。你可以在 [3.4 发行说明](/docs/handbook/release-notes/typescript-3-4.html#faster-subsequent-builds-with-the---incremental-flag)中了解关于该标志的更多信息。

若要控制这些文件的生成目录，请使用配置选项 [`tsBuildInfoFile`](#tsBuildInfoFile)。
