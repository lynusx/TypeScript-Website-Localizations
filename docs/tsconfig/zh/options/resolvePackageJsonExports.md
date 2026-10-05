---
display: 'Resolve package.json Exports'
oneline: "在解析包导入时使用 package.json 的 'exports' 字段。"
---

`--resolvePackageJsonExports` 强制 TypeScript 在从 `node_modules` 中读取包时，必须查阅 [`package.json` 文件的 `exports` 字段](https://nodejs.org/api/packages.html#exports)。

当 [`--moduleResolution`](#moduleResolution) 选项为 `node16`、`nodenext` 和 `bundler` 时，该选项默认为 `true`。
