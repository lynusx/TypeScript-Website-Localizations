---
title: 使用声明文件
layout: docs
permalink: /zh/docs/handbook/declaration-files/consumption.html
oneline: '如何为你的项目下载 d.ts 文件'
---

## 下载

获取类型声明除了 npm 之外不需要任何其他工具。

例如，获取像 lodash 这样的库的声明文件只需要运行以下命令：

```cmd
npm install --save-dev @types/lodash
```

值得注意的是，如果 npm 包本身已经包含了声明文件（如[发布](/docs/handbook/declaration-files/publishing.html)中所述），则无需下载对应的 `@types` 包。

## 使用

接下来你就可以在 TypeScript 代码中毫无阻碍地使用 lodash 了。
这在模块化代码和全局代码中均可正常工作。

例如，一旦你通过 `npm install` 安装好了类型声明，就可以使用 import 语法编写：

```ts
import * as _ from 'lodash'
_.padStart('Hello TypeScript!', 20, ' ')
```

或者如果你没有使用模块，也可以直接使用全局变量 `_`。

```ts
_.padStart('Hello TypeScript!', 20, ' ')
```

## 查找

在大多数情况下，类型声明包的名称与 `npm` 上的包名完全相同，只是带有 `@types/` 前缀；
但如果你有需要，也可以使用 [Yarn 软件包搜索](https://yarnpkg.com/)来查找你所需库的声明包。

> 注意：如果要查找的声明文件不存在，你随时可以向社区贡献一份，为下一位寻找它的开发者提供帮助。
> 详情请参阅 DefinitelyTyped 的[贡献指南页面](https://definitelytyped.org/guides/contributing.html)。
