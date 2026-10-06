---
title: 模块类 .d.ts
layout: docs
permalink: /zh/docs/handbook/declaration-files/templates/module-class-d-ts.html
---

<!--
TODO:

1. Not clear why UMD is thrown in here.
2. Give both commonjs and ES module examples.
-->

例如，当你需要配合如下形式的 JavaScript 代码使用时：

```ts
const Greeter = require('super-greeter')

const greeter = new Greeter()
greeter.greet()
```

为了同时支持通过 UMD 和模块导入：

```ts
// Type definitions for [~库名称~] [~可选版本号~]
// Project: [~项目名称~]
// Definitions by: [~你的姓名~] <[~你的 URL~]>

/*~ 这是类模块的模块模板文件。
 *~ 你应该将其重命名为 index.d.ts 并放置在与该模块同名的文件夹中。
 *~ 例如，如果你正在为 "super-greeter" 编写文件，
 *~ 该文件应为 'super-greeter/index.d.ts'
 */

// 请注意，ES6 模块不能直接导出类对象。
// 应使用 CommonJS 风格导入此文件：
//   import x = require('[~THE MODULE~]');
//
// 或者，如果开启了 --allowSyntheticDefaultImports 或
// --esModuleInterop，也可以将其作为默认导入引入：
//   import x from '[~THE MODULE~]';
//
// 请参阅 TypeScript 文档：
// https://www.typescriptlang.org/docs/handbook/modules.html#export--and-import--require
// 以了解针对 ES6 模块这一限制的常见解决办法。

/*~ 如果该模块是一个 UMD 模块，在模块加载器环境之外加载时
 *~ 暴露了一个全局变量 'myClassLib'，请在此声明该全局变量。
 *~ 否则，请删除此声明。
 */
export as namespace myClassLib

/*~ 此声明指定了类构造函数是
 *~ 该文件导出的对象
 */
export = Greeter

/*~ 在此类中编写模块的方法和属性 */
declare class Greeter {
  constructor(customGreeting?: string)

  greet: void

  myMethod(opts: MyClass.MyClassMethodOptions): number
}

/*~ 如果你还希望从模块中暴露类型，可以将它们
 *~ 放在此块中。
 *~
 *~ 请注意，如果你决定包含此命名空间，除非开启了
 *~ --esModuleInterop，否则该模块可能会被错误地作为命名空间对象导入：
 *~   import * as x from '[~THE MODULE~]'; // 错误！切勿这样做！
 */
declare namespace MyClass {
  export interface MyClassMethodOptions {
    width?: number
    height?: number
  }
}
```
