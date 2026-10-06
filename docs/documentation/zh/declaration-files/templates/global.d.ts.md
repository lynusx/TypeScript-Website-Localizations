---
title: '全局库 .d.ts'
layout: docs
permalink: /zh/docs/handbook/declaration-files/templates/global-d-ts.html
---

## 全局库

<!--
TODO:

1. mention that global nearly always means 'browser'
2. if you have a global library that you suspect is UMD, look for instructions on
   a. how to import it
   b. -OR- how to make it work with webpack
3. Make the page follow the structure of documentation,usage,source example.

-->

所谓全局（_global_）库，是指可以直接在全局作用域中访问的库（即无需使用任何形式的 `import`）。
许多库只是简单地暴露一个或多个全局变量供使用。
例如，如果你在使用 [jQuery](https://jquery.com/)，只需直接引用 `$` 变量即可：

```ts
$(() => {
  console.log('hello!')
})
```

在全局库的文档中，通常会指导你如何通过 HTML 的 script 标签来使用该库：

```html
<script src="http://a.great.cdn.for/someLib.js"></script>
```

如今，大多数流行的、可在全局访问的库实际上都是以 UMD 库的形式编写的（见下文）。
UMD 库的文档往往很难与全局库的文档区分开来。
在编写全局声明文件之前，请确保该库并非实际上是一个 UMD 库。

## 从代码中识别全局库

全局库的代码通常非常简单。
一个全局的 “Hello, world” 库可能长这样：

```js
function createGreeting(s) {
  return 'Hello, ' + s
}
```

或者长这样：

```js
window.createGreeting = function (s) {
  return 'Hello, ' + s
}
```

查看全局库的代码时，你通常会看到：

- 顶层的 `var` 语句或 `function` 声明
- 对 `window.someName` 的一次或多次赋值
- 假定存在像 `document` 或 `window` 这样的 DOM 原语

你_不会_看到：

- 对 `require` 或 `define` 等模块加载器的检测或使用
- 形如 `var fs = require("fs");` 的 CommonJS/Node.js 风格导入
- 对 `define(...)` 的调用
- 描述如何 `require` 或导入该库的文档

## 全局库示例

因为将全局库转换为 UMD 库通常很容易，所以目前很少有流行库仍然采用全局风格编写。
但是，体积较小且需要 DOM（或者_没有_任何依赖项）的库可能仍然是全局库。

## 全局库模板

你可以参考下方的 DTS 示例模板：

```ts
// Type definitions for [~库名称~] [~可选版本号~]
// Project: [~项目名称~]
// Definitions by: [~你的姓名~] <[~你的 URL~]>

/*~ 如果该库是可调用的（例如可以作为 myLib(3) 调用），
 *~ 请在此处包含这些调用签名。
 *~ 否则，请删除此部分。
 */
declare function myLib(a: string): string
declare function myLib(a: number): number

/*~ 如果希望该库的名称成为有效的类型名称，
 *~ 可以在此处声明。
 *~
 *~ 例如，这允许我们编写 'var x: myLib'；
 *~ 请确保这确实符合逻辑！如果不符合，
 *~ 只需删除此声明，并在下方的命名空间中添加类型即可。
 */
interface myLib {
  name: string
  length: number
  extras?: string[]
}

/*~ 如果你的库在全局变量上暴露了属性，
 *~ 请将它们放在这里。
 *~ 你还应该在此处放置类型（接口和类型别名）。
 */
declare namespace myLib {
  //~ 我们可以编写 'myLib.timeout = 50;'
  let timeout: number

  //~ 我们可以访问 'myLib.version'，但不能修改它
  const version: string

  //~ 存在某个类，我们可以通过 'let c = new myLib.Cat(42)' 创建实例
  //~ 或进行引用，例如 'function f(c: myLib.Cat) { ... }'
  class Cat {
    constructor(n: number)

    //~ 我们可以从 'Cat' 实例中读取 'c.age'
    readonly age: number

    //~ 我们可以从 'Cat' 实例上调用 'c.purr()'
    purr(): void
  }

  //~ 我们可以将变量声明为
  //~   'var s: myLib.CatSettings = { weight: 5, name: "Maru" };'
  interface CatSettings {
    weight: number
    name: string
    tailLength?: number
  }

  //~ 我们可以编写 'const v: myLib.VetID = 42;'
  //~  或 'const v: myLib.VetID = "bob";'
  type VetID = string | number

  //~ 我们可以调用 'myLib.checkCat(c)' 或 'myLib.checkCat(c, v);'
  function checkCat(c: Cat, s?: VetID)
}
```
