---
title: 每日构建
layout: docs
permalink: /zh/docs/handbook/nightly-builds.html
oneline: 如何使用 TypeScript 的每日构建版本
translatable: true
---

在太平洋标准时间（PST）的每日午夜，[TypeScript 的 `main` 分支](https://github.com/Microsoft/TypeScript/tree/main)都会自动构建并发布到 npm。
以下将介绍如何获取该版本并在你的开发工具中使用它。

## 使用 npm

```shell
npm install -D typescript@next
```

## 更新你的 IDE 以使用每日构建

你也可以更新编辑器或 IDE 来使用每日构建版本。
通常需要先通过 npm 安装该包。
本节后续内容均默认你已安装了 `typescript@next`。

### Visual Studio Code

VS Code 官方网站提供了[关于如何选择工作区 TypeScript 版本的文档说明](https://code.visualstudio.com/Docs/languages/typescript#_using-newer-typescript-versions)。
在工作区安装了每日构建版的 TypeScript 后，你可以按照该文档的指引进行操作，也可以直接在 JSON 视图中更新工作区设置。
一种直接的方式是打开或创建工作区的 `.vscode/settings.json` 文件，并添加如下属性：

```json
"typescript.tsdk": "<path to your folder>/node_modules/typescript/lib"
```

另外，如果你只是想在 Visual Studio Code 中体验 JavaScript 和 TypeScript 的每日构建编辑支持，而无需更改工作区版本，也可以直接安装 [JavaScript and TypeScript Nightly 扩展](https://marketplace.visualstudio.com/items?itemName=TypeScriptTeam.vscode-typescript-nightly)。

### Sublime Text

在 `Settings - User` 配置文件中添加如下内容：

```json
"typescript_tsdk": "<path to your folder>/node_modules/typescript/lib"
```

更多信息请参阅 [Sublime Text 的 TypeScript 插件安装文档](https://github.com/Microsoft/TypeScript-Sublime-Plugin#installation)。

### Visual Studio 2013 与 2015

> 注意：绝大多数变更无需重新安装新版 VS TypeScript 插件。

目前每日构建版本不包含完整的插件安装程序，但我们正在努力实现每日构建安装程序的同步发布。

1. 下载 [VSDevMode.ps1](https://github.com/Microsoft/TypeScript/blob/main/scripts/VSDevMode.ps1) 脚本。

   > 另请参考关于[使用自定义语言服务文件](https://github.com/Microsoft/TypeScript/wiki/Dev-Mode-in-Visual-Studio#using-a-custom-language-service-file)的 Wiki 页面。

2. 在 PowerShell 命令行窗口中运行：

针对 VS 2015：

```posh
VSDevMode.ps1 14 -tsScript <path to your folder>/node_modules/typescript/lib
```

针对 VS 2013：

```posh
VSDevMode.ps1 12 -tsScript <path to your folder>/node_modules/typescript/lib
```

### IntelliJ IDEA (Mac)

打开 `Preferences` > `Languages & Frameworks` > `TypeScript`：

> TypeScript Version：若通过 npm 全局安装，则设置为：`/usr/local/lib/node_modules/typescript/lib`

### IntelliJ IDEA (Windows)

打开 `File` > `Settings` > `Languages & Frameworks` > `TypeScript`：

> TypeScript Version：若通过 npm 全局安装，则设置为：`C:\Users\USERNAME\AppData\Roaming\npm\node_modules\typescript\lib`
