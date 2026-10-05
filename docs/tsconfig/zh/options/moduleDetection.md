---
display: 'Module Detection'
oneline: '指定用于检测文件是脚本还是模块的方法。'
---

该设置控制 TypeScript 如何判断一个文件是[脚本还是模块](/docs/handbook/modules/theory.html#scripts-and-modules-in-javascript)。

共有三种可选配置：

- `"auto"`（默认值）- TypeScript 不仅会查找 import 和 export 语句，在 [`module`](#module) 为 `nodenext` 或 `node16` 时还会检查 `package.json` 中的 `"type"` 字段是否设置为 `"module"`，并在 [`jsx`](#jsx) 为 `react-jsx` 时检查当前文件是否为 JSX 文件。

- `"legacy"` - 与 4.6 及更早版本的行为相同，使用 import 和 export 语句来判断文件是否为模块。

- `"force"` - 确保每个非声明文件都被视为模块。
