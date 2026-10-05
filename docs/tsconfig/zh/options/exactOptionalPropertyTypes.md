---
display: 'Exact Optional Property Types'
oneline: '按字面定义解释可选属性类型，而不是自动添加 `undefined`。'
---

启用 `exactOptionalPropertyTypes` 后，TypeScript 会对带有 `?` 前缀修饰的 `type` 或 `interface` 属性应用更严格的类型处理规则。

例如，以下接口声明了一个属性，其值可以是两个字符串之一（'dark' 或 'light'），或者该属性不应存在于对象中。

```ts
interface UserDefaults {
  // The absence of a value represents 'system'
  colorThemeOverride?: 'dark' | 'light'
}
```

如果未启用此标志，你可以为 `colorThemeOverride` 赋予三种值："dark"、"light" 和 `undefined`。

将值设置为 `undefined` 可以让大多数用于检查属性是否存在的 JavaScript 运行时判断失败（实际上表现为假值）。然而，这并不完全准确；`colorThemeOverride: undefined` 与根本未定义 `colorThemeOverride` 并不相同。例如，当键的值为 `undefined` 时，`"colorThemeOverride" in settings` 的行为与该键完全未定义时的行为存在差异。

`exactOptionalPropertyTypes` 使得 TypeScript 真正严格遵循可选属性的原始定义：

```ts twoslash
// @exactOptionalPropertyTypes
// @errors: 2322 2412
interface UserDefaults {
  colorThemeOverride?: 'dark' | 'light'
}
declare function getUserSettings(): UserDefaults
// ---cut---
const settings = getUserSettings()
settings.colorThemeOverride = 'dark'
settings.colorThemeOverride = 'light'

// But not:
settings.colorThemeOverride = undefined
```
