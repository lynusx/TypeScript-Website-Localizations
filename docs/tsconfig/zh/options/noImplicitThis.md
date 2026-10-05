---
display: 'No Implicit This'
oneline: '当 `this` 表达式具有隐含的 `any` 类型时启用错误报告。'
---

当 `this` 表达式具有隐含的 `any` 类型时引发错误。

例如，下面的类中返回了一个尝试访问 `this.width` 和 `this.height` 的函数——但是 `getAreaFunction` 内部该函数中的 `this` 上下文并不是 Rectangle 的实例。

```ts twoslash
// @errors: 2683
class Rectangle {
  width: number
  height: number

  constructor(width: number, height: number) {
    this.width = width
    this.height = height
  }

  getAreaFunction() {
    return function () {
      return this.width * this.height
    }
  }
}
```
