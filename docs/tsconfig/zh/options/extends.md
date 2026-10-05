---
display: 'Extends'
oneline: '指定一个或多个路径或 Node 模块引用，以继承基础配置文件中的设置。'
---

`extends` 的值是一个字符串，其中包含要继承的另一个配置文件的路径。该路径可以使用 Node.js 风格的解析规则。

基础配置文件中的配置会首先加载，随后被继承配置文件中的配置所覆盖。配置文件中的所有相对路径都将相对于其源自的配置文件进行解析。

值得注意的是，继承配置文件中的 [`files`](#files)、[`include`](#include) 和 [`exclude`](#exclude) 会直接*覆盖*基础配置文件中的对应配置，并且配置文件之间不允许出现循环继承。

目前，唯一不会被继承的顶级属性是 [`references`](#references)。

#### 示例

`configs/base.json`：

```json tsconfig
{
  "compilerOptions": {
    "noImplicitAny": true,
    "strictNullChecks": true
  }
}
```

`tsconfig.json`：

```json tsconfig
{
  "extends": "./configs/base",
  "files": ["main.ts", "supplemental.ts"]
}
```

`tsconfig.nostrictnull.json`：

```json tsconfig
{
  "extends": "./tsconfig",
  "compilerOptions": {
    "strictNullChecks": false
  }
}
```

配置文件中包含相对路径且未被排除在继承之外的属性，都将相对于其源自的配置文件进行解析。
