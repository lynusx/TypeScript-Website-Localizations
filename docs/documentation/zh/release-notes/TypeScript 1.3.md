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
    // OK, can access protected member from subclass
    this.doSomething()
  }
}
var t = new MyThing()
t.doSomething() // Error, cannot call protected member from outside class
```

## 元组类型（Tuple types）

元组类型用于表示已知特定位置元素类型的数组，各元素的类型不必相同。例如，你可能想要表示一个在索引 0 处为 `string`、在索引 1 处为 `number` 的数组：

```ts
// Declare a tuple type
var x: [string, number]
// Initialize it
x = ['hello', 10] // OK
// Initialize it incorrectly
x = [10, 'hello'] // Error
```

访问已知索引的元素时，可以获取到对应的正确类型：

```ts
console.log(x[0].substr(1)) // OK
console.log(x[1].substr(1)) // Error, 'number' does not have 'substr'
```

注意，在 TypeScript 1.4 中，当访问超出已知索引集合的元素时，将改用联合类型：

```ts
x[3] = 'world' // OK
console.log(x[5].toString()) // OK, 'string' and 'number' both have toString
x[6] = true // Error, boolean isn't number or string
```
