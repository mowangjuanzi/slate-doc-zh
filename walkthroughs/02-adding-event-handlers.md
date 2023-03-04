# 添加事件处理程序

> Commit ID: [22308b34172e946d88405cbb6338273dbda5dff4](https://github.com/ianstormtaylor/slate/blob/main/docs/walkthroughs/02-adding-event-handlers.md)

好，已经安装了 Slate 并在页面渲染，当输入时，会看到变化。但是需要做的不仅仅是输入纯文本字符串。

Slate 之所以如此出色是因为它易于定制。就像使用其它 React 组件一样，当某些事件触发的时候，Slate 允许传递处理程序。

当按下一个键的时候，使用 `onKeyDown` 处理程序来更改编辑器的内容。

这是之前的应用程序：

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
      <Editable />
    </Slate>
  )
}
```

现在添加 `onKeyDown` 处理程序：

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
        // 定义新的处理程序打印按下的键。
        onKeyDown={event => {
          console.log(event.key)
        }}
      />
    </Slate>
  )
}
```

棒，现在当在编辑器中按下一个键时，其对应的建码会纪录在控制台中。

现在想让它真正的转变内容。出于示例的目的，实现为所有按下的 `&` 键转换为单词 `and`。

`onKeyDown` 处理程序可能如下所示：

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
            // 防止插入 & 字符。
            event.preventDefault()
            // 事件发生时执行 `insertText` 方法。
            editor.insertText('and')
          }
        }}
      />
    </Slate>
  )
}
```

添加后，尝试输入 `&`，会发现它突然变成了 `and`！

这提示了使用 Slate 事件处理程序可以做什么的意义。每个都将调用 `event` 对象，你可以使用 `editor` 执行命令作为响应。是不是挺简单！
