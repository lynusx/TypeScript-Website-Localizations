---
title: TypeScript 5.1
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-5-1.html
oneline: TypeScript 5.1 发布说明
---

## 对返回 `undefined` 的函数进行更宽松的隐式返回处理

在 JavaScript 中，如果函数在执行结束时没有遇到 `return` 语句，它就会返回 `undefined`。

```ts
function foo() {
  // no return
}
// x = undefined
let x = foo()
```

然而，在先前版本的 TypeScript 中，*唯一*可以完全没有 return 语句的函数，只有返回类型为 `void` 和 `any` 的函数。这意味着即便你显式声明“该函数返回 `undefined`”，你也必须包含至少一条 return 语句。

```ts
// ✅ fine - we inferred that 'f1' returns 'void'
function f1() {
  // no returns
}
// ✅ fine - 'void' doesn't need a return statement
function f2(): void {
  // no returns
}
// ✅ fine - 'any' doesn't need a return statement
function f3(): any {
  // no returns
}
// ❌ error!
// A function whose declared type is neither 'void' nor 'any' must return a value.
function f4(): undefined {
  // no returns
}
```

如果某些 API 期望接收一个返回 `undefined` 的函数，这就会显得格外繁琐——你必须要么显式返回一次 `undefined`，要么在提供显式类型标注的同时写上 `return` 语句。

```ts
declare function takesFunction(f: () => undefined): undefined
// ❌ error!
// Argument of type '() => void' is not assignable to parameter of type '() => undefined'.
takesFunction(() => {
  // no returns
})
// ❌ error!
// A function whose declared type is neither 'void' nor 'any' must return a value.
takesFunction((): undefined => {
  // no returns
})
// ❌ error!
// Argument of type '() => void' is not assignable to parameter of type '() => undefined'.
takesFunction(() => {
  return
})
// ✅ works
takesFunction(() => {
  return undefined
})
// ✅ works
takesFunction((): undefined => {
  return
})
```

这种行为不仅令人困扰，而且容易引起混淆，特别是在调用不受自己控制的函数时。去理解推断为 `void` 还是 `undefined` 之间的微妙差异、返回 `undefined` 的函数是否需要 `return` 语句等问题，往往会分散开发者的注意力。

首先，TypeScript 5.1 现在允许返回 `undefined` 的函数完全不包含 return 语句。

```ts
// ✅ Works in TypeScript 5.1!
function f4(): undefined {
  // no returns
}
// ✅ Works in TypeScript 5.1!
takesFunction((): undefined => {
  // no returns
})
```

其次，如果一个函数没有任何 return 表达式，并且被传递给期望接收返回 `undefined` 函数的位置，TypeScript 会将该函数的返回值类型推断为 `undefined`。

```ts
// ✅ Works in TypeScript 5.1!
takesFunction(function f() {
  //                 ^ return type is undefined
  // no returns
})
// ✅ Works in TypeScript 5.1!
takesFunction(function f() {
  //                 ^ return type is undefined
  return
})
```

为了解决另一个类似的痛点，在 TypeScript 的 `--noImplicitReturns` 选项下，*仅*返回 `undefined` 的函数现在也享有与 `void` 类似的例外规则，不再要求每一条代码分支都必须以显式的 `return` 结尾。

```ts
// ✅ Works in TypeScript 5.1 under '--noImplicitReturns'!
function f(): undefined {
  if (Math.random()) {
    // do some stuff...
    return
  }
}
```

