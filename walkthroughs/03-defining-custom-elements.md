# 定义自定义元素

> Commit ID: [d1f90ebd12ee2da855911471648136e674efc813](https://github.com/ianstormtaylor/slate/blob/main/docs/walkthroughs/03-defining-custom-elements.md)

在之前的示例中，在之前的段落中，但是并未真正告诉过 Slate 任何关于段落块类型的信息。只是让其使用内部默认渲染器，一个普通的老式 `<div>`。

但是能做的不仅仅是这些。Slate 允许定义任何自定义块类型，像引用、代码块、列表项等等。

下面会介绍如何做。先从之前的应用开始吧：

```jsx
const initialValue = [
  {
    type: 'paragraph',
    children: [{ text: 'A line of text in a paragraph.' }],
  },
]

const App = () => {
  const [editor] = useState(() => withReact(createEditor()))

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

现在添加 “代码块” 到编辑器中。

有个问题，就是代码块不能只使用普通的段落去渲染，还需要以不同的方式渲染。为了实现这一点，需要为 `code` 元素节点定义个“渲染器”。

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

是不是很容易。

看到 `props.attributes` 引用了吗？Slate 传递应该在块的最上层元素上渲染的属性，所以不必自己构建它们。**必须**将属性组合到组件中。

看到 `props.children` 引用了吧? Slate 会自动渲染块下面的所有子元素，然后像是其它 React 组件一样通过 `props.children` 传递。这样就不必费心渲染正确的文本节点或者类似的东西。**必须**将组件中的子元素作为最低叶子进行渲染。

这是 “default” 元素组件：

```jsx
const DefaultElement = props => {
  return <p {...props.attributes}>{props.children}</p>
}
```

现在，添加渲染器到 `Editor`：

```jsx
const initialValue = [
  {
    type: 'paragraph',
    children: [{ text: 'A line of text in a paragraph.' }],
  },
]

const App = () => {
  const [editor] = useState(() => withReact(createEditor()))

  // 基于传递给 `props` 的元素定义渲染函数。
  // 在这里使用 `useCallback` 记住函数以供后续渲染。
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

好，但现在需要一种方式让用户真正的将块转化为代码块。所以改变 `onKeyDown` 函数，添加 `` Ctrl-` `` 快捷键来做这件事：

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
  const [editor] = useState(() => withReact(createEditor()))

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

现在，如果按下 `` Ctrl-` ``，现在光标所在的块会变成代码块！神奇吧！

但是忘记了一件事。就是当再次点击 `` Ctrl-` `` 时，应该将代码块改回段落。为此，需要添加逻辑来根据当前选择的块是否是代码块来更改设置的类型：

```jsx
const initialValue = [
  {
    type: 'paragraph',
    children: [{ text: 'A line of text in a paragraph.' }],
  },
]

const App = () => {
  const [editor] = useState(() => withReact(createEditor()))

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

这很简单！如果代码块中按 `` Ctrl-` ``，应该要变回段落！
