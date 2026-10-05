---
display: 'Exclude Directories'
oneline: '从监视流程中排除一组目录。'
---

你可以使用 [`excludeFiles`](#excludeFiles) 大幅减少在 `--watch` 期间监视的文件数量。在 Linux 上，这是一种减少 TypeScript 跟踪的打开文件句柄数的有效方法。

```json tsconfig
{
  "watchOptions": {
    "excludeDirectories": ["**/node_modules", "_build", "temp/*"]
  }
}
```
