---
title: 针对编程新手的 TypeScript 指南
short: 针对编程新手的 TS 指南
layout: docs
permalink: /zh/docs/handbook/typescript-from-scratch.html
oneline: 从零开始学习 TypeScript
---

恭喜你选择 TypeScript 作为你的首门编程语言之一——这是一个非常明智的决定！

你可能已经听说过 TypeScript 是 JavaScript 的某种“风味（flavor）”或“变体（variant）”。
在现代编程语言中，TypeScript（TS）与 JavaScript（JS）之间的关系相当独特，深入了解这种关系将有助于你理解 TypeScript 是如何在 JavaScript 的基础上进行扩展的。

## 什么是 JavaScript？简要历史

JavaScript（也称为 ECMAScript）最初只是作为浏览器的简单脚本语言而诞生的。
在它被发明之初，人们预期它只用于网页中嵌入的简短代码片段——编写超过几十行代码在当时都是相当少见的。
正因如此，早期的 Web 浏览器执行这类代码的速度非常慢。
然而随着时间的推移，JS 变得越来越流行，Web 开发者开始使用它来创建丰富的交互体验。

面对不断增长的 JS 使用需求，Web 浏览器开发者通过优化执行引擎（动态编译）并扩展其能力（增加 API）来做出响应，这反过来又促使 Web 开发者更加广泛地使用它。
在现代网站中，你的浏览器经常需要运行数十万行代码规模的应用程序。
这就是“Web”漫长而渐进的成长历程：从最初简单的静态页面网络，演变成承载各类丰富**应用程序**的综合平台。

不仅如此，JS 的普及程度使其早已跳出浏览器的范畴，例如使用 Node.js 构建 JS 服务端。
JS“随处运行”的特性使其成为跨平台开发的极具吸引力的选择。
如今，很多开发者**仅**使用 JavaScript 就能完成整个全栈应用的开发！

总而言之，这门语言最初仅为简易场景而设计，随后发展成为可编写数百万行代码应用程序的成熟工具。
每种语言都有其自身的**怪癖（quirks）**——那些出人意料的奇特行为，而 JavaScript 简陋的开端使其拥有**诸多**此类怪癖。例如：

- JavaScript 的相等运算符（`==`）会对其操作数进行**类型强制转换**，从而导致出乎意料的行为：

  ```js
  if ('' == 0) {
    // It is! But why??
  }
  if (1 < x < 3) {
    // True for *any* value of x!
  }
  ```

- JavaScript 还允许访问不存在的属性：

  ```js
  const obj = { width: 10, height: 15 }
  // Why is this NaN? Spelling is hard!
  const area = obj.width * obj.heigth
  ```

大多数编程语言在出现这类错误时都会抛出异常，有些甚至会在编译期间——即在代码运行之前——就报告错误。
在编写小型程序时，这些怪癖虽然令人烦恼但尚可应付；而当编写成千上万行代码的应用程序时，这些层出不穷的意外就成了一个严重的问题。

## TypeScript：静态类型检查器

我们在前文中提到，某些语言根本不允许这些存在缺陷的程序运行。
在不运行代码的情况下检测其中的错误被称为**静态检查**。
根据被操作的值的种类来判定什么是错误、什么不是错误，则被称为静态**类型**检查。

TypeScript 在执行前检查程序中的错误，并且基于**值的种类**来进行检查，因此它是一个**静态类型检查器**。
例如，上面的最后一个示例之所以出错，就是因为 `obj` 的**类型**。
以下是 TypeScript 发现的错误：

```ts twoslash
// @errors: 2551
const obj = { width: 10, height: 15 }
const area = obj.width * obj.heigth
```

### JavaScript 的类型化超集

那么，TypeScript 与 JavaScript 究竟有着怎样的关系呢？

#### 语法

TypeScript 是一门作为 JavaScript **超集**的语言：因此所有合法的 JS 语法在 TS 中都是合法的。
语法指的是我们编写文本以构成程序的方式。
例如，以下代码存在**语法**错误，因为它缺少了一个 `)`：

```ts twoslash
// @errors: 1005
let a = (4
```

TypeScript 不会仅仅因为语法本身就把任何合法的 JavaScript 代码视为错误。
这意味着你可以把任何能够正常运行的 JavaScript 代码直接放入 TypeScript 文件中，而不必担心它的书写格式。

#### 类型

然而，TypeScript 是一个**带有类型**的超集，这意味着它增加了关于不同类型的值可以如何使用的规则。
前面提到的关于 `obj.heigth` 的错误并不是一个**语法**错误，而是以错误的方式使用了某种特定值（即某种**类型**）的错误。

再举一个例子，以下是可以在浏览器中运行的 JavaScript 代码，它**确实会**打印出一个值：

```js
console.log(4 / [])
```

这个语法合法的程序会打印出 `Infinity`。
然而，TypeScript 认为将数字除以数组是一个毫无意义的操作，因此会报错：

