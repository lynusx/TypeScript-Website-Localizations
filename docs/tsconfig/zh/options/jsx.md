---
display: 'JSX'
oneline: '指定生成何种 JSX 代码。'
---

控制 JSX 语法在 JavaScript 文件中的生成方式。
这仅影响由 `.tsx` 文件编译生成的 JS 文件的输出。

- `react-jsx`：生成 `.js` 文件，将 JSX 转换为针对生产环境优化的 `_jsx` 调用
- `react-jsxdev`：生成 `.js` 文件，将 JSX 转换为仅用于开发环境的 `_jsx` 调用
- `preserve`：生成 `.jsx` 文件，保留 JSX 结构不变
- `react-native`：生成 `.js` 文件，保留 JSX 结构不变
- `react`：生成 `.js` 文件，将 JSX 转换为等效的 `React.createElement` 调用

### 示例

以下示例代码：

```tsx
export const HelloWorld = () => <h1>Hello world</h1>
```

React：`"react-jsx"`<sup>[[1]](https://reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html)</sup>

```tsx twoslash
declare module JSX {
  interface Element {}
  interface IntrinsicElements {
    [s: string]: any
  }
}
// @showEmit
// @noErrors
// @jsx: react-jsx
export const HelloWorld = () => <h1>Hello world</h1>
```

React 开发转换：`"react-jsxdev"`<sup>[[1]](https://reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html)</sup>

```tsx twoslash
declare module JSX {
  interface Element {}
  interface IntrinsicElements {
    [s: string]: any
  }
}
// @showEmit
// @noErrors
// @jsx: react-jsxdev
export const HelloWorld = () => <h1>Hello world</h1>
```

保留（Preserve）：`"preserve"`

```tsx twoslash
declare module JSX {
  interface Element {}
  interface IntrinsicElements {
    [s: string]: any
  }
}
// @showEmit
// @noErrors
// @jsx: preserve
export const HelloWorld = () => <h1>Hello world</h1>
```

React Native：`"react-native"`

```tsx twoslash
declare module JSX {
  interface Element {}
  interface IntrinsicElements {
    [s: string]: any
  }
}
// @showEmit
// @noErrors
// @jsx: react-native
export const HelloWorld = () => <h1>Hello world</h1>
```

传统 React 运行时：`"react"`

```tsx twoslash
declare module JSX {
  interface Element {}
  interface IntrinsicElements {
    [s: string]: any
  }
}
// @showEmit
// @noErrors
export const HelloWorld = () => <h1>Hello world</h1>
```

该选项也可以通过 `@jsxRuntime` 注释在每个文件级别单独使用。

在该文件中始终使用经典运行时（`"react"`）：

```tsx
/* @jsxRuntime classic */
export const HelloWorld = () => <h1>Hello world</h1>
```

在该文件中始终使用自动运行时（`"react-jsx"`）：

```tsx
/* @jsxRuntime automatic */
export const HelloWorld = () => <h1>Hello world</h1>
```
