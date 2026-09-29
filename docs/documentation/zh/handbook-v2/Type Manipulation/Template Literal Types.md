---
title: 模板字面量类型
layout: docs
permalink: /zh/docs/handbook/2/template-literal-types.html
oneline: '通过模板字面量字符串更改属性并生成映射类型。'
---

模板字面量类型建立在 [字符串字面量类型](/docs/handbook/2/everyday-types.html#literal-types) 之上，并能够通过联合类型扩展成多个字符串。

它们的语法与 [JavaScript 中的模板字面量字符串](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Template_literals) 相同，但用于类型定义的位置。当与具体的字面量类型搭配使用时，模板字面量会通过拼接内容产生一个新的字符串字面量类型。

```ts twoslash
type World = 'world'

type Greeting = `hello ${World}`
//   ^?
```

如果在插值位置使用了联合类型，结果类型将是每个联合成员所能表示的所有可能字符串字面量的集合：

```ts twoslash
type EmailLocaleIDs = 'welcome_email' | 'email_heading'
type FooterLocaleIDs = 'footer_title' | 'footer_sendoff'

type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`
//   ^?
```

对于模板字面量中的每个插值位置，其中的联合类型会进行交叉相乘：

```ts twoslash
type EmailLocaleIDs = 'welcome_email' | 'email_heading'
type FooterLocaleIDs = 'footer_title' | 'footer_sendoff'
// ---cut---
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`
type Lang = 'en' | 'ja' | 'pt'

type LocaleMessageIDs = `${Lang}_${AllLocaleIDs}`
//   ^?
```

通常建议针对大型的字符串联合类型采用提前生成的方式，但在规模较小的场景下，这种特性非常实用。

### 类型中的字符串联合类型

当基于类型内部的信息定义新字符串时，模板字面量才能真正发挥其威力。

假设有一个函数（`makeWatchedObject`）会为传入的对象添加一个新的 `on()` 方法。在 JavaScript 中，调用方式可能是：`makeWatchedObject(baseObject)`。假设基础对象如下所示：

```ts twoslash
// @noErrors
const passedObject = {
  firstName: 'Saoirse',
  lastName: 'Ronan',
  age: 26,
}
```

将被添加到基础对象的 `on` 方法预期接收两个参数：一个是 `eventName`（`string` 类型），另一个是 `callback`（`function` 类型）。

`eventName` 的格式应当为 `基础对象中的属性名 + "Changed"`；因此，基于基础对象中的 `firstName` 属性，会派生出 `firstNameChanged`。

调用 `callback` 函数时需满足：

- 应传入一个与 `基础对象中` 属性名类型相符的值；因此，由于 `firstName` 的类型是 `string`，`firstNameChanged` 事件的回调在调用时预期会接收到一个 `string`。同理，与 `age` 相关的事件在调用时预期会接收一个 `number` 类型的参数。
- 返回值类型应为 `void`（为便于演示）。

因此，`on()` 最基础的函数签名可能是这样：`on(eventName: string, callback: (newValue: any) => void)`。然而，在前面的描述中，我们已经识别出了一些重要的类型约束，希望在代码中体现出来。模板字面量类型让我们可以把这些约束带入代码中。

```ts twoslash
// @noErrors
declare function makeWatchedObject(obj: any): any
// ---cut---
const person = makeWatchedObject({
  firstName: 'Saoirse',
  lastName: 'Ronan',
  age: 26,
})

// makeWatchedObject 已为匿名对象添加了 `on` 方法

person.on('firstNameChanged', (newValue) => {
  console.log(`firstName was changed to ${newValue}!`)
})
```

注意，`on` 监听的是 `"firstNameChanged"` 事件，而不仅仅是 `"firstName"`。如果我们能确保合法的事件名集合被严格约束为“被监听对象的属性名与 'Changed' 拼接后的联合类型”，那么 `on()` 的基础定义就会变得更加健壮。虽然在 JavaScript 中进行此类计算非常简单（即 ``Object.keys(passedObject).map(x => `${x}Changed`)``，但 _在类型系统内部_，模板字面量提供了一种类似的字符串操作方法：

```ts twoslash
type PropEventSource<Type> = {
  on(
    eventName: `${string & keyof Type}Changed`,
    callback: (newValue: any) => void,
  ): void
}

/// 创建一个带有 `on` 方法的“被监听对象”
/// 以便你能监听属性的变化。
declare function makeWatchedObject<Type>(
  obj: Type,
): Type & PropEventSource<Type>
```

如此一来，当传入错误的属性时，代码就能抛出错误提示：

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

// 防止简单的人为错误（误用键名而非事件名）
person.on('firstName', () => {})

// 具备防拼写错误能力
person.on('frstNameChanged', () => {})
```

### 模板字面量的类型推断

注意，我们尚未充分利用原始传入对象所提供的全部信息。当 `firstName` 发生变化（即触发 `firstNameChanged` 事件）时，回调函数理应接收一个 `string` 类型的参数。同理，`age` 变化的回调应当接收一个 `number` 类型的参数。我们刚才仅仅简单地使用了 `any` 作为 `callback` 的参数类型。现在，利用模板字面量类型，我们可以确保属性的数据类型与该属性回调函数的首个参数类型保持一致。

实现这一点的关键在于：我们可以利用带泛型的函数，使得：

1. 将首个参数中使用的字面量捕获为字面量类型。
2. 验证该字面量类型是否属于泛型中合法属性的联合类型。
3. 利用索引访问类型，在泛型结构中查找已验证属性对应的类型。
4. _然后_ 应用此类型信息，确保回调函数的参数类型与之相同。

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

在这里，我们将 `on` 改造为了泛型方法。

当用户传入字符串 `"firstNameChanged"` 调用它时，TypeScript 会尝试为 `Key` 推断出正确的类型。为此，它会将 `Key` 与 `"Changed"` 前面的内容进行匹配，进而推断出字符串 `"firstName"`。一旦 TypeScript 计算出结果，`on` 方法就可以在原始对象上获取 `firstName` 的类型（在这个例子中是 `string`）。同样，当使用 `"ageChanged"` 调用时，TypeScript 会找到 `age` 属性对应的 `number` 类型。

类型推断可以以不同的方式组合使用，通常用于解构字符串，然后再以不同的方式进行重构。

## 内置的字符串操作类型

为了辅助字符串操作，TypeScript 内置了一组可用于字符串操作的类型。出于性能考虑，这些类型内置于编译器中，你在 TypeScript 附带的 `.d.ts` 文件里是找不到它们的。

### `Uppercase<StringType>`

将字符串中的每个字符转换为大写形式。

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

将字符串中的每个字符转换为小写形式。

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

将字符串的第一个字符转换为大写形式。

##### 示例

```ts twoslash
type LowercaseGreeting = 'hello, world'
type Greeting = Capitalize<LowercaseGreeting>
//   ^?
```

### `Uncapitalize<StringType>`

将字符串的第一个字符转换为小写形式。

##### 示例

```ts twoslash
type UppercaseGreeting = 'HELLO WORLD'
type UncomfortableGreeting = Uncapitalize<UppercaseGreeting>
//   ^?
```

<details>
    <summary>关于内置字符串操作类型的技术细节</summary>
    <p>自 TypeScript 4.1 起，这些内置函数的代码直接使用 JavaScript 字符串运行时函数进行操作，不感知区域设置（locale）。</p>
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
