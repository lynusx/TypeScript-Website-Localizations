---
title: TypeScript 3.1
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-3-1.html
oneline: TypeScript 3.1 发布说明
---

## 元组和数组上的映射类型

在 TypeScript 3.1 中，作用于元组和数组的映射对象类型<sup>[[1]](#ts-3-1-only-homomorphic)</sup>现在会生成新的元组/数组，而不是创建一个诸如 `push()`、`pop()` 和 `length` 等成员都被转换过的新类型。
例如：

```ts
type MapToPromise<T> = { [K in keyof T]: Promise<T[K]> }

type Coordinate = [number, number]

type PromiseCoordinate = MapToPromise<Coordinate> // [Promise<number>, Promise<number>]
```

`MapToPromise` 接收一个类型 `T`，当该类型是像 `Coordinate` 这样的元组时，只有数字命名的属性会被转换。
在 `[number, number]` 中，存在两个数字命名的属性：`0` 和 `1`。
当传入这样一个元组时，`MapToPromise` 会创建一个新的元组，其中 `0` 和 `1` 属性是对应原始类型的 `Promise`。
因此，最终的结果类型 `PromiseCoordinate` 的类型为 `[Promise<number>, Promise<number>]`。

## 函数上的属性声明

TypeScript 3.1 带来了在函数声明以及通过 `const` 声明的函数上定义属性的能力，只需在同一作用域内直接给这些函数的属性赋值即可。
这使我们能够编写符合规范的 JavaScript 代码，而无需借助 `namespace` 变通方案。
例如：

```ts
function readImage(path: string, callback: (err: any, image: Image) => void) {
  // ...
}

readImage.sync = (path: string) => {
  const contents = fs.readFileSync(path)
  return decodeImageSync(contents)
}
```

在此处，我们定义了一个以非阻塞异步方式读取图像的函数 `readImage`。
除 `readImage` 之外，我们还在 `readImage` 本身挂载了一个名为 `readImage.sync` 的便捷函数。

虽然 ECMAScript 导出通常是提供此类功能的更佳方式，但这项新支持使得以此种风格编写的代码在 TypeScript 中也能“开箱即用”。
此外，这种属性声明方式还允许我们在 React 函数组件（此前称为 SFC）上表达诸如 `defaultProps` 和 `propTypes` 等常见模式。

```ts
export const FooComponent = ({ name }) => <div>Hello! I am {name}</div>;

FooComponent.defaultProps = {
  name: "(anonymous)",
};
```

<!--
fs.readFile(path, (err, data) => {
        if (err) callback(err, undefined);
        else decodeImage(data, (err, image) => {
            if (err) callback(err, undefined);
            else callback(undefined, image);
        });
    });
-->

---

<sup id="ts-3-1-only-homomorphic">[1]</sup> 更具体地说，是形如上述示例的同态映射类型（homomorphic mapped types）。

## 通过 `typesVersions` 进行版本选择

来自社区的反馈以及我们自身的经验都表明，在利用最新 TypeScript 特性的同时还要兼顾旧版本的用户是一件非常困难的事。
TypeScript 引入了一项名为 `typesVersions` 的新特性，用以帮助应对这些场景。

你可以在[声明文件部分的“发布”章节](https://www.typescriptlang.org/docs/handbook/declaration-files/publishing.html#version-selection-with-typesversions)中了解相关内容。
