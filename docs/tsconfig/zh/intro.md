---
header: TSConfig 参考手册简介
firstLine: 目录中包含 TSConfig 文件，表明该目录是 TypeScript 或 JavaScript 项目的根目录...
---

目录中包含 TSConfig 文件，表明该目录是 TypeScript 或 JavaScript 项目的根目录。
TSConfig 文件可以是 `tsconfig.json` 或 `jsconfig.json`，两者支持相同的配置项集合。

本页面涵盖了 TSConfig 文件中可用的所有不同选项。这里包含 100 多个配置项，本页并非为了从头读到尾而设计，而是分为五个主要部分：

- 按类别整理的所有编译器标志概览
- 用于告知 TypeScript 哪些文件可用的[根字段](#Top%20Level)
- [`compilerOptions`](#compilerOptions) 字段，占据了本文档的大部分篇幅
- 用于调整监视模式的 [`watchOptions`](#watchOptions) 字段
- 用于调整如何向 JavaScript 项目添加类型的 [`typeAcquisition`](#typeAcquisition) 字段

如果你是从零开始创建 TSConfig，可以考虑使用 `tsc --init` 进行初始化，或使用推荐的 [TSConfig base](https://github.com/tsconfig/bases#centralized-recommendations-for-tsconfig-bases)。