```ts twoslash
// @errors: 2363
console.log(4 / [])
```

或许你**确实**想要将数字除以数组（可能只是想看看会发生什么），但大多数情况下，这都是一个编程失误。
TypeScript 的类型检查器旨在放行正确的程序，同时尽可能多地捕获常见错误。
（稍后我们将学习一些配置项，用于调整 TypeScript 检查代码的严格程度。）

如果你将代码从 JavaScript 文件迁移到 TypeScript 文件中，根据代码的具体写法，你可能会看到一些**类型错误**。
这些错误可能是代码中确实存在的问题，也可能是因为 TypeScript 过于保守。
在整份指南中，我们将演示如何添加各种 TypeScript 语法来消除此类错误。

#### 运行时行为

TypeScript 也是一门保留 JavaScript **运行时行为**的编程语言。
例如，在 JavaScript 中除以零会得到 `Infinity`，而不是抛出运行时异常。
原则上，TypeScript **绝不**改变 JavaScript 代码的运行时行为。

这意味着，如果你将代码从 JavaScript 移动到 TypeScript，即便 TypeScript 认为该代码存在类型错误，也**保证**会以完全相同的方式运行。

保持与 JavaScript 相同的运行时行为是 TypeScript 的一项根本承诺，因为这意味着你可以在这两种语言之间轻松过渡，而不必担心因细微差异导致程序无法运行。

<!--
Missing subsection on the fact that TS extends JS to add syntax for type
specification.  (Since the immediately preceding text was raving about
how JS code can be used in TS.)
-->

#### 类型擦除

大体上，一旦 TypeScript 编译器完成对代码的检查，它就会**擦除**类型，从而生成最终的“编译后”代码。
这意味着代码一旦完成编译，生成的纯 JS 代码中将不再包含任何类型信息。

这也意味着 TypeScript 绝不会根据推断出的类型来改变程序的**行为**。
归根结底，尽管在编译期间可能会看到类型错误，但类型系统本身对程序运行时的行为没有任何影响。

最后，TypeScript 不提供任何额外的运行时库。
你的程序将使用与 JavaScript 程序相同的标准库（或外部库），因此无需学习任何特定于 TypeScript 的额外框架。

<!--
Should extend this paragraph to say that there's an exception of
allowing you to use newer JS features and transpile the code to an older
JS, and this might add small stubs of functionality when needed.  (Maybe
with an example --- something like `?.` would be good in showing readers
that this document is maintained.)
-->

## 学习 JavaScript 与 TypeScript

我们经常看到这样的问题：“我应该学习 JavaScript 还是 TypeScript？”。

答案是：不学 JavaScript，你就无法学会 TypeScript！
TypeScript 与 JavaScript 共享语法和运行时行为，因此你所学到的任何关于 JavaScript 的知识，同时也是在帮助你学习 TypeScript。

市面上有极其丰富的资源可供程序员学习 JavaScript；在编写 TypeScript 时，你**绝不应**忽视这些资源。
例如，StackOverflow 上带有 `javascript` 标签的问题数量大约是 `typescript` 的 20 倍，但**所有**关于 `javascript` 的问题同样适用于 TypeScript。

如果你发现自己在搜索类似“如何在 TypeScript 中对列表排序”的内容，请记住：**TypeScript 就是拥有编译时类型检查器的 JavaScript 运行时**。
你在 TypeScript 中对列表排序的方法，与在 JavaScript 中的方法完全一致。
如果你找到直接使用 TypeScript 的资源，那当然也很好，但不要把自己局限在认为日常解决运行时任务必须依赖 TypeScript 专属答案的思维中。

## 下一步

以上是对日常 TypeScript 中使用的语法和工具的简要概述。接下来，你可以：

- 学习一些 JavaScript 基础知识，我们推荐以下任意资源：

  - [Microsoft 的 JavaScript 资源](https://developer.microsoft.com/javascript/) 或
  - [Mozilla Web Docs 上的 JavaScript 指南](https://developer.mozilla.org/docs/Web/JavaScript/Guide)

- 继续阅读[针对 JavaScript 开发者的 TypeScript 指南](/docs/handbook/typescript-in-5-minutes.html)
- [从头到尾](/docs/handbook/intro.html)阅读完整手册
- 探索 [Playground 示例](/play#show-examples)

<!-- Note: I'll be happy to write the following... -->
<!--
## Types

    * What's a type? (For newbies)
      * A type is a *kind* of value
      * Types implicitly define what operations make sense on them
      * Lots of different kinds, not just primitives
      * We can make descriptions for all kinds of values
      * The `any` type -- a quick description, what it is, and why it's bad
    * Inference 101
      * Examples
      * TypeScript can figure out types most of the time
      * Two places we'll ask you what the type is: Function boundaries, and later-initialized values
    * Co-learning JavaScript
      * You can+should read existing JS resources
      * Just paste it in and see what happens
      * Consider turning off 'strict' -->
