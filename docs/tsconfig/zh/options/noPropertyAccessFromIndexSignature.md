---
display: 'No Property Access From Index Signature'
oneline: '强制要求对使用索引类型声明的键使用索引访问器。'
---

此设置可确保通过“点”语法（`obj.key`）和“索引”语法（`obj["key"]`）访问字段的方式，与该属性在类型中的声明方式保持一致。

如果不启用此标志，TypeScript 允许使用点语法访问未显式定义的字段：

```ts twoslash
// @errors: 4111
declare function getSettings(): GameSettings
// ---cut---
interface GameSettings {
  // Known up-front properties
  speed: 'fast' | 'medium' | 'slow'
  quality: 'high' | 'low'

  // Assume anything unknown to the interface
  // is a string.
  [key: string]: string
}

const settings = getSettings()
settings.speed
//       ^?
settings.quality
//       ^?

// Unknown key accessors are allowed on
// this object, and are `string`
settings.username
//       ^?
```

启用该标志后，未知字段使用点语法而非索引语法将会报错：

```ts twoslash
// @errors: 4111
// @noPropertyAccessFromIndexSignature
declare function getSettings(): GameSettings
interface GameSettings {
  speed: 'fast' | 'medium' | 'slow'
  quality: 'high' | 'low'
  [key: string]: string
}
// ---cut---
const settings = getSettings()
settings.speed
settings.quality

// This would need to be settings["username"];
settings.username
//       ^?
```

此标志的目的是通过调用语法明确表达意图，体现你对该属性是否确实存在的确信程度。
