---
display: 'Allow Synthetic Default Imports'
oneline: "当模块没有默认导出时，允许使用 'import x from y'。"
---

当设置为 true 时，若模块**没有**明确指定默认导出，`allowSyntheticDefaultImports` 允许你编写如下形式的导入：

```ts
import React from 'react'
```

而不是：

```ts
import * as React from 'react'
```

例如，当未将 `allowSyntheticDefaultImports` 设置为 true 时：

```ts twoslash
// @errors: 1259 1192
// @checkJs
// @allowJs
// @esModuleInterop: false
// @filename: utilFunctions.js
// @noImplicitAny: false
const getStringLength = (str) => str.length

module.exports = {
  getStringLength,
}

// @filename: index.ts
import utils from './utilFunctions'

const count = utils.getStringLength('Check JS')
```

这段代码会报错，因为不存在可供导入的 `default` 对象——尽管直觉上觉得应该有。
为了方便，像 Babel 这样的转译工具会在未创建默认导出时自动创建一个，使模块看起来更像这样：

```js
// @filename: utilFunctions.js
const getStringLength = (str) => str.length
const allFunctions = {
  getStringLength,
}

module.exports = allFunctions
module.exports.default = allFunctions
```

该标志不会影响 TypeScript 输出的 JavaScript，仅用于类型检查。
该选项使 TypeScript 的行为与 Babel 保持一致，后者会生成额外代码，使模块的默认导出使用起来更符合直觉与开发习惯。
