---
title: TypeScript 1.3
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-1-3.html
oneline: TypeScript 1.3 发布说明
---

## protected 修饰符

类中新增的 `protected` 修饰符的工作机制与 C++、C# 和 Java 等常见语言类似。类的 `protected` 成员仅在声明它的类的子类内部可见：

```ts
class Thing {
  protected doSomething() {
    /* ... */
  }
}

class MyThing extends Thing {
  public myMethod() {
    // 正常，可以从子类中访问受保护成员
    this.doSomething()
  }
}
var t = new MyThing()
t.doSomething() // 错误，无法从类外部调用受保护成员
```

## 元组类型（Tuple types）

元组类型用于表示已知特定位置元素类型的数组，各元素的类型不必相同。例如，你可能想要表示一个在索引 0 处为 `string`、在索引 1 处为 `number` 的数组：

```ts
// 声明一个元组类型
var x: [string, number]
// 初始化元组
x = ['hello', 10] // 正常
// 错误的初始化
x = [10, 'hello'] // 错误
```

访问已知索引的元素时，可以获取到对应的正确类型：

```ts
console.log(x[0].substr(1)) // 正常
console.log(x[1].substr(1)) // 错误，'number' 没有 'substr'
```

注意，在 TypeScript 1.4 中，当访问超出已知索引集合的元素时，将改用联合类型：

```ts
x[3] = 'world' // 正常
console.log(x[5].toString()) // 正常，'string' 和 'number' 都具有 toString
x[6] = true // 错误，boolean 既不是 number 也不是 string
```
