---
title: 发布
layout: docs
permalink: /zh/docs/handbook/declaration-files/publishing.html
oneline: 如何将你的 d.ts 文件提供给用户
---

在按照本指南的步骤编写了声明文件之后，是时候将其发布到 npm 了。
将声明文件发布到 npm 主要有两种途径：

1. 与你的 npm 包打包在一起
2. 发布到 npm 上的 [@types 组织](https://www.npmjs.com/~types)。

如果你的类型是由源代码生成的，请将类型与源代码一起发布。TypeScript 和 JavaScript 项目都可以通过 [`declaration`](/tsconfig#declaration) 生成类型。

否则，我们建议将类型提交到 DefinitelyTyped，它会将其发布到 npm 上的 `@types` 组织中。

## 在 npm 包中包含声明文件

如果你的包有一个主 `.js` 文件，你还需要在 `package.json` 文件中指出主声明文件。
将 `types` 属性设置为指向打包后的声明文件。
例如：

```json
{
  "name": "awesome",
  "author": "Vandelay Industries",
  "version": "1.0.0",
  "main": "./lib/main.js",
  "types": "./lib/main.d.ts"
}
```

请注意，`"typings"` 字段与 `types` 是同义词，也可以使用。

## 依赖

所有依赖项均由 npm 管理。
请确保在 `package.json` 的 `"dependencies"` 部分中正确标记了你所依赖的所有声明包。
例如，假设我们编写了一个使用 Browserify 和 TypeScript 的包。

```json
{
  "name": "browserify-typescript-extension",
  "author": "Vandelay Industries",
  "version": "1.0.0",
  "main": "./lib/main.js",
  "types": "./lib/main.d.ts",
  "dependencies": {
    "browserify": "latest",
    "@types/browserify": "latest",
    "typescript": "next"
  }
}
```

在这里，我们的包依赖于 `browserify` 和 `typescript` 包。
`browserify` 没有将其声明文件与其 npm 包打包在一起，因此我们需要依赖 `@types/browserify` 来获取其声明。
另一方面，`typescript` 打包了自己的声明文件，因此不需要任何额外的依赖。

我们的包暴露了来自这两个包的声明，因此使用 `browserify-typescript-extension` 包的任何用户也需要拥有这些依赖项。
出于这个原因，我们使用的是 `"dependencies"` 而不是 `"devDependencies"`，否则我们的使用者就需要手动安装这些包。
如果我们只是编写一个命令行应用程序，并且不希望我们的包被用作库，那么我们可能会使用 `devDependencies`。

## 危险信号

### `/// <reference path="..." />`

*不要*在声明文件中使用 `/// <reference path="..." />`。

```ts
/// <reference path="../typescript/lib/typescriptServices.d.ts" />
....
```

*应该*改用 `/// <reference types="..." />`。

```ts
/// <reference types="typescript" />
....
```

请务必查阅[消费依赖](/docs/handbook/declaration-files/library-structures.html#consuming-dependencies)一节以了解更多信息。

### 打包所依赖的声明

如果你的类型定义依赖于另一个包：

- *不要*将它与你的声明合并，请分别保留在各自的文件中。
- 也*不要*将那些声明复制到你的包中。
- 如果该包没有打包其声明文件，*应该*依赖其 npm 类型声明包。

## 使用 `typesVersions` 进行版本选择

当 TypeScript 打开一个 `package.json` 文件以确定需要读取哪些文件时，它首先会查看一个名为 `typesVersions` 的字段。

#### 文件夹重定向（使用 `*`）

带有 `typesVersions` 字段的 `package.json` 可能如下所示：

```json
{
  "name": "package-name",
  "version": "1.0.0",
  "types": "./index.d.ts",
  "typesVersions": {
    ">=3.1": { "*": ["ts3.1/*"] }
  }
}
```

这个 `package.json` 告知 TypeScript 首先检查当前的 TypeScript 版本。
如果是 3.1 或更高版本，TypeScript 会计算出你所导入的相对于该包的路径，并从该包的 `ts3.1` 文件夹中读取。

这就是 `{ "*": ["ts3.1/*"] }` 的含义——如果你熟悉[路径映射](/tsconfig#paths)，它的工作机制与此完全一致。

在上面的示例中，如果我们在 TypeScript 3.1 环境下从 `"package-name"` 导入，TypeScript 将尝试从 `[...]/node_modules/package-name/ts3.1/index.d.ts`（以及其他相关路径）进行解析。
如果我们从 `package-name/foo` 导入，则会尝试查找 `[...]/node_modules/package-name/ts3.1/foo.d.ts` 和 `[...]/node_modules/package-name/ts3.1/foo/index.d.ts`。

如果在该示例中我们运行的不是 TypeScript 3.1 呢？
如果 `typesVersions` 中的字段均未匹配成功，TypeScript 会回退到 `types` 字段，因此这里的 TypeScript 3.0 及更早版本将被重定向到 `[...]/node_modules/package-name/index.d.ts`。

#### 文件重定向

当你只想单独更改单个文件的解析路径时，可以通过传入确切的文件名来告知 TypeScript 该如何以不同方式解析该文件：

```json
{
  "name": "package-name",
  "version": "1.0.0",
  "types": "./index.d.ts",
  "typesVersions": {
    "<4.0": { "index.d.ts": ["index.v3.d.ts"] }
  }
}
```

在 TypeScript 4.0 及以上版本中，导入 `"package-name"` 将解析为 `./index.d.ts`，而在 3.9 及以下版本中将解析为 `"./index.v3.d.ts"`。

需要注意的是，重定向仅影响包的*外部* API；项目内部的导入解析不受 `typesVersions` 的影响。例如，在前一个示例中，一个包含 `import * as foo from "./index"` 的 `d.ts` 文件仍然会映射到 `index.d.ts`，而不是 `index.v3.d.ts`；而另一个通过 `import * as foo from "package-name"` 导入的包则*会*得到 `index.v3.d.ts`。

## 匹配规则

TypeScript 决定编译器与语言版本是否匹配的方式是使用 Node 的 [semver 版本范围](https://github.com/npm/node-semver#ranges)。

## 多个字段

`typesVersions` 可以支持多个字段，其中每个字段名由要匹配的版本范围指定。

```json tsconfig
{
  "name": "package-name",
  "version": "1.0",
  "types": "./index.d.ts",
  "typesVersions": {
    ">=3.2": { "*": ["ts3.2/*"] },
    ">=3.1": { "*": ["ts3.1/*"] }
  }
}
```

由于版本范围可能存在重叠，因此决定应用哪个重定向取决于字段顺序。
这意味着在上面的示例中，尽管 `>=3.2` 和 `>=3.1` 匹配器都支持 TypeScript 3.2 及以上版本，但调换顺序可能会产生不同的行为，因此上述示例与以下示例并不等价：

```jsonc tsconfig
{
  "name": "package-name",
  "version": "1.0",
  "types": "./index.d.ts",
  "typesVersions": {
    // 注意：这种写法不起作用！
    ">=3.1": { "*": ["ts3.1/*"] },
    ">=3.2": { "*": ["ts3.2/*"] },
  },
}
```

## 发布到 [@types](https://www.npmjs.com/~types)

[@types](https://www.npmjs.com/~types) 组织下的包由 [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) 使用 [types-publisher 工具](https://github.com/microsoft/DefinitelyTyped-tools/tree/master/packages/publisher) 自动发布。
若要将你的声明作为 @types 包发布，请向 [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) 提交拉取请求（Pull Request）。
你可以在[贡献指南页面](https://definitelytyped.github.io/guides/contributing.html)找到更多详细信息。
