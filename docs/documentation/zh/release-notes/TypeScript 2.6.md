---
title: TypeScript 2.6
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-2-6.html
oneline: TypeScript 2.6 发布说明
---

## 严格函数类型检查

TypeScript 2.6 引入了一个新的严格检查标志 [`strictFunctionTypes`](/tsconfig#strictFunctionTypes)。
[`strictFunctionTypes`](/tsconfig#strictFunctionTypes) 是 [`strict`](/tsconfig#strict) 系列标志之一，这意味着它在 [`strict`](/tsconfig#strict) 模式下默认开启。
你可以在命令行中使用 `--strictFunctionTypes false` 或在 tsconfig.json 中将其关闭。

启用 [`strictFunctionTypes`](/tsconfig#strictFunctionTypes) 后，函数类型的参数位置将进行**逆变**检查，而非**双变**检查。
若想了解协变与逆变的背景知识，请参阅[《什么是协变和逆变？》](https://web.archive.org/web/20220823104433/https://www.stephanboyer.com/post/132/what-are-covariance-and-contravariance)。

更严格的检查适用于所有函数类型，但**方法**或**构造函数**声明中的函数类型除外。
之所以将方法排除在外，是为了确保泛型类和接口（如 `Array<T>`）仍能以协变方式正常工作。

请看下面的示例，其中 `Animal` 是 `Dog` 和 `Cat` 的父类型：

```ts
declare let f1: (x: Animal) => void
declare let f2: (x: Dog) => void
declare let f3: (x: Cat) => void
f1 = f2 // Error with --strictFunctionTypes
f2 = f1 // Ok
f2 = f3 // Error
```

第一个赋值在默认类型检查模式下是允许的，但在严格函数类型模式下会报错。
直觉上，默认模式允许该赋值，是因为它**可能**是安全的；而严格函数类型模式将其视为错误，因为无法**确定**它是安全的。
在任何模式下，第三个赋值都是错误的，因为它**永远**不安全。

换一种描述方式：在默认类型检查模式下，类型 `(x: T) => void` 对 `T` 是**双变**的（即协变**或**逆变）；而在严格函数类型模式下，对 `T` 是**逆变**的。

##### 示例

```ts
interface Comparer<T> {
  compare: (a: T, b: T) => number
}

declare let animalComparer: Comparer<Animal>
declare let dogComparer: Comparer<Dog>

animalComparer = dogComparer // Error
dogComparer = animalComparer // Ok
```

第一个赋值现在会报错。实际上，`T` 在 `Comparer<T>` 中是逆变的，因为它只出现在函数类型的参数位置。

顺便提一下，与某些语言（如 C# 和 Scala）需要显式标注方差（`out`/`in` 或 `+`/`-`）不同，TypeScript 的结构化类型系统会根据类型参数在泛型类型中的实际使用方式自然推导出方差。

##### 注意

在 [`strictFunctionTypes`](/tsconfig#strictFunctionTypes) 下，如果 `compare` 被声明为方法，第一个赋值仍然是允许的。
这是因为 `T` 在 `Comparer<T>` 中是双变的——它只出现在方法参数位置。

```ts
interface Comparer<T> {
  compare(a: T, b: T): number
}

declare let animalComparer: Comparer<Animal>
declare let dogComparer: Comparer<Dog>

animalComparer = dogComparer // Ok because of bivariance
dogComparer = animalComparer // Ok
```

TypeScript 2.6 还改进了逆变位置上的类型推断：

```ts
function combine<T>(...funcs: ((x: T) => void)[]): (x: T) => void {
  return (x) => {
    for (const f of funcs) f(x)
  }
}

function animalFunc(x: Animal) {}
function dogFunc(x: Dog) {}

let combined = combine(animalFunc, dogFunc) // (x: Dog) => void
```

在上面的例子中，所有对 `T` 的推断都来自逆变位置，因此我们推断出 `T` 的**最优公共子类型**。
这与来自协变位置的推断相反——协变位置推断的是**最优公共父类型**。

## 在模块中缓存带标签的模板对象

TypeScript 2.6 修复了带标签模板字符串的代码生成，使其更符合 ECMAScript 规范。
根据 [ECMAScript 规范](https://tc39.github.io/ecma262/#sec-gettemplateobject)，每次对模板标签求值时，应将**相同的**模板字符串对象（即同一个 `TemplateStringsArray`）作为第一个参数传入。
在 TypeScript 2.6 之前，每次生成的都是全新的模板对象。
虽然字符串内容相同，但这种行为会影响依赖字符串标识进行缓存失效的库，例如 [lit-html](https://github.com/PolymerLabs/lit-html/issues/58)。

##### 示例

```ts
export function id(x: TemplateStringsArray) {
  return x
}

export function templateObjectFactory() {
  return id`hello world`
}

let result = templateObjectFactory() === templateObjectFactory() // true in TS 2.6
```

生成的代码如下：

```js
'use strict'
var __makeTemplateObject =
  (this && this.__makeTemplateObject) ||
  function (cooked, raw) {
    if (Object.defineProperty) {
      Object.defineProperty(cooked, 'raw', { value: raw })
    } else {
      cooked.raw = raw
    }
    return cooked
  }

function id(x) {
  return x
}

var _a
function templateObjectFactory() {
  return id(_a || (_a = __makeTemplateObject(['hello world'], ['hello world'])))
}

var result = templateObjectFactory() === templateObjectFactory()
```

> 注意：此变更引入了一个新的生成辅助函数 `__makeTemplateObject`；
> 如果你将 [`importHelpers`](/tsconfig#importHelpers) 与 [`tslib`](https://github.com/Microsoft/tslib) 配合使用，则需要更新到 1.8 或更高版本。

## 命令行本地化诊断信息

TypeScript 2.6 的 npm 包内置了 13 种语言的本地化诊断信息。
使用命令行的 `--locale` 标志即可显示对应语言的本地化消息。

##### 示例

俄语错误信息：

```sh
c:\ts>tsc --v
Version 2.6.0-dev.20171003

c:\ts>tsc --locale ru --pretty c:\test\a.ts

../test/a.ts(1,5): error TS2322: Тип ""string"" не может быть назначен для типа "number".

1 var x: number = "string";
      ~
```

日语帮助信息：

```sh
PS C:\ts> tsc --v
Version 2.6.0-dev.20171003

PS C:\ts> tsc --locale ja-jp
バージョン 2.6.0-dev.20171003
構文: tsc [オプション] [ファイル ...]

例:  tsc hello.ts
    tsc --outFile file.js file.ts
    tsc @args.txt

オプション:
 -h, --help                                 このメッセージを表示します。
 --all                                      コンパイラ オプションをすべて表示します。
 -v, --version                              コンパイラのバージョンを表示します。
 --init                                     TypeScript プロジェクトを初期化して、tsconfig.json ファイルを作成します。
 -p ファイルまたはディレクトリ, --project ファイルまたはディレクトリ  構成ファイルか、'tsconfig.json' を含むフォルダーにパスが指定されたプロジェクトをコ
ンパイルします。
 --pretty                                   色とコンテキストを使用してエラーとメッセージにスタイルを適用します (試験的)。
 -w, --watch                                入力ファイルを監視します。
 -t バージョン, --target バージョン                   ECMAScript のターゲット バージョンを指定します: 'ES3' (既定)、'ES5'、'ES2015'、'ES2016'、'ES2017'、'ES
NEXT'。
 -m 種類, --module 種類                         モジュール コード生成を指定します: 'none'、'commonjs'、'amd'、'system'、'umd'、'es2015'、'ESNext'。
 --lib                                      コンパイルに含めるライブラリ ファイルを指定します:
                                              'es5' 'es6' 'es2015' 'es7' 'es2016' 'es2017' 'esnext' 'dom' 'dom.iterable' 'webworker' 'scripthost' 'es201
5.core' 'es2015.collection' 'es2015.generator' 'es2015.iterable' 'es2015.promise' 'es2015.proxy' 'es2015.reflect' 'es2015.symbol' 'es2015.symbol.wellkno
wn' 'es2016.array.include' 'es2017.object' 'es2017.sharedmemory' 'es2017.string' 'es2017.intl' 'esnext.asynciterable'
 --allowJs                                  javascript ファイルのコンパイルを許可します。
 --jsx 種類                                   JSX コード生成を指定します: 'preserve'、'react-native'、'react'。
 -d, --declaration                          対応する '.d.ts' ファイルを生成します。
 --sourceMap                                対応する '.map' ファイルを生成します。
 --outFile ファイル                             出力を連結して 1 つのファイルを生成します。
 --outDir ディレクトリ                            ディレクトリへ出力構造をリダイレクトします。
 --removeComments                           コメントを出力しないでください。
 --noEmit                                   出力しないでください。
 --strict                                   strict 型チェックのオプションをすべて有効にします。
 --noImplicitAny                            暗黙的な 'any' 型を含む式と宣言に関するエラーを発生させます。
 --strictNullChecks                         厳格な null チェックを有効にします。
 --noImplicitThis                           暗黙的な 'any' 型を持つ 'this' 式でエラーが発生します。
 --alwaysStrict                             厳格モードで解析してソース ファイルごとに "use strict" を生成します。
 --noUnusedLocals                           使用されていないローカルに関するエラーを報告します。
 --noUnusedParameters                       使用されていないパラメーターに関するエラーを報告します。
 --noImplicitReturns                        関数の一部のコード パスが値を返さない場合にエラーを報告します。
 --noFallthroughCasesInSwitch               switch ステートメントに case のフォールスルーがある場合にエラーを報告します。
 --types                                    コンパイルに含む型宣言ファイル。
 @<ファイル>
```

## 使用 '// @ts-ignore' 注释抑制 .ts 文件中的错误

TypeScript 2.6 支持在 .ts 文件中通过 `// @ts-ignore` 注释来抑制错误，该注释需放在出错行的上方。

##### 示例

```ts
if (false) {
  // @ts-ignore: Unreachable code error
  console.log('hello')
}
```

`// @ts-ignore` 注释会抑制紧随其后那一行产生的所有错误。
建议在 `@ts-ignore` 之后的注释中说明被抑制的是哪个错误。

请注意，此注释仅抑制错误上报，我们建议**非常谨慎**地使用它。

## 更快的 `tsc --watch`

TypeScript 2.6 带来了更快的 `--watch` 实现。
新版本针对使用 ES 模块的代码库优化了代码生成和类型检查流程。
当某个模块文件发生变更时，只会重新生成该模块及其依赖文件，而非重新构建整个项目。
文件数量较多的项目将从中获益最大。

新实现还提升了 tsserver 中监听模式的性能。
监听逻辑已全面重写，能够更快地响应文件变更事件。

## 只写引用现在也会被标记为未使用

TypeScript 2.6 修订了 [`noUnusedLocals`](/tsconfig#noUnusedLocals) 和 [`noUnusedParameters`](/tsconfig#noUnusedParameters) [编译器选项](/docs/handbook/compiler-options.html)的实现。
只被写入、从未被读取的声明现在也会被标记为未使用。

##### 示例

下面的 `n` 和 `m` 都会被标记为未使用，因为它们的值从未被**读取**。而此前 TypeScript 只会检查它们的值是否被**引用**。

```ts
function f(n: number) {
  n = 0
}

class C {
  private m: number
  constructor() {
    this.m = 0
  }
}
```

此外，只在自身函数体内被调用的函数也会被视为未使用。

##### 示例

```ts
function f() {
  f() // Error: 'f' is declared but its value is never read
}
```
