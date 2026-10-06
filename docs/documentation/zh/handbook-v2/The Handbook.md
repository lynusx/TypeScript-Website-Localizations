---
title: TypeScript 手册
layout: docs
permalink: /zh/docs/handbook/intro.html
oneline: 学习 TypeScript 的第一步
handbook: 'true'
---

## 关于本手册

自诞生并进入编程界二十多年以来，JavaScript 如今已成为有史以来应用最广泛的跨平台语言之一。从最初用于为网页添加简单交互的小型脚本语言起步，JavaScript 现已成长为各种规模的前端和后端应用程序的首选语言。然而，尽管用 JavaScript 编写的程序在规模、范围和复杂度上呈指数级增长，JavaScript 语言本身表达不同代码单元之间关系的能力却没有相应提升。再加上 JavaScript 相当独特的运行时语义，这种语言能力与程序复杂度之间的脱节，使得大规模的 JavaScript 开发变得难以管理。

程序员最常犯的错误可以归结为类型错误（type errors）：在期望某种类型值的地方使用了另一种类型的值。这可能是由于简单的拼写错误、未能理解库的 API 接口、对运行时行为的错误假设，或是其他原因导致的。TypeScript 的目标是成为 JavaScript 程序的静态类型检查器——换言之，它是一个在代码运行之前执行（静态），并确保程序类型正确（类型检查）的工具。

如果你没有 JavaScript 背景，并打算将 TypeScript 作为自己的第一门语言来学习，我们建议你先阅读 [Microsoft Learn JavaScript 教程](https://developer.microsoft.com/javascript/)或 [Mozilla Web Docs 上的 JavaScript 指南](https://developer.mozilla.org/docs/Web/JavaScript/Guide)。
如果你已有其他语言的开发经验，通过阅读本手册应该能够相当迅速地掌握 JavaScript 的语法。

## 本手册的结构

本手册分为两个部分：

- **手册（The Handbook）**

  TypeScript 手册旨在成为一份向日常开发者讲解 TypeScript 的综合性文档。你可以按照左侧导航栏自上而下阅读本手册。

  每一章或每一页都力求让你对所讲解的概念有深入透彻的理解。TypeScript 手册并非完整的语言规范，但它是一份涵盖该语言所有特性和行为的全面指南。

  阅读完本手册的读者应该能够：

  - 阅读并理解常用的 TypeScript 语法与模式
  - 解释重要编译器选项的作用
  - 在大多数情况下准确预测类型系统的行为

  出于清晰与精炼的考虑，手册的核心内容不会深入探讨所涉及特性的每个边界情况或细枝末节。你可以在参考文档中找到关于特定概念的更多细节。

- **参考文档（Reference Files）**

  导航栏中手册下方的参考文档部分旨在帮助你更深入地了解 TypeScript 某个特定部分的工作原理。你可以自上而下阅读，但每个部分都侧重深入解释单个概念——这意味着各章节之间并不强调连贯性。

### 非目标

本手册同时也力求简明扼要，让读者能够在几个小时内轻松读完。为了保持篇幅精炼，某些主题将不会涉及。

具体而言，本手册不会全面介绍函数、类和闭包等 JavaScript 核心基础知识。在适当的地方，我们会提供延伸阅读的链接，供你了解这些概念。

本手册也不旨在替代语言规范。在某些情况下，为了提供高层次、更易理解的解释，我们会略过边界情况或行为的形式化描述。作为补充，有专门的参考页面更加严谨、形式化地描述 TypeScript 行为的方方面面。参考页面并不面向不熟悉 TypeScript 的读者，因此它们可能会使用高级术语或提及你尚未了解的主题。

最后，除必要情况外，本手册不会涉及 TypeScript 如何与其他工具配合使用。诸如如何将 TypeScript 与 webpack、rollup、parcel、react、babel、closure、lerna、rush、bazel、preact、vue、angular、svelte、jquery、yarn 或 npm 一起配置等主题均不在讨论范围内——你可以在网络上的其他地方找到这些资源。

## 开始起步

在开始阅读[基础](/docs/handbook/2/basic-types.html)之前，我们建议你先阅读以下入门页面之一。这些入门指南旨在突出 TypeScript 与你所偏好的编程语言之间的核心异同，并澄清针对这些语言的常见误解。

- [针对编程新手的 TypeScript 指南](/docs/handbook/typescript-from-scratch.html)
- [针对 JavaScript 开发者的 TypeScript 指南](/docs/handbook/typescript-in-5-minutes.html)
- [针对 Java/C# 开发者的 TypeScript 指南](/docs/handbook/typescript-in-5-minutes-oop.html)
- [针对函数式程序员的 TypeScript 指南](/docs/handbook/typescript-in-5-minutes-func.html)

或者，直接跳转到[基础](/docs/handbook/2/basic-types.html)。
