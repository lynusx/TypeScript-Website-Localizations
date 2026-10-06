---
display: 'noUncheckedSideEffectImports'
oneline: '检查副作用导入。'
---

在 JavaScript 中，可以通过 `import` 导入一个模块而不实际从中导入任何值。

```ts
import 'some-module'
```

此类导入通常被称为*副作用导入（side effect imports）*，因为它们能提供的唯一有效行为就是执行某些副作用（例如注册全局变量，或者向原型链添加 polyfill）。

默认情况下，TypeScript 不会检查这些导入的有效性。如果导入能够解析到有效的源文件，TypeScript 会加载并检查该文件。如果未找到任何源文件，TypeScript 则会静默忽略该导入。

这种行为可能会令人意外，但它在一定程度上源于对 JavaScript 生态系统中常见模式的兼容与支持。例如，在打包工具中配合特定的 loader 使用此语法来加载 CSS 或其他资源。你的打包工具可能被配置为允许通过编写类似如下代码来引入指定的 `.css` 文件：

```tsx
import './button-component.css'

export function Button() {
  // ...
}
```

然而，这也掩盖了副作用导入中潜在的拼写错误。

启用 `--noUncheckedSideEffectImports` 时，如果 TypeScript 无法为副作用导入找到对应的源文件，将会报错。

```ts
import 'oops-this-module-does-not-exist'
//     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
// 错误：找不到模块 'oops-this-module-does-not-exist' 或其对应的
//       类型声明。
```

启用此选项后，某些原本能正常运行的代码可能会收到错误提示，例如上述 CSS 示例。为了解决这一问题，对于只想为静态资源编写副作用 `import` 的用户，更好的方式是编写包含通配符说明符的*环境模块声明（ambient module declaration）*。该声明可以放在全局文件中，内容类似于：

```ts
// ./src/globals.d.ts

// 将所有 CSS 文件识别为模块导入。
declare module '*.css' {}
```

实际上，你的项目中可能已经存在类似的文件了！例如运行 `vite init` 之类的命令可能会创建类似的 `vite-env.d.ts`。
