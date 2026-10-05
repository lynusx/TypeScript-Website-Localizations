---
display: 'Exclude Files'
oneline: "在监视模式的处理中排除指定的文件列表。"
---

你可以使用 `excludeFiles` 从被监视的文件中排除一组指定的文件。

```json tsconfig
{
  "watchOptions": {
    "excludeFiles": ["temp/file.ts"]
  }
}
```
