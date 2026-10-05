---
display: 'Map Root'
oneline: '指定调试器查找 source map 文件的位置，而不是使用生成时的相对位置。'
---

指定调试器应该在哪个位置查找 map 文件，而不是使用生成的位置。
此字符串会原封不动地写入 source map 中，例如：

```json tsconfig
{
  "compilerOptions": {
    "sourceMap": true,
    "mapRoot": "https://my-website.com/debug/sourcemaps/"
  }
}
```

将声明 `index.js` 的 source map 位于 `https://my-website.com/debug/sourcemaps/index.js.map`。
