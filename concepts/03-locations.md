# 位置

位置是在 Slate 编辑器中插入、删除、执行任何操作时引用文档中的特定位置的方式。有几种不同类型的位置接口用于不同的用例。

## `Path`

路径是位置引用最低层方式。每个路径都是简单的数字数组，通过文档树中的每个祖先节点的索引来引用节点：

```typescript
type Path = number[]
```

例如，在文档中：

```javascript
const editor = {
  children: [
    {
      type: 'paragraph',
      children: [
        {
          text: 'A line of text!',
        },
      ],
    },
  ],
}
```

文本节点的叶子的路径是： `[0, 0]`。

编辑器本身的路径是 `[]`。例如，调用 `Transforms.select(editor, [])` 选择编辑器的全部内容。

## `Point`

点比路径更加具体，且包含特定文本的 `offset`。接口是：

```typescript
interface Point {
  path: Path
  offset: number
}
```

例如，对于同一个文档，如果想引用第一个位置，可以将光标放在：

```javascript
const start = {
  path: [0, 0],
  offset: 0,
}
```

或者想引用句子的结尾：

```javascript
const end = {
  path: [0, 0],
  offset: 15,
}
```

将点视为“光标”（“插入符号”）可能会有所帮助。

> 🤖 点_总是_指的是文本节点！因为它们是唯一有光标的字符串。

## `Range`

范围不仅仅可以引用文档中的单个点，还可以引用两个点之间更广泛的内容。（范围的例子是选区。）接口是：

```typescript
interface Range {
  anchor: Point
  focus: Point
}
```

> 🤖 术语“锚点”（anchor）和“焦点”（focus）是从 DOM 中借用的，参阅[锚点](https://developer.mozilla.org/en-US/docs/Web/API/Selection/anchorNode)和[焦点](https://developer.mozilla.org/en-US/docs/Web/API/Selection/focusNode)。

锚点和焦点是通过用户交互确定的。在文档中锚点不总是在焦点_之前_。就像在 DOM 中，锚点和焦点的顺序取决于范围是向后还是向前。

以下是火狐开发者网络的解释：

> 用户可能从左到右（与文档方向相同）选择文本或从右到左（与文档方向相反）选择文本。锚点指向用户开始选择的地方，而焦点指向用户结束选择的地方。如果使用鼠标选择文本的话，锚点就指向按下鼠标键的地方，而焦点就指向松开鼠标键的地方。锚点和焦点的概念不能与选区的起始位置和终止位置混淆，因为锚点指向的位置可能在焦点指向的位置的前面，也可能在焦点指向位置的后面，这取决于选择文本时鼠标移动的方向（也就是按下鼠标键和松开鼠标键的位置）。 —— [`Selection`, MDN](https://developer.mozilla.org/en-US/docs/Web/API/Selection)

一个重要的区别是在文档中范围的锚点和焦点**始终引用文本节点叶子**且绝不引用元素。此行为与 DOM 不同，但简化了使用范围，且需要处理的边缘情况更少。

> 🤖 参考[范围 API 参考](../api/locations/range.md)获取更多信息。

## 选区

当需要引用两点之间的内容范围时，Slate API 的中很多地方都使用了范围。最常见的例子是用户的当前“选区”。

选区是一个特殊范围，是顶级 `Editor` 的属性。例如，假设当前选择了整个句子：

```javascript
const editor = {
  selection: {
    anchor: { path: [0, 0], offset: 0 },
    focus: { path: [0, 0], offset: 15 },
  },
  children: [
    {
      type: 'paragraph',
      children: [
        {
          text: 'A line of text!',
        },
      ],
    },
  ],
}
```

> 🤖 选区概念也是从 DOM 中借用的，参阅 [`Selection`, MDN](https://developer.mozilla.org/en-US/docs/Web/API/Selection), 尽管形式非常简化，因为 Slate 不允许在单个选区中包含多个范围，但也使得易于使用。

没有特定的 `Selection` 接口。它只是正好遵循了更通用的 `Range` 接口。
