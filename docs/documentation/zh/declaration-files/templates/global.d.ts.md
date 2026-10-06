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
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ If this library is callable (e.g. can be invoked as myLib(3)),
 *~ include those call signatures here.
 *~ Otherwise, delete this section.
 */
declare function myLib(a: string): string
declare function myLib(a: number): number

/*~ If you want the name of this library to be a valid type name,
 *~ you can do so here.
 *~
 *~ For example, this allows us to write 'var x: myLib';
 *~ Be sure this actually makes sense! If it doesn't, just
 *~ delete this declaration and add types inside the namespace below.
 */
interface myLib {
  name: string
  length: number
  extras?: string[]
}

/*~ If your library has properties exposed on a global variable,
 *~ place them here.
 *~ You should also place types (interfaces and type alias) here.
 */
declare namespace myLib {
  //~ We can write 'myLib.timeout = 50;'
  let timeout: number

  //~ We can access 'myLib.version', but not change it
  const version: string

  //~ There's some class we can create via 'let c = new myLib.Cat(42)'
  //~ Or reference e.g. 'function f(c: myLib.Cat) { ... }
  class Cat {
    constructor(n: number)

    //~ We can read 'c.age' from a 'Cat' instance
    readonly age: number

    //~ We can invoke 'c.purr()' from a 'Cat' instance
    purr(): void
  }

  //~ We can declare a variable as
  //~   'var s: myLib.CatSettings = { weight: 5, name: "Maru" };'
  interface CatSettings {
    weight: number
    name: string
    tailLength?: number
  }

  //~ We can write 'const v: myLib.VetID = 42;'
  //~  or 'const v: myLib.VetID = "bob";'
  type VetID = string | number

  //~ We can invoke 'myLib.checkCat(c)' or 'myLib.checkCat(c, v);'
  function checkCat(c: Cat, s?: VetID)
}
```
