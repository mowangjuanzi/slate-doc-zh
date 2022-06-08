# 定义自定义元素

> Commit ID: [9892cf0ffbd741cc2880d1f0bd0d7c1b36145bbd](https://github.com/ianstormtaylor/slate/blob/main/docs/walkthroughs/03-defining-custom-elements.md)

在之前的示例中，我从段落开始，但是我们并未真正告诉过 Slate 任何关于段落块类型的信息。我们只是让他使用它的内部默认渲染器，一个普通的老式 `<div>`。

但这不是你全部能做的。Slate 允许定义任何自定义块类型，像引用、代码块、列表项等等。

我们会告诉你怎么做。让我们从之前的应用开始：

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
      <Editable
        onKeyDown={event => {
          if (event.key === '&') {
            event.preventDefault()
            editor.insertText('and')
          }
        }}
      />
    </Slate>
  )
}
```

现在我们添加 “代码块” 到编辑器中。

有个问题，就是代码块不能仅仅使用普通的段落去渲染，而且他们还需要以不同的方式渲染。为了实现这一点，需要为 `code` 元素节点定义个“渲染器”。

元素渲染器只是简单的 React 组件，如下所示：

```jsx
// 为代码块定义 React 组件渲染器。
const CodeElement = props => {
  return (
    <pre {...props.attributes}>
      <code>{props.children}</code>
    </pre>
  )
}
```

很容易吧。

看到 `props.attributes` 引用了吗？Slate 传递应该在块的最上层元素上渲染的属性，所以你不必自己构建它们。你**必须**组合属性到组件中。

看到 `props.children` 引用了吧? Slate 会自动渲染块下面的所有子元素，然后像是其它 React 组件一样通过 `props.children` 传递。这样就不必费心渲染正确的文本节点或者类似的东西。你**必须**将组件中的子元素作为最低叶子进行渲染。

这是 “default” 元素组件：

```jsx
const DefaultElement = props => {
  return <p {...props.attributes}>{props.children}</p>
}
```

现在，让我们添加渲染器到 `Editor`：

```jsx
const initialValue = [
  {
    type: 'paragraph',
    children: [{ text: 'A line of text in a paragraph.' }],
  },
]

const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])

  // 基于传递给 `props` 的元素定义渲染函数。 We use
  // 我们在这里使用 `useCallback` 记住函数以供后续渲染。
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
        // 传入 `renderElement` 函数。
        renderElement={renderElement}
        onKeyDown={event => {
          if (event.key === '&') {
            event.preventDefault()
            editor.insertText('and')
          }
        }}
      />
    </Slate>
  )
}

const CodeElement = props => {
  return (
    <pre {...props.attributes}>
      <code>{props.children}</code>
    </pre>
  )
}

const DefaultElement = props => {
  return <p {...props.attributes}>{props.children}</p>
}
```

好，但现在需要一种方式让用户真正的将块转化为代码块。所以改变 `onKeyDown` 函数，添加 <kbd>Ctrl</kbd> + <kbd>`</kbd> 快捷键来做这件事：

```jsx
// 从 Slate 导入 `Editor` 和 `Transforms` 助手。
import { Editor, Transforms } from 'slate'

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
          if (event.key === '`' && event.ctrlKey) {
            // 默认防止插入 “`”。
            event.preventDefault()
            // 否则，将当前已选块类型设置为 “code”。
            Transforms.setNodes(
              editor,
              { type: 'code' },
              { match: n => Editor.isBlock(editor, n) }
            )
          }
        }}
      />
    </Slate>
  )
}

const CodeElement = props => {
  return (
    <pre {...props.attributes}>
      <code>{props.children}</code>
    </pre>
  )
}

const DefaultElement = props => {
  return <p {...props.attributes}>{props.children}</p>
}
```

现在，如果按下 <kbd>Ctrl</kbd> + <kbd>`</kbd>，现在光标所在的块会变成代码块！神奇！

但是我们忘记了一件事。当再次点击 <kbd>Ctrl</kbd> + <kbd>`</kbd> 时，应该将代码块改回段落。为此，需要添加逻辑来根据当前选择的块时候是代码块来更改我们设置的类型：

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
          if (event.key === '`' && event.ctrlKey) {
            event.preventDefault()
            // 确认当前已选的块是代码块。
            const [match] = Editor.nodes(editor, {
              match: n => n.type === 'code',
            })
            // 根据时候已经匹配切换块类型。
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

明白了吧！如果你在代码块中按 <kbd>Ctrl</kbd> + <kbd>`</kbd>，应该要变回段落！
