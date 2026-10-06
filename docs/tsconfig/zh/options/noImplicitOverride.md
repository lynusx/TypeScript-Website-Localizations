---
display: 'No Implicit Override'
oneline: '确保派生类中重写基类成员的方法都标记有 override 修饰符。'
---

在使用包含继承关系的类时，如果基类中的函数被重命名，子类中重写（override）该函数的方法可能会与基类“失去同步”。

例如，假设你正在为一个音乐专辑同步系统建模：

```ts twoslash
class Album {
  download() {
    // 默认行为
  }
}

class SharedAlbum extends Album {
  download() {
    // 重写以从多个数据源获取信息
  }
}
```

随后，当你添加对机器学习生成播放列表的支持时，你重构了 `Album` 类，改用 `setup` 函数：

```ts twoslash
class Album {
  setup() {
    // 默认行为
  }
}

class MLAlbum extends Album {
  setup() {
    // 重写以从算法获取信息
  }
}

class SharedAlbum extends Album {
  download() {
    // 重写以从多个数据源获取信息
  }
}
```

在这种情况下，TypeScript 不会发出任何警告，提示 `SharedAlbum` 上的 `download` 原本是*期望*重写基类中的某个函数的。

使用 `noImplicitOverride`，你可以要求重写基类的方法必须包含 `override` 关键字，从而确保子类永远不会与基类脱节。

以下示例启用了 `noImplicitOverride`，你可以看到在缺少 `override` 时出现的错误提示：

```ts twoslash
// @noImplicitOverride
// @errors: 4114
class Album {
  setup() {}
}

class MLAlbum extends Album {
  override setup() {}
}

class SharedAlbum extends Album {
  setup() {}
}
```
