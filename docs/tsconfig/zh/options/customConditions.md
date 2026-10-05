---
display: 'Custom Conditions'
oneline: '解析导入时，在解析器特定默认条件之外额外设置的条件。'
---

`--customConditions` 接受一个附加[条件（conditions）](https://nodejs.org/api/packages.html#nested-conditions)列表，当 TypeScript 解析 `package.json` 中的 [`exports`](https://nodejs.org/api/packages.html#exports) 或 [`imports`](https://nodejs.org/api/packages.html#imports) 字段时，这些条件应该匹配成功。
这些条件会添加到解析器默认使用的任何既有条件之上。

例如，在 `tsconfig.json` 中配置该字段如下：

```jsonc
{
  "compilerOptions": {
    "target": "es2022",
    "moduleResolution": "bundler",
    "customConditions": ["my-condition"],
  },
}
```

每当引用 `package.json` 中的 `exports` 或 `imports` 字段时，TypeScript 都会考虑名为 `my-condition` 的条件。

因此，当从具有以下 `package.json` 的包中导入时：

```jsonc
{
  // ...
  "exports": {
    ".": {
      "my-condition": "./foo.mjs",
      "node": "./bar.mjs",
      "import": "./baz.mjs",
      "require": "./biz.mjs",
    },
  },
}
```

TypeScript 将尝试查找与 `foo.mjs` 对应的文件。

该字段仅在 [`--moduleResolution`](#moduleResolution) 设置为 `node16`、`nodenext` 和 `bundler` 时有效。
