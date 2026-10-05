---
display: 'Emit BOM'
oneline: '在输出文件的开头写入 UTF-8 字节顺序标记（BOM）。'
---

控制 TypeScript 在写入输出文件时是否输出[字节顺序标记（BOM）](https://wikipedia.org/wiki/Byte_order_mark)。
某些运行时环境需要 BOM 才能正确解析 JavaScript 文件；其他环境则要求不能包含 BOM。
除非你有充分的理由需要修改，否则保持默认值 `false` 通常是最佳选择。
