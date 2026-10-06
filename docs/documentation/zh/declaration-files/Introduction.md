---
title: 简介
layout: docs
permalink: /zh/docs/handbook/declaration-files/introduction.html
oneline: '如何编写高质量的 TypeScript 声明（d.ts）文件'
---

“声明文件”章节旨在指导你如何编写高质量的 TypeScript 声明文件。在开始之前，我们需要假定你已经掌握了 TypeScript 语言的基础知识。

如果你还没有阅读过 [TypeScript 手册](/docs/handbook/2/basic-types.html)，
建议先阅读该手册以熟悉基础概念，尤其是类型和模块。

学习 .d.ts 文件工作原理的最常见场景，是为一个没有类型的 npm 包补充类型。
在这种情况下，你可以直接跳转到[模块 .d.ts](/docs/handbook/declaration-files/templates/module-d-ts.html)。

“声明文件”章节分为以下几个部分。

## [规范与示例](/docs/handbook/declaration-files/by-example.html)

当我们编写声明文件时，往往只有基础库的示例代码可供参考。
[规范与示例](/docs/handbook/declaration-files/by-example.html)章节展示了许多常见的 API 模式，以及如何为它们分别编写声明。
本指南面向可能尚未完全熟悉 TypeScript 每种语言特性的初学者。

## [库结构](/docs/handbook/declaration-files/library-structures.html)

[库结构](/docs/handbook/declaration-files/library-structures.html)指南帮助你了解常见的库格式，以及如何为每种格式编写合适的声明文件。
如果你正在编辑现有文件，可能不需要阅读本节。
强烈建议新声明文件的作者阅读本节，以便深入了解库的格式如何影响声明文件的编写方式。

在“模板”小节中，你会找到许多声明文件模板，它们是编写新文件时的良好起点。
如果你已经清楚你的库结构，请查看侧边栏中的“d.ts 模板”部分。

## [注意事项](/docs/handbook/declaration-files/do-s-and-don-ts.html)

声明文件中的许多常见错误都可以轻松避免。
[注意事项](/docs/handbook/declaration-files/do-s-and-don-ts.html)章节指出了常见错误，
说明了如何检测它们，
以及如何修复它们。
每个人都应该阅读本节，以帮助自己避免常见错误。

## [深入解析](/docs/handbook/declaration-files/deep-dive.html)

对于想要深入了解声明文件底层运行机制的有经验作者，
[深入解析](/docs/handbook/declaration-files/deep-dive.html)章节解释了声明文件编写中的许多高级概念，
并展示了如何利用这些概念创建更整洁、更直观的声明文件。

## [发布到 npm](/docs/handbook/declaration-files/publishing.html)

[发布](/docs/handbook/declaration-files/publishing.html)章节解释了如何将你的声明文件发布到 npm 包中，并介绍了如何管理你的依赖包。

## [查找并安装声明文件](/docs/handbook/declaration-files/consumption.html)

对于 JavaScript 库的使用者，[使用声明文件](/docs/handbook/declaration-files/consumption.html)章节提供了几个简单的步骤来查找并安装对应的声明文件。
