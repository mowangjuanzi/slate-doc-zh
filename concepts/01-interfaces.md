# 接口

> Commit ID: [706f2fc07297ea639bb8218b3de93bd9e8b4608c](https://github.com/ianstormtaylor/slate/blob/main/docs/concepts/01-interfaces.md)

Slate 使用纯 JSON 对象。要求那些 JSON 对象符合某些接口。例如，Slate 中的文本节点必须遵循 `Text` 接口：

```typescript
interface Text {
  text: string
}
```

这意味着它必须带有字符串内容 `text` 属性。

但也允许其它**任何**自定义属性，这完全取决于开发人员。可以定制数据到特定领域或者用例中，并添加想要任何格式逻辑，而不会被 Slate 阻碍。

基于接口的方法将 Slate 与大多数其他使用手动 “model” 类的富文本编辑器分类开来，可以更好的理解它。这也意味着避免了在启动时“初始化”数据模型造成的时间损失。

## 自定义属性

据个例子， Slate 中的 `Element` 节点接口是：

```typescript
interface Element {
  children: Node[]
}
```

这是个非常宽松的接口。它只要求 `children` 属性定义为包含元素的子节点。

但是也可以继承包含特定领域的自定义属性的元素（或者其它接口）。例如，可能有 “paragraph” 和 “link” 元素：

```javascript
const paragraph = {
  type: 'paragraph',
  children: [...],
}

const link = {
  type: 'link',
  url: 'https://example.com',
  children: [...]
}
```

`type` 和 `url` 属性是自定义 API。Slate 能看到它们，但不使用它们。然而当 Slate 渲染 link 元素，将会接收一个附带自定义属性的对象，以便可以将其渲染为：

```jsx
<a href={element.url}>{element.children}</a>
```

当开始使用 Slate 时，了解所有定义的接口非常重要。每个教程中都讨论了一些接口。

## 助手函数

除了类型信息之外，Slate 中的每个接口还公开了一系列助手函数，使其更易于使用。

例如，使用节点时：

```javascript
import { Node } from 'slate'

// 获取元素节点的字符串内容。
const string = Node.string(element)

// 获取 root 节点内的指定节点。
const descendant = Node.get(value, path)
```

或者使用范围时：

```javascript
import { Range } from 'slate'

// 按顺序获取范围的起点和终点。
const [start, end] = Range.edges(range)

// 检查范围是否折叠为一个点。
if (Range.isCollapsed(range)) {
  // ...
}
```

对于所有不同接口的常见用例都有很多助手函数可以使用。开始的时候通读这些会有所帮助，因此可以将复杂的逻辑简化为几行代码。

## 自定义助手

除了内置助手函数之外，可能还会想定义自定义助手函数并在自定义空间中将其公开。

例如，如果编辑器支持图像，可能需要一个助手来确定元素是否是图像元素：

```javascript
const isImageElement = element => {
  return element.type === 'image' && typeof element.url === 'string'
}
```

可以轻松将它们定义为一次性功能。也可以将它们绑定到命名空间中，就像核心接口一样，然后使用它们。

例如：

```javascript
import { Element } from 'slate'

// 可以在任何地方使用 `MyElement` 访问扩展。
export const MyElement = {
  ...Element,
  isImageElement,
  isParagraphElement,
  isQuoteElement,
}
```

这使得同时使用 Slate 内置助手以及特定领取逻辑变得很容易。
