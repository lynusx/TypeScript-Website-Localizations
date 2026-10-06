---
title: '全局修改模块 .d.ts'
layout: docs
permalink: /zh/docs/handbook/declaration-files/templates/global-modifying-module-d-ts.html
---

## _全局修改模块_

_全局修改模块_在被导入时会修改全局作用域中的现有值。
例如，可能存在一个库在导入时向 `String.prototype` 添加新成员。
由于可能引发运行时冲突，这种模式存在一定风险，
但我们仍然可以为其编写声明文件。

## 识别全局修改模块

全局修改模块通常很容易从其文档中识别出来。
通常来说，它们类似于全局插件，但需要通过 `require` 调用来使其生效。

你可能会看到类似这样的文档：

```js
// 不使用其返回值的 'require' 调用
var unused = require('magic-string-time')
/* 或者 */
require('magic-string-time')

var x = 'hello, world'
// 在内置类型上创建新方法
console.log(x.startsWithHello())

var y = [1, 2, 3]
// 在内置类型上创建新方法
console.log(y.reverseAndSort())
```

下面是一个示例：

```ts
// Type definitions for [~库名称~] [~可选版本号~]
// Project: [~项目名称~]
// Definitions by: [~你的姓名~] <[~你的 URL~]>

/*~ 这是全局修改模块的模板文件。你应该将其重命名为 index.d.ts
 *~ 并放置在与该模块同名的文件夹中。
 *~ 例如，如果你正在为 "super-greeter" 编写文件，
 *~ 该文件应为 'super-greeter/index.d.ts'
 */

/*~ 注意：如果你的全局修改模块是可调用的或可构造的，
 *~ 你需要将此处的模式与 module-class 或 module-function
 *~ 模板文件中的模式结合起来
 */
declare global {
  /*~ 在这里声明属于全局命名空间的内容，或对
   *~ 全局命名空间中的现有声明进行扩充
   */
  interface String {
    fancyFormat(opts: StringFormatOptions): string
  }
}

/*~ 如果你的模块导出了类型或值，请按常规方式编写它们 */
export interface StringFormatOptions {
  fancinessLevel: number
}

/*~ 例如，在模块上声明一个方法（除了其全局副作用之外） */
export function doSomething(): void

/*~ 如果你的模块没有任何导出，则需要此行。否则，请删除它 */
export {}
```
