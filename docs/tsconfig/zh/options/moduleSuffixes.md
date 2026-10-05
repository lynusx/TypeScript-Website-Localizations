---
display: 'Module Suffixes'
oneline: '解析模块时要搜索的文件名后缀列表。'
---

提供了一种覆盖解析模块时搜索的文件名后缀默认列表的方法。

```json tsconfig
{
  "compilerOptions": {
    "moduleSuffixes": [".ios", ".native", ""]
  }
}
```

根据上述配置，对于如下导入：

```ts
import * as foo from './foo'
```

TypeScript 将依次查找相对文件 `./foo.ios.ts`、`./foo.native.ts`，最后查找 `./foo.ts`。

请注意 [`moduleSuffixes`](#moduleSuffixes) 中的空字符串 `""`，若要让 TypeScript 同时也去查找 `./foo.ts`，这个空字符串是必不可少的。

该特性在 React Native 项目中非常有用，每个目标平台都可以使用具有不同 `moduleSuffixes` 的独立 tsconfig.json。
