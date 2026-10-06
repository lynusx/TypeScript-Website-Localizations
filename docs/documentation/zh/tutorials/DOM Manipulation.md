---
title: DOM 操作
layout: docs
permalink: /zh/docs/handbook/dom-manipulation.html
oneline: 在 TypeScript 中使用 DOM
translatable: true
---

## DOM 操作

### _深入探究 `HTMLElement` 类型_

自标准化以来的二十多年里，JavaScript 取得了长足的发展。虽然如今 JavaScript 已经可以运行在服务端、数据科学领域，甚至物联网（IoT）设备上，但我们切莫忘记它最主流的应用场景：Web 浏览器。

网站由 HTML 和/或 XML 文档构成。这些文档本身是静态的，不会自行改变。_文档对象模型（DOM）_是浏览器实现的一种编程接口，用于赋予静态网页交互功能。我们可以使用 DOM API 来改变文档的结构、样式以及内容。该 API 功能极其强大，以至于业界围绕它开发了无数前端框架（jQuery、React、Angular 等），使动态网站的开发变得更加简单。

TypeScript 是 JavaScript 的类型化超集，并且它自带了 DOM API 的类型定义。在任何默认的 TypeScript 项目中，这些定义都是开箱即用的。在 _lib.dom.d.ts_ 包含的 20,000 多行定义中，有一个类型尤为突出：`HTMLElement`。该类型是使用 TypeScript 进行 DOM 操作的基石。

