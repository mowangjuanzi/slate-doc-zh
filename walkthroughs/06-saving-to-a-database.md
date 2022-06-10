# 保存到数据库

> Commit ID: [acdc5c3417e30ec50ff2499394205246ccb70ca7](https://github.com/ianstormtaylor/slate/blob/main/docs/walkthroughs/06-saving-to-a-database.md)

现在已经学习了如何向 Slate 编辑器添加功能的基础知识，那如何保存正在编辑的内容，以便稍后回到应用程序后还能继续加载呢。

在本教程中，将展示如何添加逻辑以便将 Slate 内容保存到数据库中存储并以后可以检索。

先从基础的编辑器开始：

```jsx
const initialValue = [
  {
    type: 'paragraph',
    children: [{ text: 'A line of text in a paragraph.' }],
  },
]

const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])

  return (
    <Slate editor={editor} value={initialValue}>
      <Editable />
    </Slate>
  )
}
```

在页面上会渲染一个基础的编辑器且当输入内容时会发生改变。但是如果刷新页面，一切都会恢复到最开始的样子 —— 没有保存任何内容！

这里需要做的是将更改保存到一个地方。在本示例中，只会使用[Local Storage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)（[本地存储](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/localStorage)），但是会了解需要在哪里添加数据库钩子。

所以在 `onChange` 处理程序中需要添加在除了 `set_selection` 之外的内容更改时保存 `value`：

```jsx
const initialValue = [
  {
    type: 'paragraph',
    children: [{ text: 'A line of text in a paragraph.' }],
  },
]

const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])

  return (
    <Slate
      editor={editor}
      value={initialValue}
      onChange={value => {
        const isAstChange = editor.operations.some(
          op => 'set_selection' !== op.type
        )
        if (isAstChange) {
          // 保存值到本地存储。
          const content = JSON.stringify(value)
          localStorage.setItem('content', content)
        }
      }}
    >
      <Editable />
    </Slate>
  )
}
```

现在每当编辑页面，如果查看本地存储，应该会看到 `content` 值发生变化。

但。。。如果刷新页面，一切都会重置。这是因为需要确保从同一个本地存储位置拉取初始值，如下所示：

```jsx
const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  // 如果存在，则从本地存储拉取初始化内容。
  const initialValue = useMemo(
    () =>
      JSON.parse(localStorage.getItem('content')) || [
        {
          type: 'paragraph',
          children: [{ text: 'A line of text in a paragraph.' }],
        },
      ],
    []
  )

  return (
    <Slate
      editor={editor}
      value={initialValue}
      onChange={value => {
        const isAstChange = editor.operations.some(
          op => 'set_selection' !== op.type
        )
        if (isAstChange) {
          // 保存值到本地存储。
          const content = JSON.stringify(value)
          localStorage.setItem('content', content)
        }
      }}
    >
      <Editable />
    </Slate>
  )
}
```

现在应该在刷新时保存更改了！

成功 —— 在数据库中已包含 JSON。

但是如果你想要 JSON 意外的东西呢？是的，需要用不同的方式序列化值。例如将内容保存为纯文本而不是 JSON，可以编写一些序列化和反序列化纯文本值：

```jsx
// 从 Slate 中导入 `Node` 助手接口。
import { Node } from 'slate'

// 定义接受值并返回字符串的序列化函数。
const serialize = value => {
  return (
    value
      // 在值的子项中返回每个段落的字符串内容。
      .map(n => Node.string(n))
      // 用表示段落的换行符将其串联起来。
      .join('\n')
  )
}

// 定义接受字符串返回值的反序列化函数。
const deserialize = string => {
  // 通过拆分字符串提取子级值数组。
  return string.split('\n').map(line => {
    return {
      children: [{ text: line }],
    }
  })
}

const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  // 使用反序列化函数从本地存储中读取数据。
  const initialValue = useMemo(
    deserialize(localStorage.getItem('content')) || '',
    []
  )

  return (
    <Slate
      editor={editor}
      value={initialValue}
      onChange={value => {
        const isAstChange = editor.operations.some(
          op => 'set_selection' !== op.type
        )
        if (isAstChange) {
          // 序列化值并将字符串保存到本地存储。
          localStorage.setItem('content', serialize(value))
        }
      }}
    >
      <Editable />
    </Slate>
  )
}
```

棒！现在正在使用纯文本。

可以对喜欢的格式模仿此策略。可以序列化为 HTML、 Markdown 或者为用例量身定制的自定义 JSON 格式。

> 🤖 注意，即使_可以_按照个人喜好序列化内容，也要权衡取舍。序列化过程有成本，某些格式可能会比其他格式更难处理。一般来说，仅建议用例有特殊需求时才编写自己的格式。否则，最好将数据保存为 Slate 使用的格式。

如果你想更新编辑器的内容以响应 Slate 之外的事件，需要直接修改 children 属性。最简单的方式是替换 editor.children 的值 `editor.children = newValue` 并触发重新渲染（例如，在上面的示例中调用 `editor.onChange()`）。或者，可以使用 Slate 内部操作来转换值，例如：

```javascript
  /**
  * resetNodes 重置编辑器的值。
  * 需要注意的是，传递 `at` 参数可能会导致 “Cannot resolve a DOM point from Slate point” 错误。
  */
  resetNodes<T extends Node>(
    editor: Editor,
    options: {
      nodes?: Node | Node[],
      at?: Location
    } = {}
  ): void {
    const children = [...editor.children]

    children.forEach((node) => editor.apply({ type: 'remove_node', path: [0], node }))

    if (options.nodes) {
      const nodes = Node.isNode(options.nodes) ? [options.nodes] : options.nodes

      nodes.forEach((node, i) => editor.apply({ type: 'insert_node', path: [i], node: node }))
    }

    const point = options.at && Point.isPoint(options.at)
      ? options.at
      : Editor.end(editor, [])

    if (point) {
      Transforms.select(editor, point)
    }
  }
```
