---
display: 'Resolve package.json Imports'
oneline: "在解析导入时使用 package.json 的 'imports' 字段。"
---

`--resolvePackageJsonImports` 强制 TypeScript 在祖先目录包含 `package.json` 的文件中执行以 `#` 开头的查找时，查阅 [`package.json` 文件的 `imports` 字段](https://nodejs.org/api/packages.html#imports)。

当 [`--moduleResolution`](#moduleResolution) 设置为 `node16`、`nodenext` 或 `bundler` 时，此选项默认值为 `true`。
