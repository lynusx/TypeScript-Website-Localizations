---
display: 'List Files'
oneline: '打印编译过程中读取的所有文件。'
---

打印属于编译一部分的文件名称。当你无法确定 TypeScript 是否包含了你预期的文件时，这非常有用。

例如：

```
example
├── index.ts
├── package.json
└── tsconfig.json
```

配置：

```json tsconfig
{
  "compilerOptions": {
    "listFiles": true
  }
}
```

将输出类似如下的路径：

```
$ npm run tsc
path/to/example/node_modules/typescript/lib/lib.d.ts
path/to/example/node_modules/typescript/lib/lib.es5.d.ts
path/to/example/node_modules/typescript/lib/lib.dom.d.ts
path/to/example/node_modules/typescript/lib/lib.webworker.importscripts.d.ts
path/to/example/node_modules/typescript/lib/lib.scripthost.d.ts
path/to/example/index.ts
```

注意：如果使用 TypeScript 4.2 及以上版本，建议优先使用 [`explainFiles`](#explainFiles)，它还会提供文件为何被添加的解释说明。
