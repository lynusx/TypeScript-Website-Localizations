---
display: 'Resolve JSON Module'
oneline: '允许导入 .json 文件。'
---

允许导入扩展名为 `.json` 的模块，这是 Node 项目中的常见做法。这也包括根据静态 JSON 结构为 `import` 生成对应的类型。

默认情况下，TypeScript 不支持解析 JSON 文件：

```ts twoslash
// @errors: 2732
// @filename: settings.json
{
    "repo": "TypeScript",
    "dry": false,
    "debug": false
}
// @filename: index.ts
import settings from "./settings.json";

settings.debug === true;
settings.dry === 2;
```

启用该选项后允许导入 JSON，并对该 JSON 文件中的类型进行校验。

```ts twoslash
// @errors: 2367
// @resolveJsonModule
// @module: commonjs
// @moduleResolution: node
// @filename: settings.json
{
    "repo": "TypeScript",
    "dry": false,
    "debug": false
}
// @filename: index.ts
import settings from "./settings.json";

settings.debug === true;
settings.dry === 2;
```
