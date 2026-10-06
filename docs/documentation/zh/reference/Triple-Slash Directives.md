---
title: 三斜线指令
layout: docs
permalink: /zh/docs/handbook/triple-slash-directives.html
oneline: 如何在 TypeScript 中使用三斜线指令
translatable: true
---

三斜线指令是包含单个 XML 标签的单行注释。
注释的内容会被用作编译器指令。

三斜线指令**仅**在其所在文件的最顶端有效。
在三斜线指令之前，只能出现单行或多行注释（包括其他三斜线指令）。
如果三斜线指令出现在某条语句或声明之后，它们将被当作普通的单行注释处理，不具备任何特殊含义。

自 TypeScript 5.5 起，编译器不再生成引用指令，并且**不会**将手写的三斜线指令输出到目标文件中，除非这些指令被标记为 [`preserve="true"`](#preservetrue)。

## `/// <reference path="..." />`

在这一类指令中，`/// <reference path="..." />` 是最常用的一条指令。
它用于声明文件之间的*依赖关系*。

三斜线引用用于指示编译器在编译过程中包含其他文件。

当使用 [`out`](/tsconfig#out) 或 [`outFile`](/tsconfig#outFile) 时，它们还可以作为对输出结果进行排序的一种方式。
在预处理环节之后，文件将按照与输入相同的顺序输出到目标文件位置。

### 预处理输入文件

编译器会对输入文件执行预处理操作，以解析所有的三斜线引用指令。
在此过程中，会有额外的文件被添加到编译中。

该过程从一组*根文件*（root files）开始；
这些根文件是在命令行中指定的，或者是 `tsconfig.json` 文件中 [`files`](/tsconfig#files) 列表里指定的文件名。
这些根文件会按照指定的顺序进行预处理。
在将某个文件添加到列表之前，其中的所有三斜线引用都会被处理，且其目标文件也会被包含进来。
三斜线引用会按照在文件中出现的顺序，以深度优先的方式进行解析。

如果使用了相对路径，三斜线引用路径将相对于包含它的文件进行解析。

### 错误

引用不存在的文件属于错误。
文件对自身包含三斜线引用属于错误。

### 使用 `--noResolve`

如果指定了编译器标志 [`noResolve`](/tsconfig#noResolve)，则三斜线引用会被忽略；它们既不会导致添加新文件，也不会改变所提供文件的顺序。

## `/// <reference types="..." />`

与用作*依赖*声明的 `/// <reference path="..." />` 指令类似，`/// <reference types="..." />` 指令声明了对某个程序包（package）的依赖。

解析这些包名的过程与在 `import` 语句中解析模块名的过程类似。
理解三斜线类型引用指令的一个简单方法，就是将其视为用于声明包（declaration packages）的 `import`。

例如，在声明文件中包含 `/// <reference types="node" />`，即声明该文件使用了 `@types/node/index.d.ts` 中声明的名称；
因此，该包需要与该声明文件一起包含在编译中。

若要在 `.ts` 文件中声明对 `@types` 包的依赖，请改用命令行或 `tsconfig.json` 中的 [`types`](/tsconfig#types)。
有关更多详细信息，请参阅[在 `tsconfig.json` 文件中使用 `@types`、`typeRoots` 和 `types`](/docs/handbook/tsconfig-json.html#types-typeroots-and-types)。

## `/// <reference lib="..." />`

该指令允许文件显式包含现有的内置 _lib_ 文件。

内置 _lib_ 文件的引用方式与 _tsconfig.json_ 中的 [`lib`](/tsconfig#lib) 编译器选项相同（例如使用 `lib="es2015"` 而不是 `lib="lib.es2015.d.ts"` 等）。

对于依赖内置类型（例如 DOM API 或诸如 `Symbol`、`Iterable` 等内置 JS 运行时构造函数）的声明文件作者，推荐使用三斜线 lib 引用指令。以往，这些 .d.ts 文件必须对这类类型添加前置/重复声明。

例如，向编译中的某个文件添加 `/// <reference lib="es2017.string" />` 等同于使用 `--lib es2017.string` 进行编译。

```ts
/// <reference lib="es2017.string" />

'foo'.padStart(4)
```

## `/// <reference no-default-lib="true"/>`

该指令将文件标记为*默认库*（default library）。
你会在 `lib.d.ts` 及其不同变体的顶部看到此注释。

该指令指示编译器*不要*在编译中包含默认库（即 `lib.d.ts`）。
其效果类似于在命令行中传递 [`noLib`](/tsconfig#noLib)。

另请注意，当传递 [`skipDefaultLibCheck`](/tsconfig#skipDefaultLibCheck) 时，编译器将仅跳过对包含 `/// <reference no-default-lib="true"/>` 的文件的检查。

## `/// <amd-module />`

默认情况下，生成的 AMD 模块是匿名的。
当使用其他工具（例如 `r.js` 等打包工具）处理生成的模块时，这可能会引发问题。

`amd-module` 指令允许向编译器传递可选的模块名称：

##### amdModule.ts

```ts
/// <amd-module name="NamedModule"/>
export class C {}
```

这将导致在调用 AMD `define` 时，将名称 `NamedModule` 赋给该模块：

##### amdModule.js

```js
define('NamedModule', ['require', 'exports'], function (require, exports) {
  var C = (function () {
    function C() {}
    return C
  })()
  exports.C = C
})
```

## `/// <amd-dependency />`

> **Note**: this directive has been deprecated. Use `import "moduleName";` statements instead.

`/// <amd-dependency path="x" />` 用于通知编译器：存在一个非 TS 模块依赖，需要将其注入到生成模块的 require 调用中。

`amd-dependency` 指令还可以包含一个可选的 `name` 属性；这允许为 amd-dependency 传递一个可选的名称：

```ts
/// <amd-dependency path="legacy/moduleA" name="moduleA"/>
declare var moduleA: MyType
moduleA.callStuff()
```

生成的 JS 代码：

```js
define(['require', 'exports', 'legacy/moduleA'], function (
  require,
  exports,
  moduleA,
) {
  moduleA.callStuff()
})
```

## `preserve="true"`

三斜线指令可以标记 `preserve="true"`，以防止编译器在输出中将其移除。

例如，以下指令将在输出中被擦除：

```ts
/// <reference path="..." />
/// <reference types="..." />
/// <reference lib="..." />
```

但以下指令将被保留：

```ts
/// <reference path="..." preserve="true" />
/// <reference types="..." preserve="true" />
/// <reference lib="..." preserve="true" />
```
