# 序列化

> Commit ID: [d460bb42f053e1fcd98966b63b1e843e21eae8b7](https://github.com/ianstormtaylor/slate/blob/main/docs/concepts/10-serializing.md)

Slate 的数据模型在构建时就考虑了序列化。准确的说，文本节点的定义方式使其一目了然，但也易于序列化为 HTML 和 Markdown 等常见格式。

而且，Slate 使用 JSON 作为数据，可以非常轻松的编写序列化逻辑。

## 纯文本

例如，获取编辑器的值并返回纯文本：

```javascript
import { Node } from 'slate'

const serialize = nodes => {
  return nodes.map(n => Node.string(n)).join('\n')
}
```

这里将 `Editor` 的子节点作为 `nodes` 参数并返回纯文本表示，其中每个顶级节点都使用单个 `\n` 换行符分隔。

对以下输入：

```javascript
const nodes = [
  {
    type: 'paragraph',
    children: [{ text: 'An opening paragraph...' }],
  },
  {
    type: 'quote',
    children: [{ text: 'A wise quote.' }],
  },
  {
    type: 'paragraph',
    children: [{ text: 'A closing paragraph!' }],
  },
]
```

将会得到：

```text
An opening paragraph...
A wise quote.
A closing paragraph!
```

注意引用无法用任何方式区分，这是因为我们讨论的是纯文本。但是可以将数据序列化为想要的格式 —— 毕竟它只是 JSON。

## HTML

例如，这是差不多的 HTML `serialize` 函数：

```javascript
import escapeHtml from 'escape-html'
import { Text } from 'slate'

const serialize = node => {
  if (Text.isText(node)) {
    let string = escapeHtml(node.text)
    if (node.bold) {
      string = `<strong>${string}</strong>`
    }
    return string
  }

  const children = node.children.map(n => serialize(n)).join('')

  switch (node.type) {
    case 'quote':
      return `<blockquote><p>${children}</p></blockquote>`
    case 'paragraph':
      return `<p>${children}</p>`
    case 'link':
      return `<a href="${escapeHtml(node.url)}">${children}</a>`
    default:
      return children
  }
}
```

这个比上面的纯文本序列化器更加清楚一些。实际上它是_递归_，这样能保证深入遍历节点中的子节点，一直到文本叶子节点。对于接收到的每个节点，都会转化为 HTML 字符串。

还会将单个节点用作输入而不是数组，所以如果像是这样传递进编辑器：

```javascript
const editor = {
  children: [
    {
      type: 'paragraph',
      children: [
        { text: 'An opening paragraph with a ' },
        {
          type: 'link',
          url: 'https://example.com',
          children: [{ text: 'link' }],
        },
        { text: ' in it.' },
      ],
    },
    {
      type: 'quote',
      children: [{ text: 'A wise quote.' }],
    },
    {
      type: 'paragraph',
      children: [{ text: 'A closing paragraph!' }],
    },
  ],
  // `Editor` 对象还具有省略的其它属性。。。
}
```

接到的结果是（添加换行符增加可读性）：

```html
<p>An opening paragraph with a <a href="https://example.com">link</a> in it.</p>
<blockquote><p>A wise quote.</p></blockquote>
<p>A closing paragraph!</p>
```

就是这么简单！

## 反序列化

Slate 中的另一个常见用例是反序列化。这一些任意输入需要转化为兼容 Slate 的 JSON 结构时。例如当粘贴 HTML 到编辑器，希望确保使用合适的编辑器格式对其进行解析。

Slate 有一个内置助手： `slate-hyperscript` 包。

使用 `slate-hyperscript` 最常见的方式是编写 JSX 文档，例如编写当编写测试时。你可以这么使用它：

```jsx
/** @jsx jsx */
import { jsx } from 'slate-hyperscript'

const input = (
  <fragment>
    <element type="paragraph">A line of text.</element>
  </fragment>
)
```

并且编译器（Babel、TypeScript等）的  JSX 功能会将 `input` 变量转换为：

```javascript
const input = [
  {
    type: 'paragraph',
    children: [{ text: 'A line of text.' }],
  },
]
```

这对于编写测试用例或者希望以易读的方式编写大量 Slate 对象的地方特别有用。

然而！这对于反序列化没有帮助。

但是 `slate-hyperscript` 不仅仅适用于 JSX。它只是构建 _Slate 内容树_的一种方式。当反序列化 HTML 之类的内容时，这正是想要做的。

例如，下面是 HTML 的 `deserialize` 函数：

```javascript
import { jsx } from 'slate-hyperscript'

const deserialize = (el, markAttributes = {}) => {
  if (el.nodeType === Node.TEXT_NODE) {
    return jsx('text', markAttributes, el.textContent)
  } else if (el.nodeType !== Node.ELEMENT_NODE) {
    return null
  }

  const nodeAttributes = { ...markAttributes }

  // 为文本节点定义属性
  switch (el.nodeName) {
    case 'strong':
      nodeAttributes.bold = true
  }

  const children = Array.from(el.childNodes)
    .map(node => deserialize(node, nodeAttributes))
    .flat()

  if (children.length === 0) {
    children.push(jsx('text', nodeAttributes, ''))
  }

  switch (el.nodeName) {
    case 'BODY':
      return jsx('fragment', {}, children)
    case 'BR':
      return '\n'
    case 'BLOCKQUOTE':
      return jsx('element', { type: 'quote' }, children)
    case 'P':
      return jsx('element', { type: 'paragraph' }, children)
    case 'A':
      return jsx(
        'element',
        { type: 'link', url: el.getAttribute('href') },
        children
      )
    default:
      return children
  }
}
```

它接受 `el` HTML 元素对象并返回 Slate 片段。如果对于 HTML 字符串，可以像这样对其解析和反序列化：

```javascript
const html = '...'
const document = new DOMParser().parseFromString(html, 'text/html')
deserialize(document.body)
```

使用此输入：

```html
<p>An opening paragraph with a <a href="https://example.com">link</a> in it.</p>
<blockquote><p>A wise quote.</p></blockquote>
<p>A closing paragraph!</p>
```

得到的输出：

```javascript
const fragment = [
  {
    type: 'paragraph',
    children: [
      { text: 'An opening paragraph with a ' },
      {
        type: 'link',
        url: 'https://example.com',
        children: [{ text: 'link' }],
      },
      { text: ' in it.' },
    ],
  },
  {
    type: 'quote',
    children: [
      {
        type: 'paragraph',
        children: [{ text: 'A wise quote.' }],
      },
    ],
  },
  {
    type: 'paragraph',
    children: [{ text: 'A closing paragraph!' }],
  },
]
```

就像是序列化函数一样，可以对其扩展以适应精确域模型的需求。
