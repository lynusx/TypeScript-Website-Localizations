---
display: 'Disable Source Project Reference Redirect'
oneline: '在引用组合项目时，禁用优先使用源文件而非声明文件的行为。'
---

在处理 [TypeScript 组合项目](/docs/handbook/project-references.html)时，该选项提供了一种[回退到 3.7 之前版本](/docs/handbook/release-notes/typescript-3-7.html#build-free-editing-with-project-references)行为的方式，即使用 d.ts 文件作为模块间的边界。
而在 3.7 版本中，单一事实来源已转变为你的 TypeScript 文件。
