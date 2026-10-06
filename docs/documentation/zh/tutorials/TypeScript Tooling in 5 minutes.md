---
title: 5 分钟上手 TypeScript 工具链
layout: docs
permalink: /zh/docs/handbook/typescript-tooling-in-5-minutes.html
oneline: 了解如何使用 TypeScript 构建小型网站的教程
translatable: true
---

让我们从使用 TypeScript 构建一个简单的 Web 应用程序开始。

## 安装 TypeScript

主要有两种方式将 TypeScript 添加到项目中：

- 通过 npm（Node.js 包管理器）
- 通过安装 TypeScript 的 Visual Studio 插件

Visual Studio 2017 和 Visual Studio 2015 Update 3 默认内置了 TypeScript 语言支持，但并未包含 TypeScript 编译器 `tsc`。
如果你未通过 Visual Studio 安装 TypeScript，仍然可以[下载它](/download)。

对于 npm 用户：

```shell
> npm install -g typescript
```

## 构建你的第一个 TypeScript 文件

在编辑器中新建 `greeter.ts`，并输入以下 JavaScript 代码：

```ts twoslash
// @noImplicitAny: false
function greeter(person) {
  return 'Hello, ' + person
}

let user = 'Jane User'

document.body.textContent = greeter(user)
```

## 编译代码

虽然我们使用了 `.ts` 扩展名，但这段代码本质上就是 JavaScript。
你完全可以直接从现有的 JavaScript 应用程序中将其复制粘贴过来。

在命令行中运行 TypeScript 编译器：

```shell
tsc greeter.ts
```

运行结果会生成一个 `greeter.js` 文件，其中包含与输入完全相同的 JavaScript 代码。
至此，我们已经成功在 JavaScript 应用程序中用上了 TypeScript！

现在我们可以开始利用 TypeScript 带来的一些新工具了。
为 `person` 函数参数添加 `: string` 类型注解，如下所示：

```ts twoslash
function greeter(person: string) {
  return 'Hello, ' + person
}

let user = 'Jane User'

document.body.textContent = greeter(user)
```

## 类型注解

TypeScript 中的类型注解是一种用于记录函数或变量预期约定的轻量级方式。
在此示例中，我们希望调用 `greeter` 函数时传入一个字符串参数。
我们可以尝试修改调用代码，改为传入一个数组：

```ts twoslash
// @errors: 2345
function greeter(person: string) {
  return 'Hello, ' + person
}

let user = [0, 1, 2]

document.body.textContent = greeter(user)
```

重新编译后，你将看到一个错误：

```shell
error TS2345: Argument of type 'number[]' is not assignable to parameter of type 'string'.
```

类似地，尝试去掉调用 `greeter` 时的所有参数。
TypeScript 会提示你调用该函数时传入的参数数量与预期不符。
在这两种情况下，TypeScript 都能根据你的代码结构和所提供的类型注解进行静态分析。

请注意，即使出现了错误，`greeter.js` 文件依然被创建了。
即使代码存在错误，你也可以使用 TypeScript；但在这种情况下，TypeScript 会发出警告，提示代码很可能无法按预期运行。

## 接口

让我们进一步拓展这个示例。在这里，我们使用一个接口来描述具有 `firstName` 和 `lastName` 字段的对象。
在 TypeScript 中，只要两种类型的内部结构兼容，它们就是相互兼容的。
这使我们只需拥有接口所要求的形状即可实现该接口，而无需显式使用 `implements` 子句。

```ts twoslash
interface Person {
  firstName: string
  lastName: string
}

function greeter(person: Person) {
  return 'Hello, ' + person.firstName + ' ' + person.lastName
}

let user = { firstName: 'Jane', lastName: 'User' }

document.body.textContent = greeter(user)
```

## 类

最后，让我们通过类来做最后一次示例扩展。
TypeScript 支持 JavaScript 的新特性，例如对基于类的面向对象编程的支持。

在这里，我们将创建一个包含构造函数和若干公共字段的 `Student` 类。
请注意，类与接口能够很好地配合工作，让程序员可以自由选择合适的抽象层级。

另外值得注意的是，在构造函数参数上使用 `public` 是一种简写语法，它可以自动创建同名属性。

```ts twoslash
class Student {
  fullName: string
  constructor(
    public firstName: string,
    public middleInitial: string,
    public lastName: string,
  ) {
    this.fullName = firstName + ' ' + middleInitial + ' ' + lastName
  }
}

interface Person {
  firstName: string
  lastName: string
}

function greeter(person: Person) {
  return 'Hello, ' + person.firstName + ' ' + person.lastName
}

let user = new Student('Jane', 'M.', 'User')

document.body.textContent = greeter(user)
```

重新运行 `tsc greeter.ts`，你会发现生成的 JavaScript 代码与之前的代码相同。
TypeScript 中的类其实只是 JavaScript 中常用的基于原型的面向对象编程的一种简写形式。

## 运行你的 TypeScript Web 应用程序

现在，在 `greeter.html` 中输入以下内容：

```html
<!DOCTYPE html>
<html>
  <head>
    <title>TypeScript Greeter</title>
  </head>
  <body>
    <script src="greeter.js"></script>
  </body>
</html>
```

在浏览器中打开 `greeter.html`，即可运行你的第一个简单的 TypeScript Web 应用程序！

可选步骤：在 Visual Studio 中打开 `greeter.ts`，或者将代码复制到 TypeScript 演练场中。
你可以将光标悬停在标识符上查看其类型。
请注意，在某些情况下，这些类型会自动为你推断出来。
重新输入最后一行代码，你将看到基于 DOM 元素类型的自动补全列表和参数帮助提示。
将光标置于对 `greeter` 函数的引用上，按 F12 即可跳转到其定义。
此外，你还可以右键单击某个符号并使用重构功能进行重命名。

所提供的类型信息与各种工具相辅相成，助力在应用级规模下使用 JavaScript。
若想查看更多展示 TypeScript 能力的示例，请参阅本网站的示例（Samples）部分。

![Visual Studio picture](/images/docs/greet_person.png)
