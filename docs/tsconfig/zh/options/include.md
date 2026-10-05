---
display: 'Include'
oneline: '指定匹配要包含在编译中的文件的 glob 模式列表。'
---

指定一个文件名或 glob 模式数组，包含在程序中。
这些文件名相对于包含 `tsconfig.json` 文件的目录进行解析。

```json
{
  "include": ["src/**/*", "tests/**/*"]
}
```

这将包含：

<!-- TODO: #135
```diff
  .
- ├── scripts
- │   ├── lint.ts
- │   ├── update_deps.ts
- │   └── utils.ts
+ ├── src
+ │   ├── client
+ │   │    ├── index.ts
+ │   │    └── utils.ts
+ │   ├── server
+ │   │    └── index.ts
+ ├── tests
+ │   ├── app.test.ts
+ │   ├── utils.ts
+ │   └── tests.d.ts
- ├── package.json
- ├── tsconfig.json
- └── yarn.lock
``` -->

```
.
├── scripts                ⨯
│   ├── lint.ts            ⨯
│   ├── update_deps.ts     ⨯
│   └── utils.ts           ⨯
├── src                    ✓
│   ├── client             ✓
│   │    ├── index.ts      ✓
│   │    └── utils.ts      ✓
│   ├── server             ✓
│   │    └── index.ts      ✓
├── tests                  ✓
│   ├── app.test.ts        ✓
│   ├── utils.ts           ✓
│   └── tests.d.ts         ✓
├── package.json
├── tsconfig.json
└── yarn.lock
```

`include` 和 `exclude` 支持通配符来构建 glob 模式：

- `*` 匹配零个或多个字符（不包括目录分隔符）
- `?` 匹配任意单个字符（不包括目录分隔符）
- `**/` 匹配任意层级的嵌套目录

如果模式的最后一个路径段不包含文件扩展名或通配符，则会被视为目录，该目录下具有支持扩展名的文件都会被包含在内（默认包括 `.ts`、`.tsx` 和 `.d.ts`；如果 [`allowJs`](#allowJs) 设置为 true，还会包含 `.js` 和 `.jsx`）。
