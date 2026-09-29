---
title: 联合类型赋值
layout: docs
permalink: /zh/docs/handbook/2/types-from-types.html
oneline: '概述如何从现有类型创建更多类型。.'
---

TypeScript 的类型系统非常强大，因为它允许 _基于其他类型_ 来表达类型。

这个概念最简单的形式就是泛型。此外，还有各种各样的 _类型操作符_ 可供使用。甚至可以基于已有的 _值_ 来表达类型。

结合各种类型操作符，能以简洁、可维护的方式表达复杂的操作和值。本节将介绍如何基于现有的类型或值来表达新类型。

- [泛型](/docs/handbook/2/generics.html) —— 接受参数的类型
- [Keyof 类型操作符](/docs/handbook/2/keyof-types.html) —— 使用 `keyof` 操作符创建新类型
- [Typeof 类型操作符](/docs/handbook/2/typeof-types.html) —— 使用 `typeof` 操作符创建新类型
- [索引访问类型](/docs/handbook/2/indexed-access-types.html) —— 使用 `Type['a']` 语法访问类型的子集
- [条件类型](/docs/handbook/2/conditional-types.html) —— 在类型系统中行为类似 if 语句的类型
- [映射类型](/docs/handbook/2/mapped-types.html) —— 通过映射现有类型中的每个属性来创建类型
- [模板字面量类型](/docs/handbook/2/template-literal-types.html) —— 通过模板字面量字符串更改属性的映射类型
