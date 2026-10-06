---
title: 模块插件 .d.ts
layout: docs
permalink: /zh/docs/handbook/declaration-files/templates/module-plugin-d-ts.html
---

例如，当你需要配合扩展了另一个库的 JavaScript 代码使用时：

```ts
import { greeter } from 'super-greeter'

// 常规 Greeter API
greeter(2)
greeter('Hello world')

// 现在我们在运行时为该对象扩展一个新函数
import 'hyper-super-greeter'
greeter.hyperGreet()
```

“super-greeter”的定义如下：

```ts
/*~ 此示例展示了如何为你的函数提供多个重载 */
export interface GreeterFunction {
  (name: string): void
  (time: number): void
}

/*~ 此示例展示了如何导出由接口指定的函数 */
export const greeter: GreeterFunction
```

我们可以像下面这样扩展已有的模块：

```ts
// Type definitions for [~库名称~] [~可选版本号~]
// Project: [~项目名称~]
// Definitions by: [~你的姓名~] <[~你的 URL~]>

/*~ 这是模块插件的模板文件。你应该将其重命名为 index.d.ts
 *~ 并放置在与该模块同名的文件夹中。
 *~ 例如，如果你正在为 "super-greeter" 编写文件，
 *~ 该文件应为 'super-greeter/index.d.ts'
 */

/*~ 在此行中，导入该模块所要扩展的目标模块 */
import { greeter } from 'super-greeter'

/*~ 在此处声明与上方导入相同的模块，
 *~ 然后我们扩展 greeter 函数的现有声明
 */
export module 'super-greeter' {
  export interface GreeterFunction {
    /** 更棒的问候方式！ */
    hyperGreet(): void
  }
}
```

这使用了[声明合并](/docs/handbook/declaration-merging.html)。

## ES6 对模块插件的影响

某些插件会在现有模块上添加或修改顶层导出项。
虽然这在 CommonJS 和其他加载器中是合法的，但 ES6 模块被视为不可变的，因此这种模式将不可行。
由于 TypeScript 与模块加载器无关，所以在编译期并不会强制约束这一规则，但计划迁移到 ES6 模块加载器的开发者应当注意这一点。
