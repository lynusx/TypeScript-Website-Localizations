---
display: 'Include'
oneline: '指定需要获取类型定义的模块列表。'
---

如果你有一个 JavaScript 项目，其中 TypeScript 需要额外指引来理解全局依赖，或者你已经通过 [`disableFilenameBasedTypeAcquisition`](#disableFilenameBasedTypeAcquisition) 禁用了内置的类型推断。

你可以使用 `include` 来指定应当从 DefinitelyTyped 获取并使用哪些类型：

```json
{
  "typeAcquisition": {
    "include": ["jquery"]
  }
}
```
