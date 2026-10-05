---
display: 'Plugins'
oneline: '指定要引入的语言服务插件列表。'
---

在编辑器内部运行的语言服务插件列表。

语言服务插件是一种基于现有 TypeScript 文件向用户提供额外信息的方式。它们可以增强 TypeScript 与编辑器之间现有的消息传递，或者提供自定义的错误提示信息。

例如：

- [ts-sql-plugin](https://github.com/xialvjun/ts-sql-plugin#readme) &mdash; 为模板字符串 SQL 构建器添加 SQL 检查（linting）。
- [typescript-styled-plugin](https://github.com/Microsoft/typescript-styled-plugin) &mdash; 在模板字符串内部提供 CSS 检查。
- [typescript-eslint-language-service](https://github.com/Quramy/typescript-eslint-language-service) &mdash; 在编译器输出中提供 ESLint 错误消息与自动修复。
- [ts-graphql-plugin](https://github.com/Quramy/ts-graphql-plugin) &mdash; 在 GraphQL 查询模板字符串内部提供校验与自动补全。

VS Code 支持扩展[自动引入语言服务插件](https://code.visualstudio.com/api/references/contribution-points#contributes.typescriptServerPlugins)，因此你的编辑器中可能已经运行了某些插件，而无需在 `tsconfig.json` 中显式定义它们。
