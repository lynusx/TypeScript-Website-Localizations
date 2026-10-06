---
title: 命名空间
layout: docs
permalink: /zh/docs/handbook/namespaces.html
oneline: TypeScript 命名空间的工作原理
translatable: true
---

> **术语说明：**
> 需要特别注意的是，在 TypeScript 1.5 中，术语发生了一些变化。
> “内部模块”（Internal modules）现已更名为“命名空间”（namespaces）。
> “外部模块”（External modules）现简称为“模块”（modules），以便与 [ECMAScript 2015](https://www.ecma-international.org/ecma-262/6.0/) 的术语保持一致（即 `module X {` 等价于现在推荐使用的 `namespace X {`）。

本文概述了在 TypeScript 中使用命名空间（旧称“内部模块”）组织代码的各种方式。
正如我们在术语说明中所提到的，“内部模块”现被称为“命名空间”。
此外，在声明内部模块时使用 `module` 关键字的任何地方，都可以且应当改用 `namespace` 关键字。
这样可以避免术语名称相近给新用户带来混淆。

## 第一步

让我们从一个贯穿本页作为示例的程序开始。
我们编写了一组简易的字符串验证器，类似于你在网页表单中验证用户输入，或检查外部提供的数据文件格式时所编写的代码。

## 单文件中的验证器

```ts
interface StringValidator {
  isAcceptable(s: string): boolean
}

let lettersRegexp = /^[A-Za-z]+$/
let numberRegexp = /^[0-9]+$/

class LettersOnlyValidator implements StringValidator {
  isAcceptable(s: string) {
    return lettersRegexp.test(s)
  }
}

class ZipCodeValidator implements StringValidator {
  isAcceptable(s: string) {
    return s.length === 5 && numberRegexp.test(s)
  }
}

// Some samples to try
let strings = ['Hello', '98052', '101']

// Validators to use
let validators: { [s: string]: StringValidator } = {}
validators['ZIP code'] = new ZipCodeValidator()
validators['Letters only'] = new LettersOnlyValidator()

// Show whether each string passed each validator
for (let s of strings) {
  for (let name in validators) {
    let isMatch = validators[name].isAcceptable(s)
    console.log(`'${s}' ${isMatch ? 'matches' : 'does not match'} '${name}'.`)
  }
}
```

## 使用命名空间

随着添加的验证器越来越多，我们需要某种组织方案，以便追踪我们的类型，并且不必担心与其他对象发生名称冲突。
与其将大量不同的名称引入全局命名空间，不如将这些对象包裹到一个命名空间中。

在这个示例中，我们将所有与验证器相关的实体移动到名为 `Validation` 的命名空间中。
因为我们希望此处的接口和类在命名空间外部可见，所以在它们前面加上了 `export`。
相反，变量 `lettersRegexp` 和 `numberRegexp` 属于实现细节，因此未被导出，在命名空间外部的代码中是不可见的。
在文件底部的测试代码中，当在命名空间外部使用这些类型时，我们现在需要限定其类型名称，例如 `Validation.LettersOnlyValidator`。

## 命名空间化的验证器

```ts
namespace Validation {
  export interface StringValidator {
    isAcceptable(s: string): boolean
  }

  const lettersRegexp = /^[A-Za-z]+$/
  const numberRegexp = /^[0-9]+$/

  export class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
      return lettersRegexp.test(s)
    }
  }

  export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
      return s.length === 5 && numberRegexp.test(s)
    }
  }
}

// Some samples to try
let strings = ['Hello', '98052', '101']

// Validators to use
let validators: { [s: string]: Validation.StringValidator } = {}
validators['ZIP code'] = new Validation.ZipCodeValidator()
validators['Letters only'] = new Validation.LettersOnlyValidator()

// Show whether each string passed each validator
for (let s of strings) {
  for (let name in validators) {
    console.log(
      `"${s}" - ${
        validators[name].isAcceptable(s) ? 'matches' : 'does not match'
      } ${name}`,
    )
  }
}
```

## 跨文件拆分

随着应用程序规模的扩大，我们会希望将代码拆分到多个文件中，以便于维护。

## 多文件命名空间

在这里，我们将 `Validation` 命名空间拆分到多个文件中。
即使这些文件是相互独立的，它们也可以共同为同一个命名空间贡献内容，并且可以像在同一个地方定义一样使用。
由于文件之间存在依赖关系，我们将添加引用标签，告知编译器这些文件之间的关系。
除此之外，我们的测试代码保持不变。

##### Validation.ts

```ts
namespace Validation {
  export interface StringValidator {
    isAcceptable(s: string): boolean
  }
}
```

##### LettersOnlyValidator.ts

```ts
/// <reference path="Validation.ts" />
namespace Validation {
  const lettersRegexp = /^[A-Za-z]+$/
  export class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
      return lettersRegexp.test(s)
    }
  }
}
```

##### ZipCodeValidator.ts

```ts
/// <reference path="Validation.ts" />
namespace Validation {
  const numberRegexp = /^[0-9]+$/
  export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
      return s.length === 5 && numberRegexp.test(s)
    }
  }
}
```

##### Test.ts

```ts
/// <reference path="Validation.ts" />
/// <reference path="LettersOnlyValidator.ts" />
/// <reference path="ZipCodeValidator.ts" />

// Some samples to try
let strings = ['Hello', '98052', '101']

// Validators to use
let validators: { [s: string]: Validation.StringValidator } = {}
validators['ZIP code'] = new Validation.ZipCodeValidator()
validators['Letters only'] = new Validation.LettersOnlyValidator()

// Show whether each string passed each validator
for (let s of strings) {
  for (let name in validators) {
    console.log(
      `"${s}" - ${
        validators[name].isAcceptable(s) ? 'matches' : 'does not match'
      } ${name}`,
    )
  }
}
```

一旦涉及多个文件，我们就需要确保所有编译后的代码都被加载进来。
有两种方式可以做到这一点。

首先，我们可以使用 [`outFile`](/tsconfig#outFile) 选项进行拼接输出，将所有输入文件编译为一个单一的 JavaScript 输出文件：

```Shell
tsc --outFile sample.js Test.ts
```

编译器将根据文件中存在的引用标签自动对输出文件进行排序。你也可以逐个指定每个文件：

```Shell
tsc --outFile sample.js Validation.ts LettersOnlyValidator.ts ZipCodeValidator.ts Test.ts
```

或者，我们可以使用单文件编译（默认方式），为每个输入文件生成一个 JavaScript 文件。
如果生成了多个 JS 文件，我们需要在网页中使用 `<script>` 标签按正确的顺序加载各个生成的文件，例如：

##### MyTestPage.html（节选）

```html
<script src="Validation.js" type="text/javascript" />
<script src="LettersOnlyValidator.js" type="text/javascript" />
<script src="ZipCodeValidator.js" type="text/javascript" />
<script src="Test.js" type="text/javascript" />
```

## 别名

简化命名空间使用的另一种方法是使用 `import q = x.y.z` 为常用对象创建较短的别名。
请不要将其与用于加载模块的 `import x = require("name")` 语法混淆，该语法仅仅是为指定的符号创建一个别名。
你可以将此类导入（通常称为别名）用于任何类型的标识符，包括从模块导入创建的对象。

```ts
namespace Shapes {
  export namespace Polygons {
    export class Triangle {}
    export class Square {}
  }
}

import polygons = Shapes.Polygons
let sq = new polygons.Square() // Same as 'new Shapes.Polygons.Square()'
```

请注意，我们并没有使用 `require` 关键字；而是直接从所导入符号的限定名进行赋值。
这与使用 `var` 类似，但它同时也适用于所导入符号的类型与命名空间含义。
重要的是，对于值而言，`import` 是与原始符号独立的一个引用，因此对别名化 `var` 的修改不会反映到原始变量中。

## 与其他 JavaScript 库协同工作

为了描述非 TypeScript 编写的库的形状（shape），我们需要声明该库公开的 API。
由于大多数 JavaScript 库只公开少数顶层对象，因此命名空间是表示它们的绝佳方式。

我们将没有定义实现的声明称为“环境声明”（ambient）。
通常它们定义在 `.d.ts` 文件中。
如果你熟悉 C/C++，可以将其视为 `.h` 头文件。
让我们来看几个示例。

## 环境命名空间

知名的 D3 库将其功能定义在一个名为 `d3` 的全局对象中。
因为该库是通过 `<script>` 标签加载的（而不是通过模块加载器），所以其声明使用命名空间来定义其形状。
为了让 TypeScript 编译器能够识别该形状，我们使用环境命名空间声明。
例如，我们可以像下面这样开始编写：

##### D3.d.ts（简化节选）

```ts
declare namespace D3 {
  export interface Selectors {
    select: {
      (selector: string): Selection
      (element: EventTarget): Selection
    }
  }

  export interface Event {
    x: number
    y: number
  }

  export interface Base extends Selectors {
    event: Event
  }
}

declare var d3: D3.Base
```
