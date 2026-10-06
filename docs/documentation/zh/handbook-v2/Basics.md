---
title: 基础
layout: docs
permalink: /zh/docs/handbook/2/basic-types.html
oneline: '学习 TypeScript 的第一步：基本类型。'
preamble: >
  <p>欢迎阅读本手册的第一页。如果这是你第一次接触 TypeScript——你可能更希望先从“<a href='https://www.typescriptlang.org/docs/handbook/intro.html#get-started'>开始起步</a>”指南之一开始阅读</a>
---

JavaScript 中的每一个值都具有一组可以通过执行不同操作来观察到的行为。
这听起来有些抽象，举个简单的例子，考虑我们可能会对名为 `message` 的变量执行的一些操作：

```js
// 访问 'message' 上的属性 'toLowerCase'
// 然后调用它
message.toLowerCase()

// 调用 'message'
message()
```

如果我们对此进行分解，第一行可运行的代码访问了一个名为 `toLowerCase` 的属性并调用了它。
第二行则尝试直接调用 `message`。

但假设我们不知道 `message` 的值具体是什么——这种情况非常普遍——我们就无法确切预知运行这些代码会得到什么结果。
每个操作的行为完全取决于我们最初拥有什么样的值。

- `message` 是可调用的吗？
- 它上面是否有一个名为 `toLowerCase` 的属性？
- 如果有，`toLowerCase` 本身是可调用的吗？
- 如果这两个值都是可调用的，它们会返回什么？

在编写 JavaScript 时，这些问题的答案通常都印在我们自己的脑海中，我们只能寄希望于自己没有搞错任何细节。

假设 `message` 是按如下方式定义的：

```js
const message = 'Hello World!'
```

正如你所猜测的，如果我们尝试运行 `message.toLowerCase()`，我们将得到同样的小写字符串。

那么第二行代码呢？
如果你熟悉 JavaScript，就会知道它会抛出异常并失败：

```txt
TypeError: message is not a function
```

如果我们能够避免这类错误，那就太好了。

当我们运行代码时，JavaScript 运行时选择执行什么操作的方式，是通过判断值的_类型_（_type_）——即该值具备什么样的行为和能力。
这也是那个 `TypeError` 所暗示的含义之一——它指出字符串 `"Hello World!"` 不能作为函数被调用。

对于某些值（例如原始类型 `string` 和 `number`），我们可以在运行时使用 `typeof` 运算符来识别它们的类型。
但对于其他事物（如函数），在运行时并没有相应的机制来识别它们的类型。
例如，考虑下面这个函数：

```js
function fn(x) {
  return x.flip()
}
```

通过阅读代码，我们可以_观察_出该函数只有在传入包含可调用的 `flip` 属性的对象时才能正常工作，但 JavaScript 并没有提供一种能在代码运行时进行检查的方式来暴露这些信息。
在纯 JavaScript 中，想要知道 `fn` 接收某个特定值时会做什么，唯一的办法就是调用它并观察会发生什么。
这种行为使得在代码运行之前很难预测其表现，这也意味着在编写代码时，你更难明确自己的代码到底会产生什么结果。

从这个角度来看，_类型_（_type_）的概念正是用来描述哪些值可以传递给 `fn`，而哪些值会导致程序崩溃。
JavaScript 本质上只提供了_动态_类型（_dynamic_ typing）——即运行代码来看看会发生什么。

另一种选择则是使用_静态_类型系统（_static_ type system），在代码运行_之前_对其预期行为做出预测。

## 静态类型检查

回想一下我们之前尝试将 `string` 作为函数调用时遇到的 `TypeError`。
_大多数人_都不希望在运行代码时遇到任何错误——因为这些都被视作 bug！
而在编写新代码时，我们也会尽最大努力避免引入新的 bug。

