---
display: 'Files'
oneline: '指定包含的文件列表。与 [`include`](#include) 不同，此选项不支持 glob 通配符模式。'
---

指定要包含在程序中的文件白名单。如果其中任何文件未找到，将会报错。

```json tsconfig
{
  "compilerOptions": {},
  "files": [
    "core.ts",
    "sys.ts",
    "types.ts",
    "scanner.ts",
    "parser.ts",
    "utilities.ts",
    "binder.ts",
    "checker.ts",
    "tsc.ts"
  ]
}
```

当你只有少量文件且不需要使用 glob 通配符来引用大量文件时，这非常有用。如果需要使用通配符匹配文件，请使用 [`include`](#include)。
