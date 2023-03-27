# 节点

最重要的类型是 `Node` 对象：

- 包含整个文档内容的根级 `Editor` 节点。
- 在域中有语义的容器 `Element` 节点。
- 包含文档文本的叶级 `Text` 节点。

这三个接口组合在一起形成一棵树 —— 就像 DOM 一样。例如：这是一个简单的纯文本值：

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
  // ...编辑器也有其它属性。
}
```

Slate 的原则之一是就可能多的镜像 DOM。人们一直使用 DOM 来表示具有类富文本结构的文档。镜像 DOM 有助于新用户熟悉该类库，并且能够复用经过实战考验的模式，而无需自我发明。

> 🤖 火狐开发者网络上有下列内容可以更好的学习相应的 DOM 概念：
>
> - [文档](https://developer.mozilla.org/en-US/docs/Web/API/Document)
> - [块级元素](https://developer.mozilla.org/en-US/docs/Web/HTML/Block-level_elements)
> - [行内元素](https://developer.mozilla.org/en-US/docs/Web/HTML/Inline_elements)
> - [文本元素](https://developer.mozilla.org/en-US/docs/Web/API/Text)

Slate 文档是嵌套递归结构。在文档中，元素有子节点 —— 子节点数量没有限制。嵌套和递归结构能够对简单（比如用户提及或者话题标签）或者复杂（比如表格和带标题的数字）的行为进行建模。

## `Editor`

Slate 的顶级节点是 `Editor` 本身。它概括了文档的所有富文本 “content”。它的接口是：

```typescript
interface Editor {
  children: Node[]
  ...
}
```

稍后会介绍它的功能，但对节点而言重要的是包含 `Node` 对象树的 `children` 属性。

## `Element`

元素构成了富文本文档的中间层。它们是域自定义节点。接口是：

```typescript
interface Element {
  children: Node[]
}
```

可以为所需的任何类型的内容自定义元素。例如，在数据模型可以通过 `type` 属性来区分段落还是引用：

```javascript
const paragraph = {
  type: 'paragraph',
  children: [...],
}

const quote = {
  type: 'quote',
  children: [...],
}
```

请务必注意，元素可使用_任何_所需的自定义属性。该示例中，Slate 不会知道或者关心 `type` 属性。如果正在定义 "link" 节点，可能需要有 `url` 属性：

```javascript
const link = {
  type: 'link',
  url: 'https://example.com',
  children: [...],
}
```

或者想给所有的节点添加一个 ID 属性：

```javascript
const paragraph = {
  id: 1,
  type: 'paragraph',
  children: [...],
}
```

重要的是元素都会有 `children` 属性。

## 块级与行内

根据用例，可能想为 `Element` 节点定义另外一种行为来确定编排“顺序”。

所有的元素默认都是“块级”元素。每个都被垂直空间分割开且不会重叠。

但在某些情况下，像是链接，可能会将其设为“内联”连续元素。这样它们就可以跟文本节点相同且连续。

> 🤖 这是从 DOM 行为中借用的概念，参阅[块级元素](https://developer.mozilla.org/en-US/docs/Web/HTML/Block-level_elements)和[内联元素](https://developer.mozilla.org/en-US/docs/Web/HTML/Inline_elements)。

可以通过覆盖 `editor.isInline` 函数来定义那些节点视为行内节点。（默认总返回 `false`。）注意行内节点不能是父级块的第一个或者最后一个子节点，也不能与 `children` 数组中的另外一个内联节点相邻。Slate 将使用 [`normalizeNode`](11-normalizing.md#built-in-constraints) 自动将 `{ text: '' }` 子级分隔开。

元素可以包含块级元素或者混合文本节点的内联元素作为子元素。但元素的子元素**不能**是一部分块级元素，一部分是行内元素。

## Voids

类似块级和行内，可以根据用例定义另外一种特定于元素的行为：它们的 "void" 。

元素默认非空，这意味着其子元素完全可以作为文本编辑。但在某些情况下，例如图像，会希望确保 Slate 不会将其内容视为可编辑文本，而是视为黑箱子。

> 🤖 这是从 HTML 规范中借用的概念，参阅 [Void Elements](https://www.w3.org/TR/2011/WD-html-markup-20110405/syntax.html#void-element).

可以通过覆盖 `editor.isVoid` 函数来定义哪些元素被视为无效元素。（默认总返回 `false`。）参阅[渲染 Void 元素](../api/nodes/element.md#rendering-void-elements) 获取更多细节。

## `Text`

文本节点是树中最低级节点，包含文档的文本内容以及任何格式。接口是：

```typescript
interface Text {
  text: string
}
```

例如，粗体格式文本：

```javascript
const text = {
  text: 'A string of bold text',
  bold: true,
}
```

文本节点也可以包含任何自定义属性，这就是实现类似**粗体**，_斜体_，`代码`等自定义格式的方式。
这些自定义属性有时候称为[标记](../api/nodes/editor.md#mark-methods)。
