# 编辑器

> Commit ID: [20acca4bc8f31bd1aa6fbca2c49aaae5f31cadfe](https://github.com/ianstormtaylor/slate/blob/main/docs/concepts/07-editor.md)

Slate 编辑器的所有行为、内容、状态都汇总到单个顶级 `Editor` 对象。接口是：

```typescript
interface Editor {
  // 当前编辑器状态
  children: Node[]
  selection: Range | null
  operations: Operation[]
  marks: Omit<Text, 'text'> | null
  // 特定模式的节点行为。
  isInline: (element: Element) => boolean
  isVoid: (element: Element) => boolean
  normalizeNode: (entry: NodeEntry) => void
  onChange: () => void
  // 可重写的核心操作。
  addMark: (key: string, value: any) => void
  apply: (operation: Operation) => void
  deleteBackward: (unit: 'character' | 'word' | 'line' | 'block') => void
  deleteForward: (unit: 'character' | 'word' | 'line' | 'block') => void
  deleteFragment: () => void
  insertBreak: () => void
  insertSoftBreak: () => void
  insertFragment: (fragment: Node[]) => void
  insertNode: (node: Node) => void
  insertText: (text: string) => void
  removeMark: (key: string) => void
}
```

它比其它的稍微复杂一些，因为包含所有的顶级函数，比如定义的自定义函数以及特定领域的行为。

`children` 属性包含构成编辑器内容的节点的文档树。

`selection` 属性包含用户当前的选区（如果存在）。不要直接设置，使用 [Transforms.select](04-transforms#selection-transforms)。

`operations` 属性包含了自上次“更改”刷新以来已应用的所有操作（因为 Slate 将批次操作加入到事件循环的 tick 中。）

`marks` 属性保存了当编辑器插入文本时要应用的格式，如果 `marks` 是 `null`，格式从当前选区获取。
不能直接赋值，使用 `Editor.addMark` 和 `Editor.removeMark` 设置该值。

## 重写行为

在之前的教程中，已经暗示过，通过重写它的函数属性来重写编辑器的任何行为。

例如，如果你想在行内节点定义链接元素：

```javascript
const { isInline } = editor

editor.isInline = element => {
  return element.type === 'link' ? true : isInline(element)
}
```

或者想重写 `insertText` 行为来“链接” URL：

```javascript
const { insertText } = editor

editor.insertText = text => {
  if (isUrl(text)) {
    // ...
    return
  }

  insertText(text)
}
```

或者甚至想要定义自定义“标准化”，确保链接遵从某些约束：

```javascript
const { normalizeNode } = editor

editor.normalizeNode = entry => {
  const [node, path] = entry

  if (Element.isElement(node) && node.type === 'link') {
    // ...
    return
  }

  normalizeNode(entry)
}
```

每当真的要覆盖行为时，请务必调用现有函数作为默认行为的后备机制。除非真的想要完全删除默认行为（这并不是好主意）。

> 🤖 请查看 [编辑器实例化方法覆盖 API 参考](../api/nodes/editor.md#schema-specific-instance-methods-to-override)获取更多信息

## 助手函数

与所有 Slate 接口一样， `Editor` 接口公开了在实现某些行为时有用的助手函数。有很多编辑器相关的助手。例如：

```javascript
// 在路径中获取指定节点的起点。
const point = Editor.start(editor, [0, 0])

// 在范围中获取片段（文档的一部分）。
const fragment = Editor.fragment(editor, range)
```

还有很多基于迭代器的助手，例如：

```javascript
// 遍历范围内的每个节点。
for (const [node, path] of Editor.nodes(editor, { at: range })) {
  // ...
}

// 迭代当前选区文本节点中的每个点。
for (const point of Editor.positions(editor)) {
  // ...
}
```

> 🤖 查看[编辑器静态方法 API 参考](../api/nodes/editor.md#static-methods)获取更多信息
