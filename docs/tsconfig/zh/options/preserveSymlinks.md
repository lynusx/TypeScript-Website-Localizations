---
display: 'Preserve Symlinks'
oneline: '禁止将符号链接解析为其真实路径。这与 Node.js 中的同名标志相对应。'
---

该选项对应于 Node.js 中的同名标志；它不会将符号链接解析为其真实路径。

该标志的行为与 Webpack 的 `resolve.symlinks` 选项正好相反（即：将 TypeScript 的 `preserveSymlinks` 设置为 true 相当于将 Webpack 的 `resolve.symlinks` 设置为 false，反之亦然）。

启用此选项后，对模块和包的引用（例如 `import` 语句和 `/// <reference type="..." />` 指令）都将相对于符号链接文件所在的位置进行解析，而不是相对于符号链接所指向的实际路径解析。
