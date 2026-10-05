---
display: 'Paths'
oneline: '指定一组将导入重映射到其他查找位置的条目。'
---

一系列将导入重映射到查找位置的条目。若设置了 [`baseUrl`](#baseUrl)，则相对于 `baseUrl` 解析；否则相对于 tsconfig 文件本身解析。在 [`moduleResolution` 参考页面](/docs/handbook/modules/reference.html#paths) 中有关于 `paths` 更详尽的说明。

`paths` 允许你声明 TypeScript 在处理 `require`/`import` 时应如何解析导入。

```json tsconfig
{
  "compilerOptions": {
    "paths": {
      "jquery": ["./vendor/jquery/dist/jquery"]
    }
  }
}
```

这将允许你编写 `import "jquery"`，并在本地获取所有正确的类型定义。

```json tsconfig
{
  "compilerOptions": {
    "paths": {
      "app/*": ["./src/app/*"],
      "config/*": ["./src/app/_config/*"],
      "environment/*": ["./src/environments/*"],
      "shared/*": ["./src/app/_shared/*"],
      "helpers/*": ["./src/helpers/*"],
      "tests/*": ["./src/tests/*"]
    }
  }
}
```

在这种情况下，你可以告知 TypeScript 文件解析器支持多个自定义前缀以查找代码。

注意，此特性不会改变 `tsc` 生成的导入路径，因此 `paths` 仅应用于告知 TypeScript 已经有其他工具配置了此映射并将在运行时或打包时使用它。
