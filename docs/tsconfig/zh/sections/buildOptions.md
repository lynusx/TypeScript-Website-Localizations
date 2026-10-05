## 构建选项

类型获取（Type Acquisition）仅对 JavaScript 项目起作用。在 TypeScript 项目中，你需要显式地在项目中引入类型。但对于 JavaScript 项目，TypeScript 工具链会在后台自动下载模块所需的类型，并保存在 node_modules 目录之外。

如果你不需要该功能，可以在项目根目录的 `jsconfig.json` 中配置如下内容以关闭类型获取：

```json
{
  "typeAcquisition": {
    "enable": false
  }
}
```

在 `jsconfig.json` 中使用该配置项的常见场景，是告知 TypeScript 下载额外的类型定义以增强开发工具体验：

```json
{
  "typeAcquisition": {
    "include": ["jquery"]
  }
}
```
