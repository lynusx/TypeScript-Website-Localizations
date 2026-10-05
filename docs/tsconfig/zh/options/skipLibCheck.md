---
display: 'Skip Lib Check'
oneline: '跳过对所有 .d.ts 声明文件的类型检查。'
---

跳过对声明文件的类型检查。

这可以在牺牲类型系统精准度的前提下节省编译时间。例如，两个库可能会以不一致的方式定义同一个 `type` 的两份副本。TypeScript 不会对所有 `d.ts` 文件进行全量检查，而是仅对你在应用程序源代码中明确引用的代码进行类型检查。

考虑使用 `skipLibCheck` 的一个常见场景是 `node_modules` 中存在同一个库的类型定义的两个副本。在这些情况下，你应该考虑使用如 [yarn's resolutions](https://yarnpkg.com/lang/en/docs/selective-version-resolutions/) 之类的特性来确保依赖树中只有该依赖项的一个副本，或者通过深入理解依赖解析机制来确保仅存在单一副本，从而无需额外工具即可解决该问题。

另一种场景是，当你正在进行 TypeScript 版本升级迁移，而变更导致了 node_modules 以及 JS 标准库中产生报错，且你不想在升级 TypeScript 期间处理这些问题。

请注意，如果这些问题来自 TypeScript 标准库，你可以利用 [TypeScript 4.5 的 lib 替换](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-5.html#supporting-lib-from-node_modules)技术来替换该库。
