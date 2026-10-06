---
title: 模块插件 .d.ts
layout: docs
permalink: /zh/docs/handbook/declaration-files/templates/module-plugin-d-ts.html
---

例如，当你需要配合扩展了另一个库的 JavaScript 代码使用时：

```ts
import { greeter } from 'super-greeter'

// Normal Greeter API
greeter(2)
greeter('Hello world')

// Now we extend the object with a new function at runtime
import 'hyper-super-greeter'
greeter.hyperGreet()
```

“super-greeter”的定义如下：

```ts
/*~ This example shows how to have multiple overloads for your function */
export interface GreeterFunction {
  (name: string): void
  (time: number): void
}

/*~ This example shows how to export a function specified by an interface */
export const greeter: GreeterFunction
```

我们可以像下面这样扩展已有的模块：

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ This is the module plugin template file. You should rename it to index.d.ts
 *~ and place it in a folder with the same name as the module.
 *~ For example, if you were writing a file for "super-greeter", this
 *~ file should be 'super-greeter/index.d.ts'
 */

/*~ On this line, import the module which this module adds to */
import { greeter } from 'super-greeter'

/*~ Here, declare the same module as the one you imported above
 *~ then we expand the existing declaration of the greeter function
 */
export module 'super-greeter' {
  export interface GreeterFunction {
    /** Greets even better! */
    hyperGreet(): void
  }
}
```

这使用了[声明合并](/docs/handbook/declaration-merging.html)。

## ES6 对模块插件的影响

某些插件会在现有模块上添加或修改顶层导出项。
虽然这在 CommonJS 和其他加载器中是合法的，但 ES6 模块被视为不可变的，因此这种模式将不可行。
由于 TypeScript 与模块加载器无关，所以在编译期并不会强制约束这一规则，但计划迁移到 ES6 模块加载器的开发者应当注意这一点。