如果我们只添加了一点点代码，保存文件，重新运行代码，并立刻看到了错误，我们或许能迅速定位问题；但情况往往并非总是如此。
我们可能没有对功能进行足够充分的测试，以至于在实际中可能根本没有触发原本会被抛出的潜在错误！
或者，即使我们“幸运地”目睹了错误，此时可能也已经进行了大规模重构并添加了许多不同的代码，不得不从中大海捞针排查问题。

理想情况下，我们希望有一个工具能在代码运行_之前_帮助我们找出这些 bug。
这正是像 TypeScript 这样的静态类型检查器所做的事情。
_静态类型系统_（_Static type systems_）描述了程序运行时各种值的形状（shapes）与行为。
像 TypeScript 这样的类型检查器会利用这些信息，并在事情可能脱离正轨时及时提醒我们。

```ts twoslash
// @errors: 2349
const message = 'hello!'

message()
```

使用 TypeScript 运行上面这个示例，会在代码真正运行之前就向我们输出一条错误信息。

## 非异常失败（Non-exception Failures）

到目前为止，我们讨论的都是运行时错误等情况——即 JavaScript 运行时认为某些操作毫无意义并报错的场景。
之所以会出现这些情况，是因为 [ECMAScript 规范](https://tc39.github.io/ecma262/)明确规定了语言在遇到非预期情况时的具体行为。

例如，规范规定尝试调用不可调用的内容应当抛出错误。
这听起来可能是“理所当然的行为”，但你也可以合理推想，访问对象上不存在的属性似乎也应该抛出错误。
然而，JavaScript 却采取了不同的行为，而是返回了 `undefined`：

```js
const user = {
  name: 'Daniel',
  age: 26,
}

user.location // 返回 undefined
```

归根结底，静态类型系统必须做出权衡，决定哪些代码应该在其系统中被标记为错误——即使该代码是合法的 JavaScript 且不会立即抛出运行时错误。
在 TypeScript 中，以下代码会产生关于 `location` 未定义的错误：

```ts twoslash
// @errors: 2339
const user = {
  name: 'Daniel',
  age: 26,
}

user.location
```

尽管有时这意味着在表达方式上需要做出取舍，但其初衷是为了捕获程序中真正的 bug。
而且 TypeScript 能够捕获_非常多_真正的 bug。

例如：拼写错误（typos），

```ts twoslash
// @noErrors
const announcement = 'Hello World!'

// 你能多快发现拼写错误？
announcement.toLocaleLowercase()
announcement.toLocalLowerCase()

// 我们大概本来是想写这个……
announcement.toLocaleLowerCase()
```

未调用的函数（uncalled functions），

```ts twoslash
// @noUnusedLocals
// @errors: 2365
function flipCoin() {
  // 本意应为 Math.random()
  return Math.random < 0.5
}
```

或是基本的逻辑错误（basic logic errors）。

```ts twoslash
// @errors: 2367
const value = Math.random() < 0.5 ? 'a' : 'b'
if (value !== 'a') {
  // ...
} else if (value === 'b') {
  // 哎呀，不可达
}
```

## 赋能工具的类型系统

TypeScript 可以在我们代码中出现失误时捕获 bug。
这很棒，但 TypeScript _还_能在最初就防止我们犯下这些错误。

类型检查器拥有足够的信息来检查诸如我们是否在变量和其他属性上访问了正确的属性等事项。
一旦掌握了这些信息，它就可以开始_提示_你可能想要使用哪些属性。

这意味着 TypeScript 同样可用于辅助代码编辑，核心类型检查器可以在你在编辑器中键入时提供错误提示和代码自动补全。
这就是人们在谈论 TypeScript 中的工具支持（tooling）时所指的一部分内容。

<!-- prettier-ignore -->
```ts twoslash
// @noErrors
// @esModuleInterop
import express from "express";
const app = express();

app.get("/", function (req, res) {
  res.sen
//       ^|
});

app.listen(3000);
```

TypeScript 非常重视工具体验，而且其能力远不止键入时的自动补全和错误提示。
支持 TypeScript 的编辑器可以提供“快速修复”（quick fixes）来自动修复错误、提供重构功能以轻松重新组织代码，以及提供实用的导航功能，用于跳转到变量的定义，或者查找某个变量的所有引用。
所有这些功能都构建在类型检查器之上，并且完全跨平台，因此[你喜爱的编辑器很可能也支持 TypeScript](https://github.com/Microsoft/TypeScript/wiki/TypeScript-Editor-Support)。

## `tsc`：TypeScript 编译器

我们一直在讨论类型检查，但我们还没真正使用过类型_检查器_。
现在让我们来认识一下新朋友 `tsc`，即 TypeScript 编译器。
首先，我们需要通过 npm 安装它。

```sh
npm install -g typescript
```

> 这将全局安装 TypeScript 编译器 `tsc`。
> 如果你更倾向于从本地 `node_modules` 包中运行 `tsc`，也可以使用 `npx` 或类似工具。

现在让我们进入一个空文件夹，并尝试编写我们的第一个 TypeScript 程序：`hello.ts`：

```ts twoslash
// 向世界问好。
console.log('Hello world!')
```

请注意，这里没有任何多余的花哨代码；这个“hello world”程序看起来与你在 JavaScript 中编写的“hello world”程序一模一样。
现在让我们通过运行由 `typescript` 包为我们安装的 `tsc` 命令来进行类型检查。

```sh
tsc hello.ts
```

搞定！

等等，到底“搞定”了_什么_？
我们运行了 `tsc`，但什么都没发生！
好吧，因为没有类型错误，所以在控制台中没有任何输出，因为确实没有什么需要报告的。

但请再看一眼——实际上我们得到了_文件_输出。
如果查看当前目录，我们会看到 `hello.ts` 旁边出现了一个 `hello.js` 文件。
这就是在 `tsc` 将 `hello.ts` 文件_编译_或_转换_为纯 JavaScript 文件后的输出。
如果我们查看其内容，就能看到 TypeScript 处理 `.ts` 文件后生成的结果：

```js
// 向世界问好。
console.log('Hello world!')
```

在这种情况下，TypeScript 几乎没有什么需要转换的，因此它看起来与我们编写的代码完全相同。
编译器致力于输出清晰易读的代码，就像人类手写的一样。
虽然这并不总是那么容易做到，但 TypeScript 保持了一致的缩进，注意代码跨行的情况，并尽力保留注释。

那么，如果我们_确实_引入了类型检查错误会怎么样？
让我们重写 `hello.ts`：

```ts twoslash
// @noErrors
// 这是一个工业级的通用问候函数：
function greet(person, date) {
  console.log(`Hello ${person}, today is ${date}!`)
}

greet('Brendan')
```

如果我们再次运行 `tsc hello.ts`，注意我们在命令行中收到了一条错误！

```txt
Expected 2 arguments, but got 1.
```

TypeScript 正确地提醒我们忘记向 `greet` 函数传递实参了。
到目前为止，我们编写的依然只是标准的 JavaScript，但类型检查依然能够找出代码中的问题。
感谢 TypeScript！

## 报错时仍生成代码

在上面的例子中，你可能没有注意到一件事：我们的 `hello.js` 文件又发生了变化。
如果我们打开该文件，会发现其内容基本上仍然与我们的输入文件相同。
考虑到 `tsc` 刚刚报告了代码错误，这可能会让人有些意外，但这体现了 TypeScript 的核心价值观之一：多数时候，_你_比 TypeScript 更清楚自己在做什么。

正如前文重申的，对代码进行类型检查会限制你能够运行的程序种类，因此在类型检查器认为什么是可接受的这一问题上存在权衡。
大多数时候这都没问题，但在某些场景下，这些检查反而会造成阻碍。
例如，想象一下你正在将 JavaScript 代码迁移到 TypeScript，期间引入了类型检查错误。
你最终肯定会按照类型检查器的要求清理并修复这些问题，但原本的 JavaScript 代码本身已经在正常运行了！
为什么仅仅因为把它转成 TypeScript 就阻止你运行它呢？

因此，TypeScript 不会阻碍你的开发流程。
当然，随着时间的推移，你可能希望对错误采取更具防范性的态度，让 TypeScript 表现得更加严格。
在这种情况下，你可以使用 [`noEmitOnError`](/tsconfig#noEmitOnError) 编译器选项。
尝试修改你的 `hello.ts` 文件并带上该标志运行 `tsc`：

```sh
tsc --noEmitOnError hello.ts
```

你会发现 `hello.js` 根本没有被更新。

## 显式类型

到目前为止，我们还没有告诉 TypeScript `person` 或 `date` 是什么。
让我们修改代码，告诉 TypeScript `person` 是一个 `string`，而 `date` 应该是一个 `Date` 对象。
我们还会在 `date` 上调用 `toDateString()` 方法。

```ts twoslash
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`)
}
```

我们所做的是在 `person` 和 `date` 上添加了_类型注解_（_type annotations_），用以描述调用 `greet` 时可以传入什么类型的值。
你可以将该签名理解为“`greet` 接受一个 `string` 类型的 `person`，以及一个 `Date` 类型的 `date`”。

有了这些注解，TypeScript 就能在 `greet` 可能被错误调用的其他情况下向我们报错。
例如……

```ts twoslash
// @errors: 2345
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`)
}

greet('Maddison', Date())
```

咦？
TypeScript 在第二个实参上报错了，但这是为什么？

可能令人意外的是，在 JavaScript 中直接调用 `Date()` 返回的是一个 `string`。
而使用 `new Date()` 构造一个 `Date` 才会得到我们真正期望的对象。

无论如何，我们可以快速修复这个错误：

```ts twoslash {4}
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`)
}

