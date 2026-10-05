---
display: 'Strict Function Types'
oneline: '在对函数赋值时，检查确保参数和返回值满足子类型兼容性。'
---

启用该标志后，会对函数参数进行更严谨的检查。

以下是关闭 `strictFunctionTypes` 时的基础示例：

```ts twoslash
// @strictFunctionTypes: false
function fn(x: string) {
  console.log('Hello, ' + x.toLowerCase())
}

type StringOrNumberFunc = (ns: string | number) => void

// Unsafe assignment
let func: StringOrNumberFunc = fn
// Unsafe call - will crash
func(10)
```

开启 `strictFunctionTypes` 后，该错误会被正确检测出来：

```ts twoslash
// @errors: 2322
function fn(x: string) {
  console.log('Hello, ' + x.toLowerCase())
}

type StringOrNumberFunc = (ns: string | number) => void

// Unsafe assignment is prevented
let func: StringOrNumberFunc = fn
```

在开发该功能的过程中，我们发现了大量本质上不安全的类层次结构，包括 DOM 中的一些类。
因此，该设置仅适用于以“函数”语法编写的函数，而不适用于“方法”语法编写的成员：

```ts twoslash
type Methodish = {
  func(x: string | number): void
}

function fn(x: string) {
  console.log('Hello, ' + x.toLowerCase())
}

// Ultimately an unsafe assignment, but not detected
const m: Methodish = {
  func: fn,
}
m.func(10)
```
