---
title: 命名空间和模块
layout: docs
permalink: /zh/docs/handbook/namespaces-and-modules.html
oneline: 如何在 TypeScript 中通过模块或命名空间组织代码
translatable: true
---

本文概述了在 TypeScript 中使用模块和命名空间组织代码的各种方式。
我们还将探讨关于如何使用命名空间和模块的一些高级主题，并介绍在 TypeScript 中使用它们时的一些常见陷阱。

有关 ES 模块的更多信息，请参阅[模块](/docs/handbook/modules.html)文档。
有关 TypeScript 命名空间的更多信息，请参阅[命名空间](/docs/handbook/namespaces.html)文档。

注意：在 TypeScript 的_非常_早期的版本中，命名空间被称为“内部模块”（Internal Modules），它们早于 JavaScript 模块系统的出现。

## 使用模块

模块既可以包含代码，也可以包含声明。

模块还依赖于模块加载器（例如 CommonJS/Require.js）或支持 ES 模块的运行时环境。
模块提供了更好的代码复用性、更强的隔离性以及对打包工具更好的支持。

还值得注意的是，对于 Node.js 应用程序，模块是默认选项，并且**在现代代码中，我们推荐使用模块而不是命名空间**。

从 ECMAScript 2015 开始，模块已成为语言的原生组成部分，所有符合规范的引擎实现都应予以支持。
因此，对于新项目，推荐将模块作为代码组织机制。

## 使用命名空间

命名空间是 TypeScript 特有的一种组织代码的方式。  
命名空间本质上只是全局命名空间中具名的 JavaScript 对象。
这使得命名空间成为一种非常易于使用的结构。
与模块不同，它们可以跨越多个文件，并且可以使用 [`outFile`](/tsconfig#outFile) 进行拼接。
在 Web 应用程序中，命名空间可以是组织代码的一种不错方式，所有依赖项都可以作为 `<script>` 标签引入到 HTML 页面中。

但就像所有全局命名空间污染一样，组件之间的依赖关系可能很难识别，尤其是在大型应用程序中。

## 命名空间和模块的常见陷阱

在本节中，我们将介绍使用命名空间和模块时的各种常见陷阱，以及如何避免它们。

### 对模块使用 `/// <reference>`

一个常见的错误是尝试使用 `/// <reference ... />` 语法来引用模块文件，而不是使用 `import` 语句。
为了理解这种区别，我们首先需要了解编译器如何根据 `import` 的路径（例如 `import x from "...";`、`import x = require("...");` 等中的 `...` 路径）来定位模块的类型信息。

编译器会尝试查找对应路径下的 `.ts`、`.tsx`，然后是 `.d.ts` 文件。
如果找不到特定文件，编译器就会去查找_环境模块声明_（ambient module declaration）。
回想一下，这些需要在 `.d.ts` 文件中进行声明。

- `myModules.d.ts`

  ```ts
  // 在 .d.ts 文件或非模块的 .ts 文件中：
  declare module 'SomeModule' {
    export function fn(): string
  }
  ```

- `myOtherModule.ts`

  ```ts
  /// <reference path="myModules.d.ts" />
  import * as m from 'SomeModule'
  ```

此处的引用标签使我们能够定位包含环境模块声明的声明文件。
几个 TypeScript 示例中所使用的 `node.d.ts` 文件就是通过这种方式引用的。

### 不必要的命名空间嵌套

如果要将程序从命名空间转换为模块，很容易写出类似下面这样的文件：

- `shapes.ts`

  ```ts
  export namespace Shapes {
    export class Triangle {
      /* ... */
    }
    export class Square {
      /* ... */
    }
  }
  ```

这里的顶层命名空间 `Shapes` 无缘无故地包裹了 `Triangle` 和 `Square`。
这会让模块的使用者感到困惑和繁琐：

- `shapeConsumer.ts`

  ```ts
  import * as shapes from './shapes'
  let t = new shapes.Shapes.Triangle() // shapes.Shapes？
  ```

TypeScript 中模块的一个关键特性是：两个不同的模块绝不会向同一个作用域添加名称。
由于模块的使用者可以自行决定为其赋予什么名称，因此完全没有必要主动在命名空间中包裹导出的符号。

再次重申为什么不应该对模块内容使用命名空间：命名空间的核心思想是提供结构的逻辑分组并防止名称冲突。
由于模块文件本身已经是一个逻辑分组，并且其顶层名称由导入它的代码所定义，因此没有必要为导出的对象额外增加一层命名空间。

下面是修改后的示例：

- `shapes.ts`

  ```ts
  export class Triangle {
    /* ... */
  }
  export class Square {
    /* ... */
  }
  ```

- `shapeConsumer.ts`

  ```ts
  import * as shapes from './shapes'
  let t = new shapes.Triangle()
  ```

### 模块的权衡

正如 JS 文件与模块之间存在一一对应关系一样，TypeScript 在模块源文件与其生成的 JS 文件之间也存在一一对应关系。
其产生的一个影响是：取决于所面向的目标模块系统，可能无法将多个模块源文件拼接在一起。
例如，当目标模块系统为 `commonjs` 或 `umd` 时，不能使用 [`outFile`](/tsconfig#outFile) 选项；但在 TypeScript 1.8 及更高版本中，当目标为 `amd` 或 `system` 时，[可以使用](./release-notes/typescript-1-8.html#concatenate-amd-and-system-modules-with---outfile) [`outFile`](/tsconfig#outFile)。
