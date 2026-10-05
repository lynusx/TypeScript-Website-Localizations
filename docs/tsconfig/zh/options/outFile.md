---
display: 'Out File'
oneline: '指定一个将所有输出打包合并为一个 JavaScript 文件的路径。如果 [`declaration`](#declaration) 为 true，还将指定一个打包所有 .d.ts 输出的文件。'
---

如果指定了此选项，所有 _全局_（非模块）文件都将被拼接合并到指定的单个输出文件中。

如果 `module` 为 `system` 或 `amd`，所有模块文件也会在所有全局内容之后拼接到该文件中。

注意：除非 `module` 设置为 `None`、`System` 或 `AMD`，否则不能使用 `outFile`。
此选项*不能*用于打包 CommonJS 或 ES6 模块。
