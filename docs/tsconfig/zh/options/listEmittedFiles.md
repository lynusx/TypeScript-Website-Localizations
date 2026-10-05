---
display: 'List Emitted Files'
oneline: '在编译后输出生成的文件名称。'
---

将作为编译结果生成的文件名称打印到终端。

此标志在以下两种情况下非常有用：

- 你希望在终端构建链中转译 TypeScript，并将生成的文件名传递给下一个命令进行处理。
- 你不确定 TypeScript 是否包含了你预期的文件，需要借此调试[文件包含设置](#Project_Files_0)。

例如：

```
example
├── index.ts
├── package.json
└── tsconfig.json
```

配合：

```json tsconfig
{
  "compilerOptions": {
    "declaration": true,
    "listEmittedFiles": true
  }
}
```

将输出类似如下的路径：

```
$ npm run tsc

path/to/example/index.js
path/to/example/index.d.ts
```

通常情况下，TypeScript 在成功编译时会静默退出。
