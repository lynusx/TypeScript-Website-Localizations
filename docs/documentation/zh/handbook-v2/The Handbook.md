---
title: TypeScript 手册
layout: docs
permalink: /zh/docs/handbook/intro.html
oneline: 学习 TypeScript 的第一步
handbook: 'true'
---

## 关于本手册

在引入编程社区 20 多年后，JavaScript 如今已成为有史以来应用最广泛的跨平台语言之一。JavaScript 最初只是一种为网页添加简单交互的小型脚本语言，如今已发展成为各种规模的前后端应用的首选语言。尽管用 JavaScript 编写的程序的规模、范围和复杂性呈指数级增长，但 JavaScript 语言表达不同代码单元之间关系的能力却没有随之提升。再加上 JavaScript 相当奇特的运行时语义，这种语言能力与程序复杂性之间的脱节，使得大规模的 JavaScript 开发变得难以管理。

程序员最常犯的错误可以说就是类型错误：即在预期某种类型值的地方使用了另一种类型的值。这可能是因为简单的拼写错误、未能理解库的 API 表面、对运行时行为的错误假设，或是其他错误。TypeScript 的目标是成为 JavaScript 程序的静态类型检查器——换言之，它是一个在代码运行前执行的工具（静态），用于确保程序的类型正确无误（类型检查）。

如果你没有 JavaScript 背景，且打算将 TypeScript 作为你的第一门编程语言，我们建议你先阅读 [Microsoft Learn JavaScript 教程](https://developer.microsoft.com/zh-cn/javascript/)，或者阅读 Mozilla Web Docs 上的 [JavaScript 指南](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide)。如果你有其他语言的经验，通过阅读本手册，你应该能很快掌握 JavaScript 的语法。

## 本手册的结构

本手册分为两部分：

- **手册**

  TypeScript 手册旨在成为一份全面向日常程序员解释 TypeScript 的文档。你可以按照左侧导航栏自上而下阅读。

  每一章或每一页都能让你对特定概念建立深入的理解。TypeScript 手册并非完整的语言规范，而是旨在成为涵盖该语言所有特性和行为的综合指南。

  读完本指南后，读者应该能够：
  - 阅读并理解常用的 TypeScript 语法和模式
  - 解释重要编译器选项的作用
  - 在大多数情况下准确预测类型系统的行为

  为了保持清晰和简洁，手册的主体内容不会探讨所涉特性的每一个边缘情况或细节。你可以在参考文章中找到关于特定概念的更多细节。

- **参考文件**

  导航栏中位于手册下方的参考部分，旨在让你更深入地了解 TypeScript 特定部分的工作原理。你可以从上往下读，但每一节都旨在深入解释单一概念——这意味着它们之间不强求连贯性。

### 非目标

本手册也是一份简明的文档，让你可以在几个小时内轻松读完。为了保持篇幅简短，手册不会涵盖某些主题。

具体而言，手册不会全面介绍函数、类和闭包等核心 JavaScript 基础知识。在适当的地方，我们会提供背景阅读的链接，方便你了解这些概念。

此外，本手册也无意取代语言规范。在某些情况下，我们会跳过边缘情况或行为的正式描述，转而提供高层次、更易懂的解释。相比之下，单独的参考页更精确、正式地描述了 TypeScript 行为的诸多方面。参考页不是为不熟悉 TypeScript 的读者准备的，因此可能会使用高级术语，或者引用你尚未阅读过的主题。

最后，除非必要，本手册不会介绍 TypeScript 如何与其他工具交互。例如如何将 TypeScript 与 webpack、rollup、parcel、react、babel、closure、lerna、rush、bazel、preact、vue、angular、svelte、jquery、yarn 或 npm 配置等主题，均不在讨论范围内——你可以在网络上的其他地方找到这些资源。

## 快速起步

在学习[基础知识](/docs/handbook/2/basic-types.html)之前，我们建议阅读以下入门页面之一。这些入门页面旨在强调 TypeScript 与你擅长的编程语言之间的主要异同，并澄清针对这些语言的常见误解。

- [面向编程初学者的 TypeScript](/docs/handbook/typescript-from-scratch.html)
- [为 JavaScript 程序员准备的 TypeScript](/docs/handbook/typescript-in-5-minutes.html)
- [面向 Java/C# 程序员的 TypeScript](/docs/handbook/typescript-in-5-minutes-oop.html)
- [面向函数式程序员的 TypeScript](/docs/handbook/typescript-in-5-minutes-func.html)

否则，请直接跳转到[基础知识](/docs/handbook/2/basic-types.html)。