> 你可以查看 [DOM 类型定义](https://github.com/microsoft/TypeScript/blob/main/tsc/internal/bundled/libs/lib.dom.d.ts) 的源码

## 基础示例

假设有如下简化的 _index.html_ 文件：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>TypeScript Dom Manipulation</title>
  </head>
  <body>
    <div id="app"></div>
    <!-- Assume index.js is the compiled output of index.ts -->
    <script src="index.js"></script>
  </body>
</html>
```

我们来看一段将 `<p>Hello, World!</p>` 元素添加到 `#app` 元素中的 TypeScript 脚本：

```ts
// 1. Select the div element using the id property
const app = document.getElementById('app')

// 2. Create a new <p></p> element programmatically
const p = document.createElement('p')

// 3. Add the text content
p.textContent = 'Hello, World!'

// 4. Append the p element to the div element
app?.appendChild(p)
```

编译并运行 _index.html_ 页面后，生成的 HTML 如下：

```html
<div id="app">
  <p>Hello, World!</p>
</div>
```

## `Document` 接口

上述 TypeScript 代码的第一行使用了一个全局变量 `document`。检查该变量可以发现，它是由 _lib.dom.d.ts_ 文件中的 `Document` 接口定义的。该代码片段调用了两个方法：`getElementById` 和 `createElement`。

### `Document.getElementById`

该方法的定义如下：

```ts
getElementById(elementId: string): HTMLElement | null;
```

传入一个元素 id 字符串，它将返回 `HTMLElement` 或 `null`。该方法引入了最重要的类型之一：`HTMLElement`。它是所有其他元素接口的基础接口。例如，代码示例中的 `p` 变量类型为 `HTMLParagraphElement`。另外需要注意，该方法可能会返回 `null`。这是因为在运行之前，该方法无法确定是否真的能找到指定的元素。在代码片段的最后一行中，使用了 _可选链（optional chaining）_ 运算符来调用 `appendChild`。

### `Document.createElement`

该方法的定义如下（此处省略了标记为已弃用（_deprecated_）的定义）：

```ts
createElement<K extends keyof HTMLElementTagNameMap>(tagName: K, options?: ElementCreationOptions): HTMLElementTagNameMap[K];
createElement(tagName: string, options?: ElementCreationOptions): HTMLElement;
```

这是一个重载函数定义。第二个重载最为简单，其工作方式与 `getElementById` 方法非常相似。传入任意 `string`，它都会返回一个标准的 HTMLElement。正是由于这个定义，开发者才能够创建独特的 HTML 元素标签。

例如，`document.createElement('xyz')` 会返回一个 `<xyz></xyz>` 元素，这显然不是 HTML 规范中指定的元素。

> 如果你感兴趣，可以使用 `document.getElementsByTagName` 与自定义标签元素进行交互

对于 `createElement` 的第一个定义，它使用了一些高级泛型模式。将其拆解为多个部分来理解最为轻松，首先从泛型表达式开始：`<K extends keyof HTMLElementTagNameMap>`。该表达式定义了一个泛型参数 `K`，它被_约束（constrained）_为接口 `HTMLElementTagNameMap` 的键。该映射接口包含了所有规范中指定的 HTML 标签名及其对应的类型接口。例如，以下是前 5 个映射值：

```ts
interface HTMLElementTagNameMap {
    "a": HTMLAnchorElement;
    "abbr": HTMLElement;
    "address": HTMLElement;
    "applet": HTMLAppletElement;
    "area": HTMLAreaElement;
        ...
}
```

有些元素并没有独特的属性，因此它们只会返回 `HTMLElement`；但其他类型具有独特的属性和方法，因此会返回各自特定的接口（这些接口继承自或实现了 `HTMLElement`）。

现在来看 `createElement` 定义的其余部分：`(tagName: K, options?: ElementCreationOptions): HTMLElementTagNameMap[K]`。第一个参数 `tagName` 被定义为泛型参数 `K`。TypeScript 编译器足够智能，可以根据该实参_推断（infer）_出泛型参数。这意味着开发者在调用该方法时无需显式指定泛型参数；传入 `tagName` 参数的任何值都会被推断为 `K`，并进而用于定义的其余部分。事实也确实如此：返回值 `HTMLElementTagNameMap[K]` 获取 `tagName` 参数并用它返回对应的类型。正是通过这种定义，代码片段中的 `p` 变量才获得了 `HTMLParagraphElement` 类型。如果代码写为 `document.createElement('a')`，那么它就是一个 `HTMLAnchorElement` 类型的元素。

## `Node` 接口

`document.getElementById` 函数返回一个 `HTMLElement`。`HTMLElement` 接口继承自 `Element` 接口，而 `Element` 接口又继承自 `Node` 接口。这种基于原型的继承关系使得所有 `HTMLElement` 都能使用一部分标准方法。在代码片段中，我们正是使用了定义在 `Node` 接口上的属性将新的 `p` 元素追加到网页中。

### `Node.appendChild`

代码片段的最后一行是 `app?.appendChild(p)`。在前面的 `document.getElementById` 小节中已经详细说明过，这里之所以使用_可选链（optional chaining）_运算符，是因为 `app` 在运行时可能为 null。`appendChild` 方法的定义如下：

```ts
appendChild<T extends Node>(newChild: T): T;
```

该方法的工作方式与 `createElement` 方法类似，泛型参数 `T` 会从 `newChild` 参数中推断得出。`T` 被_约束_为另一个基础接口 `Node`。

## `children` 与 `childNodes` 的区别

前文提到，`HTMLElement` 接口继承自 `Element`，而 `Element` 又继承自 `Node`。在 DOM API 中，存在“子元素”（_children_）的概念。例如在以下 HTML 中，`p` 标签是 `div` 元素的子元素：

```tsx
;<div>
  <p>Hello, World</p>
  <p>TypeScript!</p>
</div>

const div = document.getElementsByTagName('div')[0]

div.children
// HTMLCollection(2) [p, p]

div.childNodes
// NodeList(2) [p, p]
```

获取到 `div` 元素后，`children` 属性将返回一个包含 `HTMLParagraphElement` 的 `HTMLCollection` 列表。`childNodes` 属性则会返回一个类似的节点列表 `NodeList`。每个 `p` 标签仍然是 `HTMLParagraphElement` 类型，但 `NodeList` 可以包含 `HTMLCollection` 列表所不能包含的额外 _HTML 节点_。

修改 HTML，移除其中一个 `p` 标签，但保留其文本：

```tsx
;<div>
  <p>Hello, World</p>
  TypeScript!
</div>

const div = document.getElementsByTagName('div')[0]

div.children
// HTMLCollection(1) [p]

div.childNodes
// NodeList(2) [p, text]
```

观察这两个列表的变化。`children` 现在仅包含 `<p>Hello, World</p>` 元素，而 `childNodes` 包含一个 `text` 节点，而不是两个 `p` 节点。`NodeList` 中的 `text` 部分是包含文本 `TypeScript!` 的字面 `Node`。`children` 列表中不包含此 `Node`，因为它不被视为 `HTMLElement`。

## `querySelector` 和 `querySelectorAll` 方法

这两个方法都是根据更具体的约束条件获取 DOM 元素列表的优秀工具。它们在 _lib.dom.d.ts_ 中的定义如下：

```ts
/**
 * Returns the first element that is a descendant of node that matches selectors.
 */
querySelector<K extends keyof HTMLElementTagNameMap>(selectors: K): HTMLElementTagNameMap[K] | null;
querySelector<K extends keyof SVGElementTagNameMap>(selectors: K): SVGElementTagNameMap[K] | null;
querySelector<E extends Element = Element>(selectors: string): E | null;

/**
 * Returns all element descendants of node that match selectors.
 */
querySelectorAll<K extends keyof HTMLElementTagNameMap>(selectors: K): NodeListOf<HTMLElementTagNameMap[K]>;
querySelectorAll<K extends keyof SVGElementTagNameMap>(selectors: K): NodeListOf<SVGElementTagNameMap[K]>;
querySelectorAll<E extends Element = Element>(selectors: string): NodeListOf<E>;
```

`querySelectorAll` 的定义与 `getElementsByTagName` 类似，只是它返回了一个新类型：`NodeListOf`。该返回类型本质上是标准 JavaScript 列表元素的一种自定义实现。可以说，将 `NodeListOf<E>` 替换为 `E[]` 会带来非常相似的使用体验。`NodeListOf` 仅实现了以下属性和方法：`length`、`item(index)`、`forEach((value, key, parent) => void)` 以及数值索引。此外，该方法返回的是_元素（elements）_列表，而不是 `.childNodes` 方法所返回的_节点（nodes）_。虽然这看似有些出入，但请注意，`Element` 接口本身继承自 `Node`。

要查看这些方法的实际效果，请将现有代码修改为：

```tsx
;<ul>
  <li>First :)</li>
  <li>Second!</li>
  <li>Third times a charm.</li>
</ul>

const first = document.querySelector('li') // returns the first li element
const all = document.querySelectorAll('li') // returns the list of all li elements
```

## 想了解更多内容？

_lib.dom.d.ts_ 类型定义最棒的一点在于，它们与 Mozilla 开发者网络（MDN）文档网站上标注的类型高度契合。例如，MDN 上的 [HTMLElement 页面](https://developer.mozilla.org/docs/Web/API/HTMLElement) 记录了 `HTMLElement` 接口的详细信息。这些页面列出了所有可用的属性、方法，有时甚至还提供示例。这些页面的另一个优秀之处在于它们提供了对应标准文档的链接。这里是指向 [W3C HTMLElement 推荐规范](https://www.w3.org/TR/html52/dom.html#htmlelement) 的链接。

参考来源：

- [ECMA-262 标准](http://www.ecma-international.org/ecma-262/10.0/index.html)
- [DOM 简介](https://developer.mozilla.org/docs/Web/API/Document_Object_Model/Introduction)
