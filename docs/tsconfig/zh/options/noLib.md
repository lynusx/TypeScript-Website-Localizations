---
display: 'No Lib'
oneline: '禁用包含任何库文件，包括默认的 lib.d.ts。'
---

禁用自动包含任何库文件。
如果设置了此选项，`lib` 将被忽略。

如果没有一组针对关键原生类型的接口定义（如：`Array`、`Boolean`、`Function`、`IArguments`、`Number`、`Object`、`RegExp` 和 `String`），TypeScript 就*无法*编译任何内容。如果使用了 `noLib`，你需要自行包含这些类型的定义。
