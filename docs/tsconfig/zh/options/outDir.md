---
display: 'Out Dir'
oneline: '为所有生成的文件指定输出目录。'
---

如果指定，生成的 `.js`（以及 `.d.ts`、`.js.map` 等）文件将被输出到该目录中。
原始源文件的目录结构将被保留；如果计算出的根目录不符合你的预期，请参阅 [`rootDir`](#rootDir)。

如果未指定，`.js` 文件将输出在生成它们的 `.ts` 文件的同一目录下：

```sh
$ tsc

example
├── index.js
└── index.ts
```

使用如下 `tsconfig.json`：

```json tsconfig
{
  "compilerOptions": {
    "outDir": "dist"
  }
}
```

使用这些设置运行 `tsc` 会将输出文件生成到指定的 `dist` 目录中：

```sh
$ tsc

example
├── dist
│   └── index.js
├── index.ts
└── tsconfig.json
```
