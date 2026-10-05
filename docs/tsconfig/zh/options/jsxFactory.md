---
display: 'JSX Factory'
oneline: "指定生成 React JSX 代码时使用的 JSX 工厂函数，例如 'React.createElement' 或 'h'。"
---

更改使用经典 JSX 运行时编译 JSX 元素时在 `.js` 文件中调用的函数。
最常见的更改是在使用 `preact` 时，使用 `"h"` 或 `"preact.h"` 代替默认的 `"React.createElement"`。

例如，此 TSX 文件：

```tsx
import { h } from 'preact'

const HelloWorld = () => <div>Hello</div>
```

在配置 `jsxFactory: "h"` 时的输出如下：

```tsx twoslash
// @showEmit
// @showEmittedFile: index.js
// @jsxFactory: h
// @noErrors
// @target: esnext
// @module: commonjs

import { h, Fragment } from 'preact'

const HelloWorld = () => <div>Hello</div>
```

该选项也可以像 [Babel 的 `/** @jsx h */` 指令](https://babeljs.io/docs/en/babel-plugin-transform-react-jsx#custom) 一样在每个文件级别单独使用。

```tsx twoslash
/** @jsx h */
import { h } from 'preact'

const HelloWorld = () => <div>Hello</div>
```

所选的工厂函数还会影响编译器在回退到全局命名空间之前，在何处查找 `JSX` 命名空间（以获取类型检查信息）。

如果工厂函数定义为 `React.createElement`（默认值），编译器会在检查全局 `JSX` 之前先检查 `React.JSX`。如果工厂函数定义为 `h`，它会在检查全局 `JSX` 之前先检查 `h.JSX`。
