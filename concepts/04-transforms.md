# 转换

> Commit ID: [5d3eccf26279d6e4ba0eeecc754d8894c6061dbe](https://github.com/ianstormtaylor/slate/blob/main/docs/concepts/04-transforms.md)

Slate 的数据结构不可变，所以不能直接修改或者删除节点。不过，Slate 带有一组“转换”函数用于转换编辑器的值。

Slate 的转换函数设计为非常灵活，可以对编辑器进行各种更改。然而，这种灵活可能刚开始难以理解。

通常，会将单个操作用于单个或者多个节点。例如，通过对每个块级元素的父级应用 `unwrapNodes` 来展平语法树：

```js
Transforms.unwrapNodes(editor, {
  at: [], // 编辑器的路径
  match: node =>
    !Editor.isEditor(node) &&
    node.children?.every(child => Editor.isBlock(editor, child)),
  mode: 'all', // 还有编辑器的子元素
})
```

非标准操作（或者调试/追踪哪些节点会受到 NodeOptions 的影响）可能需要使用 `Editor.nodes` 创建 
NodeEntries JavaScript 迭代器且用 for..of 执行。
例如，用对应的 alt 文本替换所有图像元素：

```js
const imageElmnts = Editor.nodes(editor, {
  at: [], // 编辑器的路径
  match: (node, path) => 'image' === node.type,
  // 模式默认是 “all”，所以也可以搜索编辑器的子元素
})
for (const nodeEntry of imageElmnts) {
  const altText =
    nodeEntry[0].alt ||
    nodeEntry[0].title ||
    /\/([^/]+)$/.exec(nodeEntry[0].url)?.[1] ||
    '☹︎'
  Transforms.select(editor, nodeEntry[1])
  Editor.insertFragment(editor, [{ text: altText }])
}
```

> 🤖 查看[转换](../api/transforms.md) 参考获取 Slate 转换的完整列表。

## 选区转换

选区关联转换是一些更简单的转换。例如，以下是将选区设置为新范围的方法：

```js
Transforms.select(editor, {
  anchor: { path: [0, 0], offset: 0 },
  focus: { path: [1, 0], offset: 2 },
})
```

但也可以更复杂。

例如，通常需要将光标按照字符，单词，行来向前/后移动不同的距离。以下是如何将光标向后移动三个单词：

```js
Transforms.move(editor, {
  distance: 3,
  unit: 'word',
  reverse: true,
})
```

> 🤖 请查看[选区转换 API 参考](../api/transforms.md#selection-transforms)获取更多信息

## 文本转换

文本转换用于编辑器的文本内容。例如，以下是在指定位置插入文本字符串的方法：

```js
Transforms.insertText(editor, 'some words', {
  at: { path: [0, 0], offset: 3 },
})
```

或者从编辑器中删除整个范围内的所有内容：

```js
Transforms.delete(editor, {
  at: {
    anchor: { path: [0, 0], offset: 0 },
    focus: { path: [1, 0], offset: 2 },
  },
})
```

> 🤖 请查看[文本转换 API 参考](../api/transforms.md#text-transforms)获取更多信息

## 节点转换

节点转换作用于构成编辑器值的单个元素和文本节点。例如，在指定位置插入新文本节点：

```js
Transforms.insertNodes(
  editor,
  {
    text: 'A new string of text.',
  },
  {
    at: [0, 1],
  }
)
```

或者将节点从该路径移动到其它路径：

```js
Transforms.moveNodes(editor, {
  at: [0, 0],
  to: [0, 1],
})
```

> 🤖 请查看[节点转换 API 参考](../api/transforms.md#node-transforms)获取更多信息

## `at` 选项

许多转换作用于文档的指定位置。默认情况下使用用户的当前选区。但是可以使用 `at` 选项覆盖。

例如在插入文本时，会在用户的当前光标处插入字符串：

```js
Transforms.insertText(editor, 'some words')
```

而这将会在指定位置插入：

```js
Transforms.insertText(editor, 'some words', {
  at: { path: [0, 0], offset: 3 },
})
```

`at` 选项的用途非常广泛，很容易的用于实现更复杂的装换。因为它是 `Location` 所以它可以是 `Path`、`Point`、`Range`。每种类型的位置都会导致略有不同的转换。

例如，在插入文本时，如果指定 `Range` 位置，首先删除范围，在插入文本的位置折叠为单个点。

所以要用新字符串替换文本，可以执行如下操作：

```js
Transforms.insertText(editor, 'some words', {
  at: {
    anchor: { path: [0, 0], offset: 0 },
    focus: { path: [0, 0], offset: 3 },
  },
})
```

如果指定 `Path` 位置，将会扩大到范围，覆盖该路径的整个节点。然后，使用基于范围的行为，删除整个节点的内容且使用文本替换。

所以使用新字符串替换整个节点的文本，可以执行如下操作：

```js
Transforms.insertText(editor, 'some words', {
  at: [0, 0],
})
```

基于位置的行为适用于所有采用 `at` 选项的转换。可能刚开始很难理解，但它让 API 更强大且能够表达细微差别的转换。

## `match` 选项

许多基于节点的转换采用 `match` 函数选项，限制转换仅应用于函数返回 `true` 的节点。当跟 `at` 一起使用时， `match` 也会变得非常强大。

例如，考虑一个基础的转换，将节点从一个路径移动到另一个路径：

```js
Transforms.moveNodes(editor, {
  at: [2],
  to: [5],
})
```

虽然看起来就像是将路径移动到另一个地方。在底层发生了两件事。。。

首先， `at` 选项扩展为范围代表 `[2]` 节点内的所有内容。看起来就像是：

```js
at: {
  anchor: { path: [2, 0], offset: 0 },
  focus: { path: [2, 2], offset: 19 }
}
```

其次，`match` 选项默认仅为匹配指定路径的函数，在这里是 `[2]`：

```js
match: (node, path) => Path.equals(path, [2])
```

然后 Slate 遍历范围，移动所有通过了匹配器函数的节点到新位置。在本例中，因为 `match` 默认仅匹配精确的 `[2]` 路径，因此该节点被移动。

但是想要移动 `[2]` 的子节点呢？

可能会循环子节点并依次移动，但会变得非常复杂，因为当移动节点时，所引用的路径也会过时。

不过，可以使用 `at` 和 `match` 选项来匹配所有子节点：

```js
Transforms.moveNodes(editor, {
  // 这将会扩展到 `[2]` 的整个节点的范围。
  at: [2],
  // 匹配路径较长的节点，即子节点。
  match: (node, path) => path.length === 2,
  to: [5],
})
```

这是我们使用相同的 `at` 路径（扩展到范围），但是默认情况下不仅匹配路径，而是提供恰好匹配指定子节点的 `match` 函数。

使用 `match` 可以使复杂的逻辑变得很简单。

例如，考虑为任何不是斜体的文本节点添加粗体标记：

```js
Transform.setNodes(
  editor,
  { bold: true },
  {
    // 路径引用编辑器，且扩展为
    // 包含编辑器所有内容的范围。
    at: [],
    // 仅匹配不是斜体的文本节点。
    match: (node, path) => Text.isText(node) && node.italic !== true,
  }
)
```

当执行转换时，如果曾经遍历节点且依次转换，请考虑 `match` 是否可以解决该用例，并将管理循环的复杂性转移给 Slate。
`match` 函数可以使用 `node.children` 检查子节点或使用 `Node.parent` 检查父节点。

## 转换和标准化

如果节点树_不_应该在转换中标准化，则可能需要将转换序列使用 [`Editor.withoutNormalizing`](../api/nodes/editor.md#editorwithoutnormalizingeditor-editor-fn---void--void) 包裹起来。参阅 [标准化 —— 对其它代码的影响](./11-normalizing.md#implications-for-other-code)；
