# 命令

在编辑富文本内容时，用户会执行插入文本，删除文本，拆分段落，添加格式等操作，在底层，这些编辑使用转换和操作来表达。但是在高层级上，称之为“命令”。

命令是代表用户特定意图的高级操作。在 `Editor` 接口上表示为助手函数。 少量常用富文本行为助手包含在核心中，但鼓励编写自己的助手来为特定域建模。

例如，以下是一些内置命令：

```javascript
Editor.insertText(editor, 'A new string of text to be inserted.')

Editor.deleteBackward(editor, { unit: 'word' })

Editor.insertBreak(editor)
```

但是可以（将会）定义自定义命令来对域建模。例如，对允许的内容类型定义 `formatQuote` 命令或者`insertImage` 命令或者 `toggleBold` 命令。

命令总是要描述要采取的行为，就像**用户**正在执行该操作一样。因此不需要定义执行命令的位置，因为总是根据用户的当前选区操作。

> 🤖 命令的概念大致上是基于 DOM 内置的 [`execCommand`](https://developer.mozilla.org/en-US/docs/Web/API/Document/execCommand) API。然后 Slate 定义的 API 更简单（可扩展），因为 DOM 的版本太过固执且前后不一。

在底层，Slate 负责将每个命令转化为一组低级“操作”，用于产生新值。这就是能让协作编辑称为可能的原因。但是也不需要担心，因为它是自动发生的。

## 自定义命令

当穿件自定义命令时，可以创建自己的命名空间：

```javascript
const MyEditor = {
  ...Editor,

  insertParagraph(editor) {
    // ...
  },
}
```

当编写命令时，会经常使用 Slate 的 `Transforms` 助手。

## 转换

转换是一组特定的助手，允许对文档执行各种特定的更改，例如：

```javascript
// 在范围内的所有文本节点设置“粗体”格式。
Transforms.setNodes(
  editor,
  { bold: true },
  {
    at: range,
    match: node => Text.isText(node),
    split: true,
  }
)

// Wrap the lowest block at a point in the document in a quote block.
Transforms.wrapNodes(
  editor,
  { type: 'quote', children: [] },
  {
    at: point,
    match: node => Editor.isBlock(editor, node),
    mode: 'lowest',
  }
)

// 在指定路径插入新文本以替换节点中的文本。
Transforms.insertText(editor, 'A new string of text.', { at: path })

// ...还有更多的转换！
```

转换助手设计为组合在一起。所以可以回每个命令使用其中的几个。
