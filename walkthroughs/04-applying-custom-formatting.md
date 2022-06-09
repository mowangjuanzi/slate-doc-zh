# 应用自定义格式

之前的教程中，学习了如何创建自定义块类型，以在不同的容器中渲染文本块。但 Slate 允许的不仅仅只是“块”。

在本教程中，将会展示如何添加自定义格式选项，例如 **粗体**, _斜体_, `代码` 或者 ~~删除线~~.

从之前的应用开始：

```jsx
const renderElement = props => {
  switch (props.element.type) {
    case 'code':
      return <CodeElement {...props} />
    default:
      return <DefaultElement {...props} />
  }
}

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
      <Editable
        renderElement={renderElement}
        onKeyDown={event => {
          if (event.key === '`' && event.ctrlKey) {
            event.preventDefault()
            const [match] = Editor.nodes(editor, {
              match: n => n.type === 'code',
            })
            Transforms.setNodes(
              editor,
              { type: match ? 'paragraph' : 'code' },
              { match: n => Editor.isBlock(editor, n) }
            )
          }
        }}
      />
    </Slate>
  )
}
```

现在将编辑 `onKeyDown` 处理程序，以便当按下 <kbd>Ctrl</kbd> + <kbd>B</kbd> 时，将当前已选的文本添加粗体格式：

```jsx
const initialValue = [
  {
    type: 'paragraph',
    children: [{ text: 'A line of text in a paragraph.' }],
  },
]

const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])

  const renderElement = useCallback(props => {
    switch (props.element.type) {
      case 'code':
        return <CodeElement {...props} />
      default:
        return <DefaultElement {...props} />
    }
  }, [])

  return (
    <Slate editor={editor} value={initialValue}>
      <Editable
        renderElement={renderElement}
        onKeyDown={event => {
          if (!event.ctrlKey) {
            return
          }

          switch (event.key) {
            // 当按下 “`” 时，保留现有的代码块逻辑。
            case '`': {
              event.preventDefault()
              const [match] = Editor.nodes(editor, {
                match: n => n.type === 'code',
              })
              Transforms.setNodes(
                editor,
                { type: match ? 'paragraph' : 'code' },
                { match: n => Editor.isBlock(editor, n) }
              )
              break
            }

            // 当按下 “B” 时，所选内容进行文本加粗。
            case 'b': {
              event.preventDefault()
              Transforms.setNodes(
                editor,
                { bold: true },
                // 应用于文本节点，如果仅选择文本
                // 节点的一部分，则会将文本节点风采。
                { match: n => Text.isText(n), split: true }
              )
              break
            }
          }
        }}
      />
    </Slate>
  )
}
```

好，我们已经设置了快捷键处理程序。。。但是！如果碰巧尝试选择文本并按 <kbd>Ctrl</kbd> + <kbd>B</kbd>，将会不会注意到任何变化。因为 Slate 还不知道如何渲染“粗体”标记。

对于添加的每种格式，Slate 会将文本内容分解为“叶子”，Slate 需要知道如何像元素一样阅读它。所以需要定义 `Leaf` 组件：

```jsx
// 定义 React 组件渲染带有粗体文本的叶子。
const Leaf = props => {
  return (
    <span
      {...props.attributes}
      style={{ fontWeight: props.leaf.bold ? 'bold' : 'normal' }}
    >
      {props.children}
    </span>
  )
}
```

是不是很熟悉？

现在，需要告诉 Slate 叶子的事。需要在 prop 中传入 `renderLeaf` 到编辑器。

```jsx
const initialValue = [
  {
    type: 'paragraph',
    children: [{ text: 'A line of text in a paragraph.' }],
  },
]

const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])

  const renderElement = useCallback(props => {
    switch (props.element.type) {
      case 'code':
        return <CodeElement {...props} />
      default:
        return <DefaultElement {...props} />
    }
  }, [])

  // 定义使用 `useCallback` 记住的叶子渲染函数。
  const renderLeaf = useCallback(props => {
    return <Leaf {...props} />
  }, [])

  return (
    <Slate editor={editor} value={initialValue}>
      <Editable
        renderElement={renderElement}
        // 传入 `renderLeaf` 函数。
        renderLeaf={renderLeaf}
        onKeyDown={event => {
          if (!event.ctrlKey) {
            return
          }

          switch (event.key) {
            case '`': {
              event.preventDefault()
              const [match] = Editor.nodes(editor, {
                match: n => n.type === 'code',
              })
              Transforms.setNodes(
                editor,
                { type: match ? null : 'code' },
                { match: n => Editor.isBlock(editor, n) }
              )
              break
            }

            case 'b': {
              event.preventDefault()
              Transforms.setNodes(
                editor,
                { bold: true },
                { match: n => Text.isText(n), split: true }
              )
              break
            }
          }
        }}
      />
    </Slate>
  )
}

const Leaf = props => {
  return (
    <span
      {...props.attributes}
      style={{ fontWeight: props.leaf.bold ? 'bold' : 'normal' }}
    >
      {props.children}
    </span>
  )
}
```

现在，如果尝试选择一段文本并按下 <kbd>Ctrl</kbd> + <kbd>B</kbd> 会看到变成了粗体！神奇吧！
