---
title: 从类型中创建类型
layout: docs
permalink: /zh/docs/handbook/2/types-from-types.html
oneline: '概述如何基于现有类型创建更多类型。'
---

TypeScript 的类型系统非常强大，因为它允许*基于其他类型*来表达类型。

这一思想最简单的体现形式是泛型。此外，我们还可以使用各种各样的*类型运算符*。
我们甚至还可以基于已有的*值*来表达类型。

通过组合使用各种类型运算符，我们能以简洁、可维护的方式来表达复杂的操作和值。
在本节中，我们将介绍如何基于现有的类型或值来表达新类型。

- [泛型](/docs/handbook/2/generics.html) - 接收参数的类型
- [keyof 类型运算符](/docs/handbook/2/keyof-types.html) - 使用 `keyof` 运算符创建新类型
- [typeof 类型运算符](/docs/handbook/2/typeof-types.html) - 使用 `typeof` 运算符创建新类型
- [索引访问类型](/docs/handbook/2/indexed-access-types.html) - 使用 `Type['a']` 语法访问类型的子集
- [条件类型](/docs/handbook/2/conditional-types.html) - 在类型系统中行为类似于 if 语句的类型
- [映射类型](/docs/handbook/2/mapped-types.html) - 通过映射现有类型中的每个属性来创建类型
- [模板字面量类型](/docs/handbook/2/template-literal-types.html) - 通过模板字面量字符串修改属性的映射类型
