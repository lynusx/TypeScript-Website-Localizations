---
title: TypeScript 1.1
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-1-1.html
oneline: TypeScript 1.1 发布说明
---

## 性能提升

1.1 编译器通常比以往任何版本都要快 4 倍左右。有关令人瞩目的对比图表，请参阅[这篇博文](https://web.archive.org/web/20141007020020/http://blogs.msdn.com/b/typescript/archive/2014/10/06/announcing-typescript-1-1-ctp.aspx)。

## 更好的模块可见性规则

TypeScript 现在仅在指定了 [`declaration`](/tsconfig#declaration) 标志时，才会严格强制执行模块中类型的可见性规则。这在 Angular 等场景中非常有用，例如：

```ts
module MyControllers {
  interface ZooScope extends ng.IScope {
    animals: Animal[]
  }
  export class ZooController {
    // Used to be an error (cannot expose ZooScope), but now is only
    // an error when trying to generate .d.ts files
    constructor(public $scope: ZooScope) {}
    /* more code */
  }
}
```
