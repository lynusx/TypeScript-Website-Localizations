---
display: 'TS Build Info File'
oneline: '用于存储 `.tsbuildinfo` 增量构建信息的文件。'
---

该设置用于指定一个文件，用于存储作为复合项目（composite projects）一部分的增量编译信息，从而加快大型 TypeScript 代码库的构建速度。你可以在[用户手册](/docs/handbook/project-references.html)中阅读关于复合项目的更多内容。

默认值取决于其他设置的组合：

- 如果设置了 `outFile`，默认值为 `<outFile>.tsbuildinfo`。
- 如果同时设置了 `rootDir` 和 `outDir`，则文件为 `<outDir>/<从 rootDir 到配置文件的相对路径>/<配置文件名>.tsbuildinfo`。
  例如，如果 `rootDir` 为 `src`，`outDir` 为 `dest`，配置文件为
  `./tsconfig.json`，那么默认值为 `./tsconfig.tsbuildinfo`，
  因为从 `src/` 到 `./tsconfig.json` 的相对路径是 `../`。
- 如果设置了 `outDir`，默认值为 `<outDir>/<配置文件名>.tsbuildInfo`。
- 否则，默认值为 `<配置文件名>.tsbuildInfo`。
