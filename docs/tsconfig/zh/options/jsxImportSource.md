---
display: 'JSX Import Source'
oneline: '在使用 `jsx: react-jsx*` 时，指定用于导入 JSX 工厂函数的模块标识符。'
---

当 [`jsx`](#jsx) 设置为 TypeScript 4.1 引入的 `"react-jsx"` 或 `"react-jsxdev"` 时，声明用于导入 `jsx` 和 `jsxs` 工厂函数的模块标识符。

从 [React 17](https://reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html) 开始，该库支持通过独立导入的新形式 JSX 转换。

例如，对于以下代码：

```tsx
import React from 'react'

function App() {
  return <h1>Hello World</h1>
}
```

使用如下 TSConfig：

```json tsconfig
{
  "compilerOptions": {
    "target": "esnext",
    "module": "commonjs",
    "jsx": "react-jsx"
  }
}
```

TypeScript 生成的 JavaScript 输出为：

```tsx twoslash
// @showEmit
// @noErrors
// @jsx: react-jsx
// @module: commonjs
// @target: esnext
declare module JSX {
  interface Element {}
  interface IntrinsicElements {
    [s: string]: any
  }
}
import React from 'react'

function App() {
  return <h1>Hello World</h1>
}
```

例如，如果你想要使用 `"jsxImportSource": "preact"`，你需要如下的 tsconfig：

```json tsconfig
{
  "compilerOptions": {
    "target": "esnext",
    "module": "commonjs",
    "jsx": "react-jsx",
    "jsxImportSource": "preact",
    "types": ["preact"]
  }
}
```

这将生成类似如下的代码：

```tsx twoslash
// @showEmit
// @jsxImportSource: preact
// @types: preact
// @jsx: react-jsx
// @target: esnext
// @module: commonjs
// @noErrors

export function App() {
  return <h1>Hello World</h1>
}
```

此外，你也可以使用文件级 pragma 注释来设置此选项，例如：

```tsx
/** @jsxImportSource preact */

export function App() {
  return <h1>Hello World</h1>
}
```

这会为 `_jsx` 工厂函数添加 `preact/jsx-runtime` 导入。

_注意：_ 为了使其按预期工作，你的 `tsx` 文件必须包含 `export` 或 `import`，以便被视为一个模块。
