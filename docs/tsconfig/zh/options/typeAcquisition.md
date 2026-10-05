---
display: 'Type Acquisition'
oneline: '指定自动获取声明文件的相关选项。'
---

当你在编辑器中打开 JavaScript 项目时，TypeScript 会使用 DefinitelyTyped 的 `@types` 定义集合自动为你的 `node_modules` 提供类型。
这被称为类型自动获取（Automatic Type Acquisition），你可以使用配置中的 `typeAcquisition` 对象对其进行自定义。

如果你想禁用或自定义此功能，可以在项目根目录下创建一个 `jsconfig.json`：

```json
{
  "typeAcquisition": {
    "enable": false
  }
}
```

如果你有某个需要包含的特定模块（但它不在 `node_modules` 中）：

```json
{
  "typeAcquisition": {
    "include": ["jest"]
  }
}
```

如果某个模块不应被自动获取（例如，该库存在于 `node_modules` 中，但团队已约定不使用它）：

```json
{
  "typeAcquisition": {
    "exclude": ["jquery"]
  }
}
```
