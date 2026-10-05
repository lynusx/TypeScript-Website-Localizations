---
display: 'Exclude'
oneline: '指定从类型获取中排除的模块列表。'
---

提供了一项配置，用于在 JavaScript 项目中禁用特定模块的类型获取（type-acquisition）。对于在测试基础设施中引入了主应用程序中不需要的其他库的项目，这非常有用。

```json
{
  "typeAcquisition": {
    "exclude": ["jest", "mocha"]
  }
}
```