greet('Maddison', new Date())
```

请记住，我们并不总是需要编写显式的类型注解。
在许多情况下，即使省略了类型注解，TypeScript 甚至可以直接为我们_推断_（或“推导”）出类型。

```ts twoslash
let msg = 'hello there!'
//  ^?
```

尽管我们没有告诉 TypeScript `msg` 的类型是 `string`，但它依然能够推导出来。
这是一项特性，而且在类型系统无论如何都会推断出相同类型的情况下，最好不要添加冗余的注解。

> 注意：上面代码示例中的消息气泡代表当你在编辑器中将鼠标悬停在该词上方时所展示的内容。

## 类型擦除

让我们看看使用 `tsc` 编译上述函数 `greet` 输出 JavaScript 时会发生什么：

```ts twoslash
// @showEmit
// @target: es5
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`)
}

greet('Maddison', new Date())
```

请注意这里的两点：

1. 我们的 `person` 和 `date` 形参不再带有类型注解。
2. 我们的“模板字符串”——即使用反引号（`` ` `` 字符）的字符串——被转换成了带有拼接操作的普通字符串。

关于第二点稍后会详细说明，现在让我们先关注第一点。
类型注解并不是 JavaScript（严格来说是 ECMAScript）的一部分，因此没有任何浏览器或其他运行时可以直接原样运行未经修改的 TypeScript。
这就是为什么 TypeScript 首先需要编译器的原因——它需要某种方法来剥离或转换所有特定于 TypeScript 的代码，以便你能够运行它。
绝大多数特定于 TypeScript 的代码都会被擦除，同样地，这里的类型注解也已被完全擦除。

> **请记住**：类型注解永远不会改变程序的运行时行为。

## 降级编译

上面的另一个区别在于，我们的模板字符串从

```js
;`Hello ${person}, today is ${date.toDateString()}!`
```

被重写为

```js
'Hello '.concat(person, ', today is ').concat(date.toDateString(), '!')
```

为什么会这样？

模板字符串是 ECMAScript 2015（又称 ECMAScript 6、ES2015、ES6 等——_说来话长_）版本中引入的特性。
TypeScript 具备将代码从较新版本的 ECMAScript 重写为较旧版本（如 ECMAScript 3 或 ECMAScript 5，即 ES5）的能力。
这种从较新或“较高”版本的 ECMAScript 向下迁移到较旧或“较低”版本的处理过程，有时被称为_降级编译_（_downleveling_）。

默认情况下，TypeScript 的目标版本是 ES5，这是一个非常古老的 ECMAScript 版本。
我们本可以通过使用 [`target`](/tsconfig#target) 选项来选择更新一点的版本。
使用 `--target es2015` 运行会将 TypeScript 的编译目标更改为 ECMAScript 2015，这意味着代码应该能够在任何支持 ECMAScript 2015 的环境中运行。
因此，运行 `tsc --target es2015 hello.ts` 会得到以下输出：

```js
function greet(person, date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`)
}
greet('Maddison', new Date())
```

> 尽管默认目标版本是 ES5，但绝大多数现代浏览器都支持 ES2015。
> 因此，除非对某些古老浏览器的兼容性至关重要，否则大多数开发者都可以放心地将 ES2015 或更高版本指定为目标。

## 严格程度

不同的用户使用 TypeScript 时，对类型检查器的诉求各不相同。
有些人希望获得一种更为宽松、按需选择的体验，只验证程序的某些部分，同时仍然享有良好的工具支持。
这就是 TypeScript 的默认体验：类型是可选的，推断采用最宽松的类型，并且不检查潜在的 `null`/`undefined` 值。
就像即使报错 `tsc` 依然会生成代码一样，这些默认配置的设立是为了不给你的开发带来阻碍。
如果你正在迁移现有的 JavaScript 代码，这可能是一个理想的起步阶段。

相比之下，很多用户更希望 TypeScript 从一开始就尽可能全面地进行校验，这也是该语言同样提供了严格程度设置的原因。
这些严格性设置将静态类型检查从非黑即白的开关（要么检查代码，要么不检查），变成了一个类似刻度盘的调节装置。
你把这个刻度盘拧得越高，TypeScript 为你检查的内容就越多。
这可能会需要一些额外的工作，但从长远来看，这绝对物超所值，并且能带来更全面的检查与更精确的工具支持。
在条件允许的情况下，新项目应始终开启这些严格性检查。

TypeScript 提供了若干可以开启或关闭的类型检查严格性标志，除非另有说明，我们所有的示例都将在开启所有这些标志的情况下编写。
CLI 中的 [`strict`](/tsconfig#strict) 标志，或者 [`tsconfig.json`](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html) 中的 `"strict": true` 可以同时开启所有这些检查，但我们也可以单独关闭其中的某一项。
其中你应该了解的两个最重要的标志是 [`noImplicitAny`](/tsconfig#noImplicitAny) 和 [`strictNullChecks`](/tsconfig#strictNullChecks)。

## `noImplicitAny`

回想一下，在某些地方，TypeScript 并不会尝试为我们推断类型，而是会回退到最宽松的类型：`any`。
这并不是什么大灾难——毕竟回退到 `any` 实际上就是普通的 JavaScript 体验。

然而，使用 `any` 往往违背了最初使用 TypeScript 的初衷。
程序中的类型定义越完备，你获得的验证和工具支持就越多，这意味着你在编码时遇到的 bug 也会越少。
开启 [`noImplicitAny`](/tsconfig#noImplicitAny) 标志后，任何类型被隐式推断为 `any` 的变量都会引发错误。

## `strictNullChecks`

默认情况下，`null` 和 `undefined` 等值可以赋值给任何其他类型。
这可能会让编写某些代码变得更轻松，但遗漏对 `null` 和 `undefined` 的处理是世界上无数 bug 的根源——甚至有人将其称为[价值十亿美元的错误](https://www.youtube.com/watch?v=ybrQvs4x0Ps)！
[`strictNullChecks`](/tsconfig#strictNullChecks) 标志使对 `null` 和 `undefined` 的处理更加显式，并让我们_免于_担忧自己是否_遗漏_了对 `null` 和 `undefined` 的处理。
