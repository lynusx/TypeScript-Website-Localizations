---
display: 'Allow Importing TS Extensions'
oneline: '允许导入语句中包含 TypeScript 文件扩展名。'
---

`--allowImportingTsExtensions` 允许 TypeScript 文件在相互导入时使用 TypeScript 特有的扩展名，如 `.ts`、`.mts` 或 `.tsx`。

仅当启用了 `--noEmit` 或 `--emitDeclarationOnly` 时才允许使用该标志，因为在 JavaScript 输出文件中，这些导入路径在运行时将无法解析。
这里的预期是你的解析器（例如打包工具、运行时或其他工具）能够让这些 `.ts` 文件之间的导入正常工作。
