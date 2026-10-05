---
display: 'Declaration Dir'
oneline: '指定所生成声明文件的输出目录。'
---

提供了一种配置声明文件生成根目录的方法。

```
example
├── index.ts
├── package.json
└── tsconfig.json
```

配合如下 `tsconfig.json`：

```json tsconfig
{
  "compilerOptions": {
    "declaration": true,
    "declarationDir": "./types"
  }
}
```

会将 `index.ts` 的 d.ts 文件放置在 `types` 目录中：

```
example
├── index.js
├── index.ts
├── package.json
├── tsconfig.json
└── types
    └── index.d.ts
```
