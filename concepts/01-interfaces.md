# 接口

Slate 使用纯 JSON 对象。要求那些 JSON 对象符合某些接口。例如，Slate 中的文本节点必须遵循 `Text` 接口：

```typescript
interface Text {
  text: string
}
```

这意味着它必须带有字符串内容 `text` 属性。

但也允许其它**任何**自定义属性，这完全取决于开发人员。可以定制数据到特定领域或者用例中，并添加想要任何格式逻辑，而不会被 Slate 阻碍。

This interface-based approach separates Slate from most other rich text editors which require you to work with their hand-rolled "model" classes and makes it much easier to reason about. It also means that it avoids startup time penalties related to "initializing" the data model.

## 自定义属性

据个例子， Slate 中的 `Element` 节点接口是：

```typescript
interface Element {
  children: Node[]
}
```

This is a very permissive interface. All it requires is that the `children` property gets defined containing the element's child nodes.

But you can extend elements \(or any other interface\) with your custom properties that are specific to your domain. For example, you might have "paragraph" and "link" elements:

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

The `type` and `url` properties are your custom API. Slate sees that they exist, but doesn't use them. However, when Slate renders a link element, you'll receive an object with the custom properties attached so that you can render it as:

```jsx
<a href={element.url}>{element.children}</a>
```

When getting started with Slate, it's important to understand all of the interfaces it defines. There are a handful of interfaces that are discussed in each of the guides.

## 助手函数

In addition to the typing information, each interface in Slate also exposes a series of helper functions that make them easier to work with.

For example, when working with nodes:

```javascript
import { Node } from 'slate'

// Get the string content of an element node.
const string = Node.string(element)

// Get the node at a specific path inside a root node.
const descendant = Node.get(value, path)
```

Or, when working with ranges:

```javascript
import { Range } from 'slate'

// Get the start and end points of a range in order.
const [start, end] = Range.edges(range)

// Check if a range is collapsed to a single point.
if (Range.isCollapsed(range)) {
  // ...
}
```

There are many helper functions available for all common use cases when working with different interfaces. When getting started it helps to read through all of them so you can often simplify complex logic into just a handful of lines of code.

## 自定义助手

In addition to the built-in helper functions, you might want to define your custom helper functions and expose them on your custom namespaces.

For example, if your editor supports images, you might want a helper that determines if an element is an image element:

```javascript
const isImageElement = element => {
  return element.type === 'image' && typeof element.url === 'string'
}
```

You can define these as one-off functions easily. But you might also bundle them up into namespaces, just like the core interfaces do, and use them instead.

For example:

```javascript
import { Element } from 'slate'

// You can use `MyElement` everywhere to have access to your extensions.
export const MyElement = {
  ...Element,
  isImageElement,
  isParagraphElement,
  isQuoteElement,
}
```

This makes it easy to reuse domain-specific logic alongside the built-in Slate helpers.
