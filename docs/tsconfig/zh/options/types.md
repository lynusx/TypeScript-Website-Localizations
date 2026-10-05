---
display: 'Types'
oneline: '指定包含的类型声明包名称，这些包无需在源文件中被显式引用。'
---

默认情况下，`types` 设置为 `[]`。在 TypeScript 6.0 之前的版本中，默认会将所有*可见的* "`@types`" 包包含在编译中。任何外层文件夹中 `node_modules/@types` 下的包都被视为*可见的*。例如，这意味着包括 `./node_modules/@types/`、`../node_modules/@types/`、`../../node_modules/@types/` 等路径下的包。

如果指定了 `types`，则只有列出的包才会被包含在全局作用域中。例如：

```json tsconfig
{
  "compilerOptions": {
    "types": ["node", "jest", "express"]
  }
}
```

此 `tsconfig.json` 文件将*仅*包含 `./node_modules/@types/node`、`./node_modules/@types/jest` 和 `./node_modules/@types/express`。`node_modules/@types/*` 下的其他包将不会被包含。

### 这会影响什么？

该选项不会影响应用程序代码中 `@types/*` 的导入方式。例如，对于上述 `compilerOptions` 示例，如果包含如下代码：

```ts
import * as moment from 'moment'

moment().format('MMMM Do YYYY, h:mm:ss a')
```

`moment` 的导入仍然具有完整的类型支持。

设置此选项后，如果某个模块未包含在 `types` 数组中：

- 不会向你的项目中添加全局变量（例如 Node 中的 `process` 或 Jest 中的 `expect`）
- 其导出内容不会出现在自动导入建议中

此特性与 [`typeRoots`](#typeRoots) 的不同之处在于：它是精确指定你希望包含的具体类型包，而 [`typeRoots`](#typeRoots) 则是指定要包含的特定文件夹。
