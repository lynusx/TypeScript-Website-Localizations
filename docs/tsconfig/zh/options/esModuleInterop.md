---
display: 'ES Module Interop'
oneline: '生成额外的 JavaScript 代码以更轻松地支持导入 CommonJS 模块。这同时会启用 [`allowSyntheticDefaultImports`](#allowSyntheticDefaultImports) 以实现类型兼容。'
---

默认情况下（当 `esModuleInterop` 为 false 或未设置时），TypeScript 将 CommonJS/AMD/UMD 模块视作与 ES6 模块类似。在此过程中，有两个特别的假设后来被证明是有缺陷的：

- 类似于 `import * as moment from "moment"` 的命名空间导入，其行为等同于 `const moment = require("moment")`

- 类似于 `import moment from "moment"` 的默认导入，其行为等同于 `const moment = require("moment").default`

这种不匹配导致了以下两个问题：

- ES6 模块规范规定命名空间导入（`import * as x`）只能是一个对象。TypeScript 将其视为等同于 `= require("x")`，使得该导入可以被当作函数直接调用。根据规范，这是不合法的。

- 尽管符合 ES6 模块规范，但大多数基于 CommonJS/AMD/UMD 模块的库并没有像 TypeScript 的实现那样严格遵循该规范。

开启 `esModuleInterop` 将在 TypeScript 转译后的代码中修复这两个问题。第一点改变了编译器的行为，第二点则通过两个新的辅助函数进行修复，这些辅助函数提供了垫片（shim）以确保输出的 JavaScript 具备兼容性：

```ts
import * as fs from 'fs'
import _ from 'lodash'

fs.readFileSync('file.txt', 'utf8')
_.chunk(['a', 'b', 'c', 'd'], 2)
```

禁用 `esModuleInterop` 时：

```ts twoslash
// @noErrors
// @showEmit
// @esModuleInterop: false
// @module: commonjs
import * as fs from 'fs'
import _ from 'lodash'

fs.readFileSync('file.txt', 'utf8')
_.chunk(['a', 'b', 'c', 'd'], 2)
```

将 `esModuleInterop` 设置为 `true` 时：

```ts twoslash
// @noErrors
// @showEmit
// @esModuleInterop
// @module: commonjs
import * as fs from 'fs'
import _ from 'lodash'

fs.readFileSync('file.txt', 'utf8')
_.chunk(['a', 'b', 'c', 'd'], 2)
```

_注意_：命名空间导入 `import * as fs from "fs"` 仅包含被导入对象[自身拥有的属性](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty)（基本上是在该对象自身上设置的属性，而非通过原型链继承的属性）。如果你导入的模块使用继承属性来定义其 API，则需要使用默认导入形式（`import fs from "fs"`），或者禁用 `esModuleInterop`。

_注意_：你可以通过启用 [`importHelpers`](#importHelpers) 让生成的 JS 代码更加精简：

```ts twoslash
// @noErrors
// @showEmit
// @esModuleInterop
// @importHelpers
// @module: commonjs
import * as fs from 'fs'
import _ from 'lodash'

fs.readFileSync('file.txt', 'utf8')
_.chunk(['a', 'b', 'c', 'd'], 2)
```

启用 `esModuleInterop` 还会同时启用 [`allowSyntheticDefaultImports`](#allowSyntheticDefaultImports).
