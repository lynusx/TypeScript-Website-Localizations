---
display: 'Base URL'
oneline: '指定用于解析裸模块说明符的基础目录。'
---

设置一个基础目录，用于解析裸模块说明符（bare specifier）的模块名。例如，在以下目录结构中：

```
project
├── ex.ts
├── hello
│   └── world.ts
└── tsconfig.json
```

当设置 `"baseUrl": "./"` 时，TypeScript 将从 `tsconfig.json` 所在的目录开始查找文件：

```ts
import { helloWorld } from 'hello/world'

console.log(helloWorld)
```

这种解析方式的优先级高于从 `node_modules` 中查找。

该功能最初是为浏览器中配合 AMD 模块加载器使用而设计的，不建议在其他场景中使用。从 TypeScript 4.1 开始，使用 [`paths`](#paths) 时不再要求设置 `baseUrl`。
