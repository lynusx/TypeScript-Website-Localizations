---
display: 'Root Dirs'
oneline: '允许在解析模块时将多个文件夹视为同一个。'
---

使用 `rootDirs`，你可以通知编译器存在多个充当单个根目录的“虚拟”目录。
这允许编译器在这些“虚拟”目录内解析相对模块导入，就如同它们被合并到了同一个目录中一样。

例如：

```
 src
 └── views
     └── view1.ts (can import "./template1", "./view2`)
     └── view2.ts (can import "./template1", "./view1`)

 generated
 └── templates
         └── views
             └── template1.ts (can import "./view1", "./view2")
```

```json tsconfig
{
  "compilerOptions": {
    "rootDirs": ["src/views", "generated/templates/views"]
  }
}
```

这不会影响 TypeScript 输出 JavaScript 的方式，它仅模拟了它们在运行时能够通过这些相对路径正常工作的假定。

`rootDirs` 可用于通过在另一个文件夹中为生成的 `.d.ts` 文件提供存放位置，来为非 TypeScript 或 JavaScript 文件提供独立的“类型层”。这种技术在打包应用中非常有用，尤其是当你需要 `import` 并不一定是代码的文件时：

```sh
 src
 └── index.ts
 └── css
     └── main.css
     └── navigation.css

 generated
 └── css
     └── main.css.d.ts
     └── navigation.css.d.ts
```

```json tsconfig
{
  "compilerOptions": {
    "rootDirs": ["src", "generated"]
  }
}
```

这种技术允许你提前为非代码源文件生成类型。之后，导入即可根据源文件的位置自然运作。
例如，`./src/index.ts` 可以导入文件 `./src/css/main.css`，而 TypeScript 将通过对应生成的声明文件获悉打包工具对该文件类型的处理行为。

```ts twoslash
// @filename: main.css.d.ts
export const appClass = 'mainClassF3EC2'
// ---cut---
// @filename: index.ts
import { appClass } from './main.css'
```
