---
display: 'Declaration'
oneline: '为项目中的 TypeScript 和 JavaScript 文件生成 .d.ts 文件。'
---

为项目中的每个 TypeScript 或 JavaScript 文件生成 `.d.ts` 文件。
这些 `.d.ts` 文件是描述模块外部 API 的类型定义文件。
有了 `.d.ts` 文件，像 TypeScript 这样的工具便能为未提供类型标注的代码提供智能提示（IntelliSense）和精确的类型。

当 `declaration` 设置为 `true` 时，针对如下 TypeScript 代码运行编译器：

```ts twoslash
export let helloWorld = 'hi'
```

将生成类似这样的 `index.js` 文件：

```ts twoslash
// @showEmit
export let helloWorld = 'hi'
```

以及对应的 `helloWorld.d.ts`：

```ts twoslash
// @showEmittedFile: index.d.ts
// @showEmit
// @declaration
export let helloWorld = 'hi'
```

为 JavaScript 文件生成 `.d.ts` 文件时，你可能需要使用 [`emitDeclarationOnly`](#emitDeclarationOnly) 或使用 [`outDir`](#outDir)，以确保 JavaScript 文件不会被覆盖。
