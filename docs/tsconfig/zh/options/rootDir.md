---
display: 'Root Dir'
oneline: '指定源文件中的根目录。'
---

**默认值**：所有非声明输入文件的最长公共路径。如果设置了 [`composite`](#composite)，则默认值为包含 `tsconfig.json` 文件的目录。

当 TypeScript 编译文件时，它会在输出目录中保持与输入目录相同的目录结构。

例如，假设你有以下输入文件：

```
MyProj
├── tsconfig.json
├── core
│   ├── a.ts
│   ├── b.ts
│   ├── sub
│   │   ├── c.ts
├── types.d.ts
```

推导出的 `rootDir` 值为所有非声明输入文件的最长公共路径，在本例中为 `core/`。

如果你的 [`outDir`](#outDir) 为 `dist`，TypeScript 会输出如下目录树：

```
MyProj
├── dist
│   ├── a.js
│   ├── b.js
│   ├── sub
│   │   ├── c.js
```

然而，你可能希望让 `core` 成为输出目录结构的一部分。
通过在 `tsconfig.json` 中设置 `rootDir: "."`，TypeScript 将输出如下目录树：

```
MyProj
├── dist
│   ├── core
│   │   ├── a.js
│   │   ├── b.js
│   │   ├── sub
│   │   │   ├── c.js
```

需要注意的是，`rootDir` **不会影响哪些文件参与编译**。
它与 `tsconfig.json` 中的 [`include`](#include)、[`exclude`](#exclude) 或 [`files`](#files) 配置没有任何相互影响。

请注意，TypeScript 绝不会将输出文件写入 [`outDir`](#outDir) 之外的目录中，也绝不会跳过任何文件的发射（emit）。
出于这个原因，`rootDir` 还强制要求所有需要发射的文件都必须位于 `rootDir` 路径之下。

例如，假设你有如下目录树：

```
MyProj
├── tsconfig.json
├── core
│   ├── a.ts
│   ├── b.ts
├── helpers.ts
```

如果将 `rootDir` 指定为 `core`，_并且_ 将 [`include`](#include) 指定为 `*`，则会引发错误，因为这会导致一个文件（`helpers.ts`）需要被输出到 [`outDir`](#outDir) _之外_（即 `../helpers.js`）。
