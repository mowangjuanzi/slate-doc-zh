# 插件

> Commit ID: [f183bde599133e1e6ce3549e1f3055e936246b8e](https://github.com/ianstormtaylor/slate/blob/main/docs/concepts/08-plugins.md)

已经了解如何重写 Slate 编辑器的行为。这些重写可以打包为“插件”，以便复用，测试和共享。这是 Slate 架构中最强大的方面之一。

插件仅仅是函数，接受一个 `Editor` 对象，以某种方式增强并返回。

例如，标记图像节点为 “void”：

```javascript
const withImages = editor => {
  const { isVoid } = editor

  editor.isVoid = element => {
    return element.type === 'image' ? true : isVoid(element)
  }

  return editor
}
```

然后使用插件，只需：

```javascript
import { createEditor } from 'slate'

const editor = withImages(createEditor())
```

插件组合模型使得 Slate 非常容易扩展！

## 助手函数

除了插件函数之外，还想公开与插件一起使用的助手函数。例如：

```javascript
import { Editor, Element } from 'slate'

const MyEditor = {
  ...Editor,
  insertImage(editor, url) {
    const element = { type: 'image', url, children: [{ text: '' }] }
    Transforms.insertNodes(editor, element)
  },
}

const MyElement = {
  ...Element,
  isImageElement(value) {
    return Element.isElement(element) && element.type === 'image'
  },
}
```

然后就可以在任何地方使用 `MyEditor` 和 `MyElement`，并且在一个地方访问所有助手。
