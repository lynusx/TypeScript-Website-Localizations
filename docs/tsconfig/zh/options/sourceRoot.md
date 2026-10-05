---
display: 'Source Root'
oneline: '指定调试器查找引用源文件的根路径。'
---

指定调试器查找 TypeScript 文件的位置，而不是使用相对源码位置。该字符串会被原封不动地写入 source map 中，你可以使用文件路径或 URL：

```json tsconfig
{
  "compilerOptions": {
    "sourceMap": true,
    "sourceRoot": "https://my-website.com/debug/source/"
  }
}
```

上述配置将声明 `index.js` 对应的源文件位于 `https://my-website.com/debug/source/index.ts`。
