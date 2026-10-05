---
display: 'JSX Fragment Factory'
oneline: "指定生成 React JSX 代码时片段所使用的 JSX 片段引用，例如 'React.Fragment' 或 'Fragment'。"
---

在指定了 [`jsxFactory`](#jsxFactory) 编译器选项且目标为 React JSX 生成时，指定要使用的 JSX 片段工厂函数，例如 `Fragment`。

例如，使用如下 TSConfig：

```json tsconfig
{
  "compilerOptions": {
    "target": "esnext",
    "module": "commonjs",
    "jsx": "react",
    "jsxFactory": "h",
    "jsxFragmentFactory": "Fragment"
  }
}
```

此 TSX 文件：

```tsx
import { h, Fragment } from 'preact'

const HelloWorld = () => (
  <>
    <div>Hello</div>
  </>
)
```

输出将如下所示：

```tsx twoslash
// @showEmit
// @showEmittedFile: index.js
// @jsxFactory: h
// @jsxFragmentFactory: Fragment
// @noErrors
// @target: esnext
// @module: commonjs

import { h, Fragment } from 'preact'

const HelloWorld = () => (
  <>
    <div>Hello</div>
  </>
)
```

该选项也可以像 [Babel 的 `/* @jsxFrag h */` 指令](https://babeljs.io/docs/en/babel-plugin-transform-react-jsx#fragments) 一样在每个文件级别单独使用。

例如：

```tsx twoslash
/** @jsx h */
/** @jsxFrag Fragment */

import { h, Fragment } from 'preact'

const HelloWorld = () => (
  <>
    <div>Hello</div>
  </>
)
```
