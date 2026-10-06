---
title: 模板字面量类型
layout: docs
permalink: /zh/docs/handbook/2/template-literal-types.html
oneline: '通过模板字面量字符串修改属性来生成映射类型。'
---

模板字面量类型建立在[字符串字面量类型](/docs/handbook/2/everyday-types.html#literal-types)之上，并能够通过联合类型展开为多个字符串。

它们与 [JavaScript 中的模板字面量字符串](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)具有相同的语法，但用于类型位置。
当与具体字面量类型一起使用时，模板字面量通过拼接内容来生成新的字符串字面量类型。

```ts twoslash
type World = 'world'

type Greeting = `hello ${World}`
//   ^?
```

当在插值位置使用联合类型时，该类型表示每个联合成员所能代表的所有可能字符串字面量的集合：

```ts twoslash
type EmailLocaleIDs = 'welcome_email' | 'email_heading'
type FooterLocaleIDs = 'footer_title' | 'footer_sendoff'

type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`
//   ^?
```

对于模板字面量中的每个插值位置，多个联合类型会进行交叉相乘：

```ts twoslash
type EmailLocaleIDs = 'welcome_email' | 'email_heading'
type FooterLocaleIDs = 'footer_title' | 'footer_sendoff'
// ---cut---
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`
type Lang = 'en' | 'ja' | 'pt'

type LocaleMessageIDs = `${Lang}_${AllLocaleIDs}`
//   ^?
```

对于庞大的字符串联合类型，我们通常建议提前生成，但在较小规模的场景下这非常有用。

### 类型中的字符串联合

模板字面量的强大之处在于根据类型内部的信息定义新字符串。

考虑这样一种场景：某个函数（`makeWatchedObject`）为传入的对象添加了一个名为 `on()` 的新函数。在 JavaScript 中，其调用可能类似于：
`makeWatchedObject(baseObject)`。我们可以想象基础对象长成这样：

```ts twoslash
// @noErrors
const passedObject = {
  firstName: 'Saoirse',
  lastName: 'Ronan',
  age: 26,
}
```

将要添加到基础对象的 `on` 函数接收两个参数：`eventName`（`string` 类型）和 `callback`（`function` 类型）。

`eventName` 的形式应为 `attributeInThePassedObject + "Changed"`；例如，基于基础对象中的属性 `firstName` 推导出的 `firstNameChanged`。

`callback` 函数在被调用时：

- 应当被传入一个与 `attributeInThePassedObject` 名称关联类型相匹配的值；例如，由于 `firstName` 的类型是 `string`，因此 `firstNameChanged` 事件的回调在调用时期望接收一个 `string` 参数。类似地，与 `age` 相关的事件期望在调用时传入一个 `number` 参数
- 应当拥有 `void` 返回值类型（为了演示简单起见）

因此，`on()` 的朴素函数签名可能是：`on(eventName: string, callback: (newValue: any) => void)`。然而，在前面的描述中，我们指出了希望在代码中体现的重要类型约束。模板字面量类型允许我们将这些约束带入代码中。

```ts twoslash
// @noErrors
declare function makeWatchedObject(obj: any): any
// ---cut---
const person = makeWatchedObject({
  firstName: 'Saoirse',
  lastName: 'Ronan',
  age: 26,
})

// makeWatchedObject 已将 `on` 添加到匿名对象中

person.on('firstNameChanged', (newValue) => {
  console.log(`firstName was changed to ${newValue}!`)
})
```

请注意，`on` 监听的是事件 `"firstNameChanged"`，而不仅仅是 `"firstName"`。如果我们能确保合法事件名称的集合受限于被监听对象属性名联合并在末尾添加 "Changed"，那么我们最初对 `on()` 的简单定义就会更加健壮。虽然我们很习惯在 JavaScript 中进行这种计算，例如 ``Object.keys(passedObject).map(x => `${x}Changed`)``，但*类型系统内部*的模板字面量提供了类似的字符串操作方式：

```ts twoslash
type PropEventSource<Type> = {
  on(
    eventName: `${string & keyof Type}Changed`,
    callback: (newValue: any) => void,
  ): void
}

/// 创建一个带有 `on` 方法的“被监听对象”，
/// 以便你可以监听属性的更改。
declare function makeWatchedObject<Type>(
  obj: Type,
): Type & PropEventSource<Type>
```

借助这个定义，我们构建的代码在传入错误属性时就会报错：

```ts twoslash
// @errors: 2345
type PropEventSource<Type> = {
  on(
    eventName: `${string & keyof Type}Changed`,
    callback: (newValue: any) => void,
  ): void
}

declare function makeWatchedObject<T>(obj: T): T & PropEventSource<T>
// ---cut---
const person = makeWatchedObject({
  firstName: 'Saoirse',
  lastName: 'Ronan',
  age: 26,
})

person.on('firstNameChanged', () => {})

// 防止常见人为错误（使用键名而非事件名称）
person.on('firstName', () => {})

// 能够防范拼写错误
person.on('frstNameChanged', () => {})
```

### 模板字面量的类型推断

注意，我们并没有充分利用原始传入对象所提供的所有信息。当 `firstName` 改变时（即发生 `firstNameChanged` 事件），我们应当期望回调接收一个 `string` 类型的参数。类似地，针对 `age` 改变的回调应当接收一个 `number` 参数。我们草率地使用了 `any` 来作为 `callback` 参数的类型。同样地，模板字面量类型使得确保属性的数据类型与其回调函数的第一个参数类型相同成为可能。

使其成为可能的关键洞察在于：我们可以使用带有泛型的函数，从而实现：

1. 第一个参数中使用的字面量被捕获为字面量类型
2. 该字面量类型可以被验证为属于泛型中有效属性的联合
3. 使用索引访问在泛型的结构中查找已验证属性的类型
4. *然后*将该类型信息应用于确保回调函数的参数具有相同类型

```ts twoslash
type PropEventSource<Type> = {
  on<Key extends string & keyof Type>(
    eventName: `${Key}Changed`,
    callback: (newValue: Type[Key]) => void,
  ): void
}

declare function makeWatchedObject<Type>(
  obj: Type,
): Type & PropEventSource<Type>

const person = makeWatchedObject({
  firstName: 'Saoirse',
  lastName: 'Ronan',
  age: 26,
})

person.on('firstNameChanged', (newName) => {
  //                        ^?
  console.log(`new name is ${newName.toUpperCase()}`)
})

person.on('ageChanged', (newAge) => {
  //                  ^?
  if (newAge < 0) {
    console.warn('warning! negative age')
  }
})
```

在这里，我们将 `on` 变成了一个泛型方法。

当用户使用字符串 `"firstNameChanged"` 调用时，TypeScript 会尝试为 `Key` 推断出正确的类型。
为此，它将 `Key` 与 `"Changed"` 之前的内容进行匹配，并推断出字符串 `"firstName"`。
一旦 TypeScript 计算出这一点，`on` 方法就可以获取原始对象上 `firstName` 的类型，在此例中为 `string`。
类似地，当使用 `"ageChanged"` 调用时，TypeScript 会找到属性 `age` 的类型，即 `number`。

推断可以以不同的方式进行组合，通常用于解构字符串，并以不同方式重新构建它们。

## 内置字符串操作类型

为了协助字符串操作，TypeScript 包含了一组可用于字符串操作的类型。出于性能考虑，这些类型内置于编译器中，在 TypeScript 附带的 `.d.ts` 文件中是找不到的。

### `Uppercase<StringType>`

将字符串中的每个字符转换为大写版本。

##### 示例

```ts twoslash
type Greeting = 'Hello, world'
type ShoutyGreeting = Uppercase<Greeting>
//   ^?

type ASCIICacheKey<Str extends string> = `ID-${Uppercase<Str>}`
type MainID = ASCIICacheKey<'my_app'>
//   ^?
```

### `Lowercase<StringType>`

将字符串中的每个字符转换为对应的小写版本。

##### 示例

```ts twoslash
type Greeting = 'Hello, world'
type QuietGreeting = Lowercase<Greeting>
//   ^?

type ASCIICacheKey<Str extends string> = `id-${Lowercase<Str>}`
type MainID = ASCIICacheKey<'MY_APP'>
//   ^?
```

### `Capitalize<StringType>`

将字符串中的首字母转换为对应的大写版本。

##### 示例

```ts twoslash
type LowercaseGreeting = 'hello, world'
type Greeting = Capitalize<LowercaseGreeting>
//   ^?
```

### `Uncapitalize<StringType>`

将字符串中的首字母转换为对应的小写版本。

##### 示例

```ts twoslash
type UppercaseGreeting = 'HELLO WORLD'
type UncomfortableGreeting = Uncapitalize<UppercaseGreeting>
//   ^?
```

<details>
    <summary>内置字符串操作类型的技术细节</summary>
    <p>从 TypeScript 4.1 开始，这些内置函数的底层代码直接使用 JavaScript 运行时字符串函数进行操作，并且不感知语言环境（locale）。</p>
    <code><pre>
function applyStringMapping(symbol: Symbol, str: string) {
    switch (intrinsicTypeKinds.get(symbol.escapedName as string)) {
        case IntrinsicTypeKind.Uppercase: return str.toUpperCase();
        case IntrinsicTypeKind.Lowercase: return str.toLowerCase();
        case IntrinsicTypeKind.Capitalize: return str.charAt(0).toUpperCase() + str.slice(1);
        case IntrinsicTypeKind.Uncapitalize: return str.charAt(0).toLowerCase() + str.slice(1);
    }
    return str;
}</pre></code>
</details>
