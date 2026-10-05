---
display: 'Type Roots'
oneline: '指定多个行为类似于 `./node_modules/@types` 的目录。'
---

默认情况下，所有可见的（_visible_）"`@types`" 包都会包含在编译中。
任何外层目录中 `node_modules/@types` 下的包都被视为可见的（_visible_）。
例如，这意味着包含 `./node_modules/@types/`、`../node_modules/@types/`、`../../node_modules/@types/` 等路径下的包。

如果指定了 `typeRoots`，则仅（_only_）包含 `typeRoots` 目录下的包。例如：

```json tsconfig
{
  "compilerOptions": {
    "typeRoots": ["./typings", "./vendor/types"]
  }
}
```

此配置文件将包含 `./typings` 和 `./vendor/types` 下的所有（_all_）包，而不会包含 `./node_modules/@types` 中的任何包。
所有路径均相对于 `tsconfig.json`。
