---
display: 'No Error Truncation'
oneline: '禁止在错误信息中截断类型。'
---

不截断错误信息。

设置为 `false`（默认值）：

```ts twoslash
// @errors: 2322 2454
var x: {
  propertyWithAnExceedinglyLongName1: string
  propertyWithAnExceedinglyLongName2: string
  propertyWithAnExceedinglyLongName3: string
  propertyWithAnExceedinglyLongName4: string
  propertyWithAnExceedinglyLongName5: string
  propertyWithAnExceedinglyLongName6: string
  propertyWithAnExceedinglyLongName7: string
  propertyWithAnExceedinglyLongName8: string
}

// String representation of type of 'x' should be truncated in error message
var s: string = x
```

设置为 `true`：

```ts twoslash
// @errors: 2322 2454
// @noErrorTruncation: true
var x: {
  propertyWithAnExceedinglyLongName1: string
  propertyWithAnExceedinglyLongName2: string
  propertyWithAnExceedinglyLongName3: string
  propertyWithAnExceedinglyLongName4: string
  propertyWithAnExceedinglyLongName5: string
  propertyWithAnExceedinglyLongName6: string
  propertyWithAnExceedinglyLongName7: string
  propertyWithAnExceedinglyLongName8: string
}

// String representation of type of 'x' should be truncated in error message
var s: string = x
```
