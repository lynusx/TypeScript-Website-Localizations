---
display: 'Strip Internal'
oneline: '禁止为 JSDoc 注释中带有 `@internal` 的代码生成声明。'
---

禁止为 JSDoc 注释中带有 `@internal` 标记的代码生成声明。
这是一个编译器内部选项；使用时需自行承担风险，因为编译器不会校验生成结果的有效性。
如果你需要一个工具来在 `d.ts` 文件中处理更多层级的可见性控制，可以参考 [api-extractor](https://api-extractor.com)。

```ts twoslash
/**
 * 一周中可用的天数
 * @internal
 */
export const daysInAWeek = 7

/** 计算某人在一周内的收入 */
export function weeklySalary(dayRate: number) {
  return daysInAWeek * dayRate
}
```

当该标志设置为 `false`（默认值）时：

```ts twoslash
// @showEmittedFile: index.d.ts
// @showEmit
// @declaration
/**
 * 一周中可用的天数
 * @internal
 */
export const daysInAWeek = 7

/** 计算某人在一周内的收入 */
export function weeklySalary(dayRate: number) {
  return daysInAWeek * dayRate
}
```

当 `stripInternal` 设置为 `true` 时，生成的 `d.ts` 将隐去相应内容：

```ts twoslash
// @stripinternal
// @showEmittedFile: index.d.ts
// @showEmit
// @declaration
/**
 * 一周中可用的天数
 * @internal
 */
export const daysInAWeek = 7

/** 计算某人在一周内的收入 */
export function weeklySalary(dayRate: number) {
  return daysInAWeek * dayRate
}
```

生成的 JavaScript 输出依然保持不变。
