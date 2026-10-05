---
display: 'Explain Files'
oneline: '打印在编译期间读取的文件以及将其包含在内的原因。'
---

打印 TypeScript 视作项目一部分的文件名称，以及它们被包含在编译中的原因。

例如，对于仅包含单个 `index.ts` 文件的项目：

```sh
example
├── index.ts
├── package.json
└── tsconfig.json
```

在 `tsconfig.json` 中将 `explainFiles` 设置为 true：

```json
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "explainFiles": true
  }
}
```

在该文件夹中运行 TypeScript，输出将如下所示：

```
❯ tsc
node_modules/typescript/lib/lib.d.ts
  Default library for target 'es5'
node_modules/typescript/lib/lib.es5.d.ts
  Library referenced via 'es5' from file 'node_modules/typescript/lib/lib.d.ts'
node_modules/typescript/lib/lib.dom.d.ts
  Library referenced via 'dom' from file 'node_modules/typescript/lib/lib.d.ts'
node_modules/typescript/lib/lib.webworker.importscripts.d.ts
  Library referenced via 'webworker.importscripts' from
    file 'node_modules/typescript/lib/lib.d.ts'
node_modules/typescript/lib/lib.scripthost.d.ts
  Library referenced via 'scripthost'
    from file 'node_modules/typescript/lib/lib.d.ts'
index.ts
  Matched by include pattern '**/*' in 'tsconfig.json'
```

上述输出展示了：

- 基于 [`target`](#target) 进行的初始 lib.d.ts 查找，以及被引用的 `.d.ts` 文件链
- 通过 [`include`](#include) 的默认模式定位到的 `index.ts` 文件

该选项用于调试文件为何会被包含在编译中。
