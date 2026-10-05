---
display: 'No Implicit Use Strict'
oneline: "禁止在生成的 JavaScript 文件中添加 'use strict' 指令。"
---

你通常不需要使用此选项。默认情况下，当将模块文件输出到非 ES6 目标时，TypeScript 会在文件顶部生成 `"use strict";` 序言指令。
该设置将禁用此序言指令。
