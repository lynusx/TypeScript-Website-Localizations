---
title: 规范与示例
layout: docs
permalink: /zh/docs/handbook/declaration-files/by-example.html
oneline: '如何为模块创建 d.ts 文件'
---

本指南旨在指导你如何编写高质量的声明文件。
本指南的结构是先展示某 API 的文档说明，再给出该 API 的使用示例，
并解释如何编写对应的声明。

这些示例大致按照复杂度递增的顺序排列。

## 包含属性的对象

_文档_

> 全局变量 `myLib` 包含一个用于创建问候语的函数 `makeGreeting`，
> 以及一个表示迄今已创建问候语数量的属性 `numberOfGreetings`。

_代码_

```ts
let result = myLib.makeGreeting('hello, world')
console.log('The computed greeting is:' + result)

let count = myLib.numberOfGreetings
```

_声明_

使用 `declare namespace` 描述通过点号表示法访问的类型或值。

```ts
declare namespace myLib {
  function makeGreeting(s: string): string
  let numberOfGreetings: number
}
```

## 函数重载

_文档_

`getWidget` 函数接收一个数字并返回一个 Widget，或者接收一个字符串并返回一个 Widget 数组。

_代码_

```ts
let x: Widget = getWidget(43)

let arr: Widget[] = getWidget('all of them')
```

_声明_

```ts
declare function getWidget(n: number): Widget
declare function getWidget(s: string): Widget[]
```

## 可复用类型（接口）

_文档_

> 指定问候语时，必须传入一个 `GreetingSettings` 对象。
> 该对象具有以下属性：
>
> 1 - greeting：必填的字符串
>
> 2 - duration：可选的时长（单位为毫秒）
>
> 3 - color：可选的字符串，例如 '#ff00ff'

_代码_

```ts
greet({
  greeting: 'hello world',
  duration: 4000,
})
```

_声明_

使用 `interface` 定义带有属性的类型。

```ts
interface GreetingSettings {
  greeting: string
  duration?: number
  color?: string
}

declare function greet(setting: GreetingSettings): void
```

## 可复用类型（类型别名）

_文档_

> 任何需要问候语的地方，都可以提供一个 `string`、一个返回 `string` 的函数或一个 `Greeter` 实例。

_代码_

```ts
function getGreeting() {
  return 'howdy'
}
class MyGreeter extends Greeter {}

greet('hello')
greet(getGreeting)
greet(new MyGreeter())
```

_声明_

你可以使用类型别名为类型定义简写形式：

```ts
type GreetingLike = string | (() => string) | MyGreeter

declare function greet(g: GreetingLike): void
```

## 组织类型

_文档_

> `greeter` 对象可以将日志记录到文件或显示弹窗。
> 你可以向 `.log(...)` 传递 LogOptions，向 `.alert(...)` 传递弹窗选项。

_代码_

```ts
const g = new Greeter('Hello')
g.log({ verbose: true })
g.alert({ modal: false, title: 'Current Greeting' })
```

_声明_

使用命名空间来组织类型。

```ts
declare namespace GreetingLib {
  interface LogOptions {
    verbose?: boolean
  }
  interface AlertOptions {
    modal: boolean
    title?: string
    color?: string
  }
}
```

你也可以在单个声明中创建嵌套命名空间：

```ts
declare namespace GreetingLib.Options {
  // 通过 GreetingLib.Options.Log 引用
  interface Log {
    verbose?: boolean
  }
  interface Alert {
    modal: boolean
    title?: string
    color?: string
  }
}
```

## 类

_文档_

> 你可以通过实例化 `Greeter` 对象来创建问候器，也可以通过继承它来创建自定义的问候器。

_代码_

```ts
const myGreeter = new Greeter('hello, world')
myGreeter.greeting = 'howdy'
myGreeter.showGreeting()

class SpecialGreeter extends Greeter {
  constructor() {
    super('Very special greetings')
  }
}
```

_声明_

使用 `declare class` 描述类或类似类的对象。
类可以拥有属性和方法，也可以拥有构造函数。

```ts
declare class Greeter {
  constructor(greeting: string)

  greeting: string
  showGreeting(): void
}
```

## 全局变量

_文档_

> 全局变量 `foo` 包含了现有小部件（widget）的数量。

_代码_

```ts
console.log('Half the number of widgets is ' + foo / 2)
```

_声明_

使用 `declare var` 声明变量。
如果变量是只读的，可以使用 `declare const`。
如果变量具有块级作用域，也可以使用 `declare let`。

```ts
/** 当前小部件（widget）的数量 */
declare var foo: number
```

## 全局函数

_文档_

> 你可以传入一个字符串来调用函数 `greet`，从而向用户显示问候语。

_代码_

```ts
greet('hello, world')
```

_声明_

使用 `declare function` 声明函数。

```ts
declare function greet(greeting: string): void
```
