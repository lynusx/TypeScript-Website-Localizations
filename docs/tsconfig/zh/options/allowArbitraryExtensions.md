---
display: 'Allow Arbitrary Extensions'
oneline: '允许导入具有任意扩展名的文件（只要存在对应的声明文件）。'
---

在 TypeScript 5.0 中，当导入路径以非已知的 JavaScript 或 TypeScript 文件扩展名结尾时，编译器将以 `{file basename}.d.{extension}.ts` 的形式查找该路径对应的声明文件。
例如，如果你在打包工具项目中使用 CSS 加载器（loader），你可能希望为这些样式表编写（或生成）声明文件：

```css
/* app.css */
.cookie-banner {
  display: none;
}
```

```ts
// app.d.css.ts
declare const css: {
  cookieBanner: string
}
export default css
```

```ts
// App.tsx
import styles from './app.css'

styles.cookieBanner // string 类型
```

默认情况下，此类导入会报错，以提醒你 TypeScript 无法理解该文件类型，且你的运行时环境可能不支持导入它。
但如果你已经配置了运行时或打包工具来处理该文件，则可以通过新增的 `--allowArbitraryExtensions` 编译器选项来消除此错误。

注意，从历史上看，通常可以通过添加名为 `app.css.d.ts` 而不是 `app.d.css.ts` 的声明文件来实现类似的效果——然而，这只是依靠 Node 的 CommonJS `require` 解析规则才能生效。
严格来说，前者会被解释为名为 `app.css.js` 的 JavaScript 文件的声明文件。
因为在 Node 的 ESM 支持中相对路径导入必须包含扩展名，所以在 `--moduleResolution node16` 或 `nodenext` 下的 ESM 文件中，TypeScript 会在上述示例中报错。

有关更多信息，请查阅[该特性的提案](https://github.com/microsoft/TypeScript/issues/50133)及其[对应的 Pull Request](https://github.com/microsoft/TypeScript/pull/51435)。
