---
display: 'Lib Replacement'
oneline: '允许使用自定义库文件替换默认的 `lib` 文件。'
---

TypeScript 4.5 引入了使用自定义文件替换默认 `lib` 文件的功能。
所有内置库文件都会首先尝试从名为 `@typescript/lib-*` 的包中解析。
例如，你可以通过如下 `package.json` 将 `dom` 库锁定在 [`@types/web` 包](https://www.npmjs.com/package/@types/web?activeTab=readme) 的特定版本：

```json
{
  "devDependencies": {
    "@typescript/lib-dom": "npm:@types/web@0.0.199"
  }
}
```

安装后，应存在名为 `@typescript/lib-dom` 的包，TypeScript 在查找 `lib.dom.d.ts` 时将始终从该包中寻找。

`--libReplacement` 标志允许你禁用此行为。
如果你没有使用任何 `@typescript/lib-*` 包，现在可以通过 `--libReplacement false` 禁用这些包的查找。
将来 `--libReplacement false` 可能会成为默认行为，因此如果你当前依赖该特性，应考虑使用 `--libReplacement true` 显式启用它。
