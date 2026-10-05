---
display: 'Disable Filename Based Type Acquisition'
oneline: '禁用基于项目中文件名进行推断的类型获取。'
---

TypeScript 的类型获取功能可以根据项目中的文件名推断应该添加哪些类型定义。这意味着如果你的项目中存在像 `jquery.js` 这样的文件，它将自动从 DefinitelyTyped 下载 jQuery 的类型定义。

你可以通过 `disableFilenameBasedTypeAcquisition` 禁用此行为。

```json
{
  "typeAcquisition": {
    "disableFilenameBasedTypeAcquisition": true
  }
}
```
