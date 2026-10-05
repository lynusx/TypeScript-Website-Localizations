---
display: 'Module Resolution'
oneline: '指定 TypeScript 如何根据给定的模块说明符查找文件。'
---

指定模块解析策略：

- `'node16'` 或 `'nodenext'`：用于现代版本的 Node.js。Node.js v12 及更高版本同时支持 ECMAScript import 和 CommonJS `require`，二者使用不同的算法进行解析。当这些 `moduleResolution` 值与对应的 [`module`](#module) 值结合使用时，会根据 Node.js 在输出的 JavaScript 代码中看到的是 `import` 还是 `require`，为每次解析选择正确的算法。
- `'node10'`（此前称为 `'node'`）：用于早于 v10 的 Node.js 版本，这些版本仅支持 CommonJS `require`。在现代代码中你大概率不需要使用 `node10`。
- `'bundler'`：用于配合打包工具使用。与 `node16` 和 `nodenext` 类似，该模式支持 package.json 的 `"imports"` 和 `"exports"` 字段；但与 Node.js 解析模式不同的是，`bundler` 从不要求导入中的相对路径包含文件扩展名。
- `'classic'`：在 TypeScript 1.6 发布之前使用。不应再使用 `classic`。

有关更多信息，请参阅解释 [TypeScript 模块解析背后的理论](https://www.typescriptlang.org/docs/handbook/modules/theory.html#module-resolution)以及[各个选项细节](/docs/handbook/modules/reference.html#the-moduleresolution-compiler-option)的参考页面。
