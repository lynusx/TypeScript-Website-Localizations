---
display: 'Keyof Strings Only'
oneline: '让 keyof 仅返回 string，而非 string、number 或 symbol。已废弃的历史选项。'
---

当对带有字符串索引签名的类型应用 `keyof` 类型操作符时，该标志会将其返回值从 `string | number` 改为仅返回 `string`。

该标志用于帮助开发者保持在 [TypeScript 2.9 发布之前](/docs/handbook/release-notes/typescript-2-9.html#support-number-and-symbol-named-properties-with-keyof-and-mapped-types)的历史行为。
