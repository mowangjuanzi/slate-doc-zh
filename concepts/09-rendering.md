# 渲染

> Commit ID: [f183bde599133e1e6ce3549e1f3055e936246b8e](https://github.com/ianstormtaylor/slate/blob/main/docs/concepts/08-rendering.md)

Slate 使用 React 构建是最好的部分，因此适用现有的应用程序。不会重新发明需要学习的视图层。尽可能的保持 React。

为此，Slate 允许控制富文本域中自定义节点和属性的渲染行为。

可以通过将“渲染 props” 传递给顶级 `<Editable>` 组件定义行为。

例如，想要渲染自定义元素组件，可以传递 `renderElement` prop：

```jsx
import { createEditor } from 'slate'
import { Slate, Editable, withReact } from 'slate-react'

const MyEditor = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  const renderElement = useCallback(({ attributes, children, element }) => {
    switch (element.type) {
      case 'quote':
        return <blockquote {...attributes}>{children}</blockquote>
      case 'link':
        return (
          <a {...attributes} href={element.url}>
            {children}
          </a>
        )
      default:
        return <p {...attributes}>{children}</p>
    }
  }, [])

  return (
    <Slate editor={editor}>
      <Editable renderElement={renderElement} />
    </Slate>
  )
}
```

> 🤖 请务必在自定义组件中同时使用 `props.attributes` 和 `props.children` 渲染！attributes 必须添加到组件内的顶级 DOM 元素，因为这是 Slate DOM 助手函数运行所必需的。children 是 Slate 自动管理的文档的实际文本内容。

不必使用简单的 HTML 元素，也可以使用自定义 React 组件：

```javascript
const renderElement = useCallback(props => {
  switch (props.element.type) {
    case 'quote':
      return <QuoteElement {...props} />
    case 'link':
      return <LinkElement {...props} />
    default:
      return <DefaultElement {...props} />
  }
}, [])
```

## 叶子

当渲染文本级别格式时，字符被分为文本的“叶子”，每个文本都包含相同的格式。

要自定义每个叶子的渲染，使用自定义 `renderLeaf` prop：

```jsx
const renderLeaf = useCallback(({ attributes, children, leaf }) => {
  return (
    <span
      {...attributes}
      style={{
        fontWeight: leaf.bold ? 'bold' : 'normal',
        fontStyle: leaf.italic ? 'italic' : 'normal',
      }}
    >
      {children}
    </span>
  )
}, [])
```

请注意，尽管我们处理的方式跟 `renderElement` 略有不同。由于文本格式往往非常简单，所以选择放弃 `switch` 语句而只开关一些样式。（但是如果愿意的话，没有什么能阻拦使用自定义组件）。

文本级别格式的一个缺点就是不能保证任何指定的格式都是“连续的” —— 意味着将保持单个叶子。这种叶子限制类似 DOM，但这是无效的：

```html
<em>t<strong>e</em>x</strong>t
```

因为上面的元素没有正确的关闭，所以无效，但是可以按照如下方式编写上述 HTML：

```html
<em>t</em><strong><em>e</em>x</strong>t
```

如果碰巧在文本中添加了另外一个 `<strike>` 重叠部分，则必须重新排列结束标记。在 Slate 中渲染叶子是相似的 —— 不能保证即使一个单词应用了一个格式，叶子还是连续的，因为这取决于与其它格式重叠。

当然，叶子这个东西听起来很复杂。但是不必考虑太多，只要将文本级别格式与元素级别格式用于其预期目的即可：

- 文本属性用于**不连续的**的字符级格式。
- 元素属性用于文档中**连续的**语义元素。

## 装饰

装饰是另外一种文本级格式。类似于常规的就自定义属性，区别适用于文档的 Range，而不是与指定文本节点相关联。

然而，装饰是在**渲染时**根据内容本身计算的。这对于语法高亮或者关键词搜索等动态格式很有帮助，对内容（或者外部内容）的更改可能会更改格式。

装饰与 Mark 的不同点是它们不在编辑器状态中存储。

## 工具栏、菜单、浮层等等！

除了在 Slate 中控制节点的渲染，还可以使用 `useSlate` hook 从其它组件中检索当前编辑器上下文。

这样其它组件可以执行命令，查询编辑器状态或者其它任何事情。

一个常见的用例是渲染带有格式化按钮的工具栏，这些根据当前选区高亮显示：

```jsx
const MyEditor = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  return (
    <Slate editor={editor}>
      <Toolbar />
      <Editable />
    </Slate>
  )
}

const Toolbar = () => {
  const editor = useSlate()
  return (
    <div>
      <Button active={isBoldActive(editor)}>B</Button>
      <Button active={isItalicActive(editor)}>I</Button>
    </div>
  )
}
```

因为 `<Toolbar>` 使用 `useSlate` hook 检索上下文，所以会在编辑器更改时重新渲染，以便按钮的状态保持同步。
