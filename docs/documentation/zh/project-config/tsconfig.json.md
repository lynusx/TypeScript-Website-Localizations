---
title: 什么是 tsconfig.json
layout: docs
permalink: /zh/docs/handbook/tsconfig-json.html
oneline: 了解 TSConfig 的工作原理
translatable: true
---

## 概述

目录中如果存在 `tsconfig.json` 文件，则表明该目录是 TypeScript 项目的根目录。
`tsconfig.json` 文件指定了编译该项目所需的根文件以及编译器选项。

JavaScript 项目可以使用 `jsconfig.json` 文件来代替，两者的作用几乎相同，但 `jsconfig.json` 默认启用了部分与 JavaScript 相关的编译器标志。

项目可以通过以下方式之一进行编译：

## 使用 `tsconfig.json` 或 `jsconfig.json`

- 在不指定任何输入文件的情况下调用 tsc，此时编译器会从当前目录开始向上级父目录链搜索 `tsconfig.json` 文件。
- 在不指定输入文件的情况下调用 tsc，并使用 `--project`（或简写 `-p`）命令行选项，指定包含 `tsconfig.json` 文件的目录路径，或者包含配置的有效 `.json` 文件路径。

在命令行中指定了输入文件时，`tsconfig.json` 文件将被忽略。

## 示例

`tsconfig.json` 示例文件：

- 使用 [`files`](/tsconfig#files) 属性

  ```json tsconfig
  {
    "compilerOptions": {
      "module": "commonjs",
      "noImplicitAny": true,
      "removeComments": true,
      "preserveConstEnums": true,
      "sourceMap": true
    },
    "files": [
      "core.ts",
      "sys.ts",
      "types.ts",
      "scanner.ts",
      "parser.ts",
      "utilities.ts",
      "binder.ts",
      "checker.ts",
      "emitter.ts",
      "program.ts",
      "commandLineParser.ts",
      "tsc.ts",
      "diagnosticInformationMap.generated.ts"
    ]
  }
  ```

- 使用 [`include`](/tsconfig#include) 和 [`exclude`](/tsconfig#exclude) 属性

  ```json tsconfig
  {
    "compilerOptions": {
      "module": "system",
      "noImplicitAny": true,
      "removeComments": true,
      "preserveConstEnums": true,
      "outFile": "../../built/local/tsc.js",
      "sourceMap": true
    },
    "include": ["src/**/*"],
    "exclude": ["**/*.spec.ts"]
  }
  ```

## TSConfig 基础配置（Bases）

根据你打算运行代码的 JavaScript 运行时环境，你可以在 [github.com/tsconfig/bases](https://github.com/tsconfig/bases/) 找到可直接使用的基础配置。
这些是你的项目可以继承（extends）的 `tsconfig.json` 文件，它们通过处理运行时支持来简化你自身的 `tsconfig.json`。

例如，如果你正在编写一个使用 Node.js 12 及以上版本的项目，可以使用 npm 模块 [`@tsconfig/node12`](https://www.npmjs.com/package/@tsconfig/node12)：

```json tsconfig
{
  "extends": "@tsconfig/node12/tsconfig.json",

  "compilerOptions": {
    "preserveConstEnums": true
  },

  "include": ["src/**/*"],
  "exclude": ["**/*.spec.ts"]
}
```

这样可以让你的 `tsconfig.json` 专注于项目的个性化配置，而不是繁琐的运行时机制。目前已经有若干 TSConfig 基础配置，我们也期待社区能针对不同的环境贡献更多配置。

## 详细信息

`"compilerOptions"` 属性可以省略，此时将使用编译器的默认值。请参阅我们支持的完整[编译器选项（Compiler Options）](/tsconfig)列表。

## TSConfig 参考

深入了解数百个配置选项，请参阅 [TSConfig 参考](/tsconfig)。

## Schema

`tsconfig.json` 的 Schema 可以在 [JSON Schema Store](https://json.schemastore.org/tsconfig) 中找到。
