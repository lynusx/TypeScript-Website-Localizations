---
title: TypeScript 1.7
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-1-7.html
oneline: TypeScript 1.7 发布说明
---

## ES6 目标下的 `async`/`await` 支持（Node v4+）

TypeScript 现已为原生支持 ES6 生成器（Generator）的引擎（例如 Node v4 及以上版本）提供异步函数支持。
异步函数以 `async` 关键字作为前缀；
`await` 会暂停函数的执行，直到异步函数返回的 Promise 被兑现（fulfilled），并从返回的 `Promise` 中解包出对应的值。

##### 示例

在以下示例中，每个输入的元素都将依次打印，每次间隔 400 毫秒：

```ts
'use strict'

// printDelayed is a 'Promise<void>'
async function printDelayed(elements: string[]) {
  for (const element of elements) {
    await delay(400)
    console.log(element)
  }
}

async function delay(milliseconds: number) {
  return new Promise<void>((resolve) => {
    setTimeout(resolve, milliseconds)
  })
}

printDelayed(['Hello', 'beautiful', 'asynchronous', 'world']).then(() => {
  console.log()
  console.log('Printed every element!')
})
```

更多信息请参阅 [async 函数参考](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/async_function)。

## 支持 `--target ES6` 配合 `--module`

TypeScript 1.7 在 [`module`](/tsconfig#module) 选项的可用候选值中加入了 `ES6`，并允许你在指定编译目标为 `ES6` 时指定模块输出格式。
这为你针对特定运行时精确指定所需特性提供了更高的灵活性。

##### 示例

```json tsconfig
{
  "compilerOptions": {
    "module": "amd",
    "target": "es6"
  }
}
```

## 多态 `this` 类型

从方法中返回当前对象（即 `this`）以创建[链式调用风格的 API（fluent-style APIs）](https://wikipedia.org/wiki/Fluent_interface)是一种常见的模式。
例如，考虑以下 `BasicCalculator` 模块：

```ts
export default class BasicCalculator {
  public constructor(protected value: number = 0) {}

  public currentValue(): number {
    return this.value
  }

  public add(operand: number) {
    this.value += operand
    return this
  }

  public subtract(operand: number) {
    this.value -= operand
    return this
  }

  public multiply(operand: number) {
    this.value *= operand
    return this
  }

  public divide(operand: number) {
    this.value /= operand
    return this
  }
}
```

用户可以将 `2 * 5 + 1` 表达为：

```ts
import calc from './BasicCalculator'

let v = new calc(2).multiply(5).add(1).currentValue()
```

这往往能带来非常优雅的代码编写方式；然而，对于希望继承 `BasicCalculator` 的类来说，以前存在一个问题。
设想用户想要编写一个 `ScientificCalculator`：

```ts
import BasicCalculator from './BasicCalculator'

export default class ScientificCalculator extends BasicCalculator {
  public constructor(value = 0) {
    super(value)
  }

  public square() {
    this.value = this.value ** 2
    return this
  }

  public sin() {
    this.value = Math.sin(this.value)
    return this
  }
}
```

因为以前 TypeScript 会将 `BasicCalculator` 中每个返回 `this` 的方法的类型推断为 `BasicCalculator`，所以每当调用 `BasicCalculator` 的方法时，类型系统就会忽略当前对象实际上是 `ScientificCalculator`。

例如：

```ts
import calc from './ScientificCalculator'

let v = new calc(0.5)
  .square()
  .divide(2)
  .sin() // Error: 'BasicCalculator' has no 'sin' method.
  .currentValue()
```

现在情况有所改变——在类的实例方法内部，TypeScript 现在会将 `this` 推断为一个名为 `this` 的特殊类型。
`this` 类型就是这样书写的，其基本含义是“方法调用中点号左侧的类型”。

`this` 类型还可以与交叉类型结合使用，用于描述使用 Mixin 模式来实现继承的库（例如 Ember.js）：

```ts
interface MyType {
  extend<T>(other: T): this & T
}
```

## ES7 幂运算符

TypeScript 1.7 支持即将到来的 [ES7/ES2016 幂运算符](https://github.com/rwaldron/exponentiation-operator)：`**` 和 `**=`。
在输出为 ES3/ES5 时，这些运算符将被转换为使用 `Math.pow`。

##### 示例

```ts
var x = 2 ** 3
var y = 10
y **= 2
var z = -(4 ** 3)
```

将生成以下 JavaScript 输出：

```js
var x = Math.pow(2, 3)
var y = 10
y = Math.pow(y, 2)
var z = -Math.pow(4, 3)
```

## 改进对象字面量解构的检查

TypeScript 1.7 使得带有对象字面量或数组字面量初始值设定项的解构模式的检查更加灵活且符合直觉。

当对象字面量由对象绑定模式的隐含类型进行上下文类型化（contextually typed）时：

- 对象绑定模式中带有默认值的属性在对象字面量中变为可选属性。
- 对象绑定模式中在对象字面量中没有匹配项的属性，必须在对象绑定模式中具有默认值，并且会自动添加到对象字面量类型中。
- 对象字面量中在对象绑定模式中没有匹配项的属性将被报错。

当数组字面量由数组绑定模式的隐含类型进行上下文类型化时：

- 数组绑定模式中在数组字面量中没有匹配项的元素，必须在数组绑定模式中具有默认值，并且会自动添加到数组字面量类型中。

##### 示例

```ts
// Type of f1 is (arg?: { x?: number, y?: number }) => void
function f1({ x = 0, y = 0 } = {}) {}

// And can be called as:
f1()
f1({})
f1({ x: 1 })
f1({ y: 1 })
f1({ x: 1, y: 1 })

// Type of f2 is (arg?: (x: number, y?: number) => void
function f2({ x, y = 0 } = { x: 0 }) {}

f2()
f2({}) // Error, x not optional
f2({ x: 1 })
f2({ y: 1 }) // Error, x not optional
f2({ x: 1, y: 1 })
```

## 编译目标为 ES3 时的装饰器支持

当编译目标为 ES3 时，现在允许使用装饰器。
TypeScript 1.7 从 `__decorate` 帮助函数中移除了对 ES5 专有的 `reduceRight` 的使用。
这些改动还以向后兼容的方式内联了对 `Object.getOwnPropertyDescriptor` 和 `Object.defineProperty` 的调用，通过移除对上述 `Object` 方法的重复调用，精简了针对 ES5 及更高版本的生成代码。
