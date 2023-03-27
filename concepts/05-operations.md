# 操作

操作是调用转换时发生的细粒度，低层次操作。单个转换可能会导致多个低层次操作应用于编辑器。

Slate 的核心定义了所有可能可以在富文本文档上发生的操作。例如：

```javascript
editor.apply({
  type: 'insert_text',
  path: [0, 0],
  offset: 15,
  text: 'A new string of text to be inserted.',
})

editor.apply({
  type: 'remove_node',
  path: [0, 0],
  node: {
    text: 'A line of text!',
  },
})

editor.apply({
  type: 'set_selection',
  properties: {
    anchor: { path: [0, 0], offset: 0 },
  },
  newProperties: {
    anchor: { path: [0, 0], offset: 15 },
  },
})
```

在底层，Slate 将复杂的转换变为低级操作并自动应用于编辑器中。因此，除非正在实现协同编辑，否则很少需要考虑操作。

> 🤖 Slate 的编辑行为定义为操作，这使得协同编辑称为可能，因为每个更改都可以很轻易的定义，应用，组合甚至撤销！
