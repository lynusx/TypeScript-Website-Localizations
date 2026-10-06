---
display: 'Emit Decorator Metadata'
oneline: '为源文件中带装饰器的声明生成设计类型元数据。'
---

启用对为装饰器生成类型元数据的实验性支持，该功能可与 [`reflect-metadata`](https://www.npmjs.com/package/reflect-metadata) 模块配合使用。

例如，以下是 TypeScript 源码：

```ts twoslash
// @experimentalDecorators
function LogMethod(
  target: any,
  propertyKey: string | symbol,
  descriptor: PropertyDescriptor,
) {
  console.log(target)
  console.log(propertyKey)
  console.log(descriptor)
}

class Demo {
  @LogMethod
  public foo(bar: number) {
    // 空操作
  }
}

const demo = new Demo()
```

当未将 `emitDecoratorMetadata` 设置为 true（默认值）时，生成的 JavaScript 如下：

```ts twoslash
// @experimentalDecorators
// @showEmit
function LogMethod(
  target: any,
  propertyKey: string | symbol,
  descriptor: PropertyDescriptor,
) {
  console.log(target)
  console.log(propertyKey)
  console.log(descriptor)
}

class Demo {
  @LogMethod
  public foo(bar: number) {
    // 空操作
  }
}

const demo = new Demo()
```

当将 `emitDecoratorMetadata` 设置为 true 时，生成的 JavaScript 如下：

```ts twoslash
// @experimentalDecorators
// @showEmit
// @emitDecoratorMetadata
function LogMethod(
  target: any,
  propertyKey: string | symbol,
  descriptor: PropertyDescriptor,
) {
  console.log(target)
  console.log(propertyKey)
  console.log(descriptor)
}

class Demo {
  @LogMethod
  public foo(bar: number) {
    // 空操作
  }
}

const demo = new Demo()
```