欲了解更多信息，请查阅[原始 Issue](https://github.com/microsoft/TypeScript/issues/36288) 以及[对应的 Pull Request](https://github.com/microsoft/TypeScript/pull/53607)。

## Getter 与 Setter 支持完全不相关的类型

TypeScript 4.3 引入了允许成对的 `get` 与 `set` 访问器指定不同类型的能力。

```ts
interface Serializer {
  set value(v: string | number | boolean)
  get value(): string
}
declare let box: Serializer
// Allows writing a 'boolean'
box.value = true
// Comes out as a 'string'
console.log(box.value.toUpperCase())
```

起初，我们要求 `get` 的类型必须是 `set` 类型的子类型。这意味着如下写法：

```ts
box.value = box.value
```

始终是有效的。

然而，在现有以及拟议的 API 中，存在大量 getter 与 setter 类型完全不相关的情况。例如最常见的一个例子——DOM 中的 `style` 属性以及 [`CSSStyleRule`](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleRule) API。每个样式规则都拥有[一个 `style` 属性](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleRule/style)，其读取类型为 [`CSSStyleDeclaration`](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration)；但是，如果你尝试向该属性写入值，它只接受字符串！

TypeScript 5.1 现在允许 `get` 与 `set` 访问器属性拥有完全不相关的类型，前提是它们具有显式的类型标注。尽管当前版本的 TypeScript 尚未更改这些内置接口的类型，但 `CSSStyleRule` 现在可以定义为如下形式：

```ts
interface CSSStyleRule {
  // ...
  /** Always reads as a `CSSStyleDeclaration` */
  get style(): CSSStyleDeclaration
  /** Can only write a `string` here. */
  set style(newValue: string)
  // ...
}
```

这也支持了其他设计模式，例如要求 `set` 访问器仅接受“有效”数据，但允许 `get` 访问器在底层状态尚未初始化时返回 `undefined`。

```ts
class SafeBox {
  #value: string | undefined
  // Only accepts strings!
  set value(newValue: string) {}
  // Must check for 'undefined'!
  get value(): string | undefined {
    return this.#value
  }
}
```

事实上，这与 `--exactOptionalProperties` 下对可选属性的检查方式类似。

欲了解更多信息，请查阅[对应的 Pull Request](https://github.com/microsoft/TypeScript/pull/53417)。

## JSX 元素与 JSX 标签类型的类型检查解耦

TypeScript 在处理 JSX 时的一个痛点，在于它对每个 JSX 元素标签类型的严苛要求。

作为背景，JSX 元素通常是以下两者之一：

```tsx
// A self-closing JSX tag
<Foo />
// A regular element with an opening/closing tag
<Bar></Bar>
```

在对 `<Foo />` 或 `<Bar></Bar>` 进行类型检查时，TypeScript 总是会在名为 `JSX` 的命名空间中查找名为 `Element` 的类型——更直接地说，就是查找 `JSX.Element`。

但在检查 `Foo` 或 `Bar` 本身是否可以作为合法的标签名时，TypeScript 过去大致只会获取 `Foo` 或 `Bar` 返回或构造的类型，并检查其是否与 `JSX.Element` 兼容（如果是可构造类型，则检查是否与另一个名为 `JSX.ElementClass` 的类型兼容）。

这种限制意味着：如果组件返回或“渲染”了比 `JSX.Element` 更宽泛的类型，它们就无法被正常使用。例如，某个 JSX 库可能完全允许组件返回 `string` 或 `Promise`。

举一个更具体的例子：[React 正在考虑为返回 `Promise` 的组件添加有限支持](https://github.com/acdlite/rfcs/blob/first-class-promises/text/0000-first-class-support-for-promises.md)，但在旧版 TypeScript 中，除非有人大幅放宽 `JSX.Element` 的类型，否则根本无法表达这一特性。

```tsx
import * as React from 'react'
async function Foo() {
  return <div></div>
}
let element = <Foo />
//             ~~~
// 'Foo' cannot be used as a JSX component.
//   Its return type 'Promise<Element>' is not a valid JSX element.
```

为了给库作者提供表达这种需求的能力，TypeScript 5.1 现在会查找名为 `JSX.ElementType` 的类型。`ElementType` 精确指定了哪些类型可以作为 JSX 元素中的合法标签。因此在今天，它的定义可能类似于：

```tsx
namespace JSX {
    export type ElementType =
        // All the valid lowercase tags
        keyof IntrinsicAttributes
        // Function components
        (props: any) => Element
        // Class components
        new (props: any) => ElementClass;
    export interface IntrinsicAttributes extends /*...*/ {}
    export type Element = /*...*/;
    export type ElementClass = /*...*/;
}
```

衷心感谢 [Sebastian Silbermann](https://github.com/eps1lon) 贡献了[该改进](https://github.com/microsoft/TypeScript/pull/51328)！

## 带命名空间的 JSX 属性名

TypeScript 现在在使用 JSX 时支持带命名空间的属性名。

```tsx
import * as React from 'react'
// Both of these are equivalent:
const x = <Foo a:b="hello" />
const y = <Foo a:b="hello" />
interface FooProps {
  'a:b': string
}
function Foo(props: FooProps) {
  return <div>{props['a:b']}</div>
}
```

当名称的第一段为小写名称时，带命名空间的标签名也会以类似的方式在 `JSX.IntrinsicAttributes` 上查找。

```tsx
// In some library's code or in an augmentation of that library:
namespace JSX {
  interface IntrinsicElements {
    ['a:b']: { prop: string }
  }
}
// In our code:
let x = <a:b prop="hello!" />
```

感谢 [Oleksandr Tarasiuk](https://github.com/a-tarasyuk) 提供了[这项贡献](https://github.com/microsoft/TypeScript/pull/53799)。

## 模块解析中会参考 `typeRoots`

当 TypeScript 指定的模块查找策略无法解析路径时，现在会相对于指定的 `typeRoots` 来解析包。

更多详情请参阅[此 Pull Request](https://github.com/microsoft/TypeScript/pull/51715)。

## 将声明移动到现有文件

除了支持将声明移动到新文件外，TypeScript 现在还提供了一项预览功能：支持将声明移动到现有文件中。你可以在较新版本的 Visual Studio Code 中体验此功能。

![Moving a function 'getThanks' to an existing file in the workspace.](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2023/05/moveToFile-5.1-preview.gif)

请注意，该功能目前处于预览阶段，我们正在收集进一步的反馈。

https://github.com/microsoft/TypeScript/pull/53542

## JSX 标签的关联光标编辑

TypeScript 现在支持 JSX 标签名称的*关联编辑*（linked editing）。关联编辑（有时也称为“镜像光标”）允许编辑器自动同时编辑多个位置。

![An example of JSX tags with linked editing modifying a JSX fragment and a div element.](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2023/04/linkedEditingJsx-5.1-1.gif)

这项新功能在 TypeScript 和 JavaScript 文件中均可使用，并可以在 Visual Studio Code Insiders 中启用。在 Visual Studio Code 中，你既可以在设置界面中修改 `Editor: Linked Editing` 选项：

![Visual Studio Code's Editor: Linked Editing` option](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2023/04/linkedEditing-5.1-vscode-ui-1.png)

也可以在 JSON 配置文件中配置 `editor.linkedEditing`：

```jsonc
{
  // ...
  "editor.linkedEditing": true,
}
```

Visual Studio 17.7 Preview 1 也将支持该功能。

你可以在此处查看[我们对联动编辑的实现](https://github.com/microsoft/TypeScript/pull/53284)！

## `@param` JSDoc 标记的代码片段补全

在 TypeScript 和 JavaScript 文件中输入 `@param` 标记时，TypeScript 现在都会提供代码片段补全。当你在编写代码文档或在 JavaScript 中添加 JSDoc 类型时，这有助于减少键盘输入和光标来回跳转。

![An example of completing JSDoc `param` comments on an 'add' function.](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2023/04/paramTagSnippets-5-1-1.gif)

你可以在 GitHub 上[查看该新功能的实现方式](https://github.com/microsoft/TypeScript/pull/53260)。

## 优化

### 避免不必要的类型实例化

TypeScript 5.1 现在会避免在已知不包含外部类型参数引用的对象类型中执行类型实例化。这有望减少大量不必要的计算，并将 [material-ui 文档目录](https://github.com/mui/material-ui/tree/b0351248fb396001a30330daac86d0e0794a0c1d/docs)的类型检查时间缩短了 50% 以上。

你可以在 GitHub 上[查看该项改进所涉及的变更](https://github.com/microsoft/TypeScript/pull/53246)。

### 字面量联合类型的否定情况检查

在检查源类型是否属于联合类型的一部分时，TypeScript 首先会使用该源类型的内部类型标识符执行快速查找。如果该查找失败，TypeScript 才会逐一检查与联合类型中每个类型的兼容性。

当将一个字面量类型与纯字面量类型的联合类型进行关联时，TypeScript 现在可以避免对联合中的每个其他类型进行完整遍历。这一假设是安全的，因为 TypeScript 总是会对字面量类型进行内部缓存（intern/cache）——不过仍需处理一些与“全新”（fresh）字面量类型相关的边缘情况。

[这项优化](https://github.com/microsoft/TypeScript/pull/53192)成功将[此 Issue 中的代码](https://github.com/microsoft/TypeScript/issues/53191)的类型检查时间从约 45 秒缩短至约 0.4 秒。

### 减少 JSDoc 解析中对扫描器（Scanner）的调用

在较旧版本的 TypeScript 中解析 JSDoc 注释时，会使用扫描器/词法分析器（scanner/tokenizer）将注释拆分成细粒度的 token，然后再将内容重新拼装在一起。这有助于规范化注释文本（例如将多个空格折叠为一个）；但其交互极其频繁琐碎，意味着解析器和扫描器需要非常频繁地来回跳转，从而为 JSDoc 解析带来了额外开销。

TypeScript 5.1 将更多拆分 JSDoc 注释的逻辑移到了扫描器/词法分析器内部。扫描器现在会直接将更大的内容块返回给解析器，供其按需处理。

[这些改动](https://github.com/microsoft/TypeScript/pull/53081)将几个大小约为 10MB、以自然语言注释为主的 JavaScript 文件的解析时间缩短了大约一半。再看一个更具实际意义的例子：在我们的性能测试套件中，[xstate](https://github.com/statelyai/xstate) 的快照解析时间减少了约 300ms，使得加载和分析速度更快。

## 破坏性变更

### 最低运行环境要求提升至 ES2020 和 Node.js 14.17

TypeScript 5.1 现在引入了 ECMAScript 2020 中加入的 JavaScript 特性。因此，TypeScript 至少必须在较新的运行环境中运行。对于大多数用户而言，这意味着 TypeScript 现在只能在 Node.js 14.17 及更高版本上运行。

如果你尝试在较旧版本的 Node.js（例如 Node 10 或 12）下运行 TypeScript 5.1，在运行 `tsc.js` 或 `tsserver.js` 时可能会看到如下错误：

```
node_modules/typescript/lib/tsserver.js:2406
  for (let i = startIndex ?? 0; i < array.length; i++) {
                           ^

SyntaxError: Unexpected token '?'
    at wrapSafe (internal/modules/cjs/loader.js:915:16)
    at Module._compile (internal/modules/cjs/loader.js:963:27)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:1027:10)
    at Module.load (internal/modules/cjs/loader.js:863:32)
    at Function.Module._load (internal/modules/cjs/loader.js:708:14)
    at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:60:12)
    at internal/main/run_main_module.js:17:47
```

此外，如果你尝试安装 TypeScript，npm 可能会输出如下错误信息：

```
npm WARN EBADENGINE Unsupported engine {
npm WARN EBADENGINE   package: 'typescript@5.1.1-rc',
npm WARN EBADENGINE   required: { node: '>=14.17' },
npm WARN EBADENGINE   current: { node: 'v12.22.12', npm: '8.19.2' }
npm WARN EBADENGINE }
```

来自 Yarn 的错误信息：

```
error typescript@5.1.1-rc: The engine "node" is incompatible with this module. Expected version ">=14.17". Got "12.22.12"
error Found incompatible module.
```

<!-- or from pnpm -->

[关于此变更的更多信息请参阅此处](https://github.com/microsoft/TypeScript/pull/53291)。

### 显式指定 `typeRoots` 将禁用对 `node_modules/@types` 的向上遍历查找

此前，当在 `tsconfig.json` 中指定了 `typeRoots` 选项，但解析到任一 `typeRoots` 目录失败时，TypeScript 仍会继续向上遍历父级目录，尝试在每个父级目录的 `node_modules/@types` 文件夹中解析包。

这种行为可能会引发过多的查找，因此在 TypeScript 5.1 中已被禁用。因此，基于 `tsconfig.json` 的 `types` 选项或 `/// <reference >` 指令中的条目，你可能会开始遇到如下错误：

```
error TS2688: Cannot find type definition file for 'node'.
error TS2688: Cannot find type definition file for 'mocha'.
error TS2688: Cannot find type definition file for 'jasmine'.
error TS2688: Cannot find type definition file for 'chai-http'.
error TS2688: Cannot find type definition file for 'webpack-env"'.
```

解决方案通常是在 `typeRoots` 中显式添加 `node_modules/@types` 的具体条目：

```jsonc
{
  "compilerOptions": {
    "types": ["node", "mocha"],
    "typeRoots": [
      // Keep whatever you had around before.
      "./some-custom-types/",
      // You might need your local 'node_modules/@types'.
      "./node_modules/@types",
      // You might also need to specify a shared 'node_modules/@types'
      // if you're using a "monorepo" layout.
      "../../node_modules/@types",
    ],
  },
}
```

更多信息可以在[我们 Issue 追踪器上的原始变更](https://github.com/microsoft/TypeScript/pull/51715)中找到。
