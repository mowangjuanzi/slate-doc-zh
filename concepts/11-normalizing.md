# 标准化

Slate 编辑器可以编辑复杂嵌套的数据结构。大部分情况很棒。但在某些情况下可能会引起数据结构不一致 —— 最常见的是允许用户粘贴任意富文本内容。

“标准化”是确保编辑器内容始终具有特定形态的方法。类似“验证”，除了确认内容是否有效之外，还要修复内容使其再次有效。

## 内置约束

Slate 带有一些开箱即用的内置约束。这些约束使内容的处理比标准 `contenteditable` _更_可预测。Slate 中的所有内置逻辑都依赖这些约束，因此不能省略它们，它们是。。。

1. **所有的 `Element` 节点最下级都必须包含至少一个 `Text` 子节点** —— 甚至是 [void 元素](./02-nodes.md#voids)。如果元素节点未包含任何子节点，则会添加一个空文本节点作为其唯一的子节点。存在该约束以确保选区的锚点和焦点（依赖引用文本节点）始终可以放置在任何节点内。不这样，空元素（或者 void 元素）将不可选。
2. **相同自定义属性且相邻的两个文本将会合并。** 如果相邻的两个节点具有相同的格式，将会合并成一个文本节点，其中包含两者组合后的文本字符串。这样做是为了防止文档中的文本节点数量不断扩大，因为添加/删除格式都会导致拆分文本节点。
3. **块节点只能包含其它块或者行内/文本节点。** 例如，一个 `paragraph` 块不能同时有另一个 `paragraph` 块元素_和_ `link` 内联元素作为子节点。子节点的类型由第一个子节点确定，其它任何不符合的子节点都将被删除。这确保了常见的富文本行为（像是“块一分为二” 的功能）始终如一。
4. **内联元素不能父级块的首尾第一个节点，也不能与子数组中的首个内联节点相邻。** 如果是这种情况，将添加一个空文本节点一修复它以符合约束。
5. **顶级编辑器节点只能包含块节点。** 将会删除任何顶级子节点中的内联或文本节点。这确保了编辑器中始终存在块节点，以便“块元素一分为二”之类的行为按预期工作。
6. **节点必须是 JSON 可序列化的。** 例如，避免在数据模型中使用 `undefined`。这确保了[操作](./05-operations.md) 是 JSON 可序列化，协作库假设的属性。
7. **属性值不能为 `null`。** 相反，应该使用可选属性，例如 `foo?: string` 而不是 `foo: string | null`。此限制是因为在[操作](./05-operations.md)中使用 `null` 表示缺少的属性。

这些默认约束都是强制性的，因为它们使 Slate 文档的工作_更_加可预测。

> 🤖 虽然这些约束是我们能够想到的最好约束，但是我们也在寻找使 Slate 的内置约束尽可能减少的办法 —— 只要保持标准行为更易于推理。如果能想到一种通过不同方法减少或者删除内置约束的办法，我们洗耳恭听！

## 增加约束

内置约束相当通用。但是也可以在内置之上为特定域添加自己的约束。

为此，可以在编辑上扩展 `normalizeNode` 函数。操作每次应用插入或者更新节点（及其子节点）时都会调用 `normalizeNode` 函数，确保每次更改都不会让其处于无效状态，如果是的话，则会修复节点。

例如，这是一个确保 `paragraph` 块值存在文本或者内联元素作为子节点的插件：

```javascript
import { Transforms, Element, Node } from 'slate'

const withParagraphs = editor => {
  const { normalizeNode } = editor

  editor.normalizeNode = entry => {
    const [node, path] = entry

    // 如果元素是段落，确保子节点是有效的。
    if (Element.isElement(node) && node.type === 'paragraph') {
      for (const [child, childPath] of Node.children(editor, path)) {
        if (Element.isElement(child) && !editor.isInline(child)) {
          Transforms.unwrapNodes(editor, { at: childPath })
          return
        }
      }
    }

    // 调用原始的 `normalizeNode` 以强制执行其它约束。
    normalizeNode(entry)
  }

  return editor
}
```

这个例子相当简单。每当在段落元素上调用 `normalizeNode` 时，都会遍历其每个子元素以确保子元素都不是块元素。如果有一个是块元素，将会解包，这样块就会被移除，其子元素就会取而代之。节点就会被“修复”。

但是子元素有嵌套块呢？

## 多重标准化

关于 `normalizeNode` 约束要理解的一件事就是它们是**多重的**。

如果再次检查上面的例子，会注意到上面的 `return` 语句：

```javascript
if (Element.isElement(child) && !editor.isInline(child)) {
  Transforms.unwrapNodes(editor, { at: childPath })
  return
}
```

你可能刚开始觉得很奇怪，因为有 `return`，原始的 `normalizeNodes` 将永远不会调用且内置约束将没有机会运行自己的标准化。

但对于标准化来说，这只是一个小“把戏”。

当调用 `Transforms.unwrapNodes` 时，实际上是正在标准化当前节点的内容。因此即使结束当前标准化过程，通过更改节点，也将开始_新的_标准化过程。这导致了一种_递归_标准化。

这种多重特性使得编写规范_更加_容易，因为一次只需要担心且修复一个问题，而不是修复_所有_可能使节点处于无效状态的问题。

要了解实战中是如何工作的，从这个无效文档开始：

```jsx
<editor>
  <paragraph a>
    <paragraph b>
      <paragraph c>word</paragraph>
    </paragraph>
  </paragraph>
</editor>
```

编辑器首先在 `<paragraph c>` 上运行 `normalizeNode`。它是有效的，因为子节点只包含文本节点。

接下来，会向上移动，并在 `<paragraph b>` 运行 `normalizeNode`。此段落无效，因为它包含块元素（`<paragraph c>`）。所以子快就会解包从而产生一个新文档：

```jsx
<editor>
  <paragraph a>
    <paragraph b>word</paragraph>
  </paragraph>
</editor>
```

并且在执行修复后，顶级 `<paragraph a>` 变了。它被标准化了，且变为有效，所以解包 `<paragraph b>` 后变为：

```jsx
<editor>
  <paragraph a>word</paragraph>
</editor>
```

现在当 `normalizeNode` 运行时，没有发生变化，因此该文档是有效的！

> 🤖 在大多数情况下并不需要考虑这些内部结构。只需要知道调用 `normalizeNode` 并发现无状态，可以修复单个无效状态并相信再次调用 `normalizeNode` 直到节点变为有效。

## 空的子节点早期约束执行

特殊的标准化需要在所有其它标准化之前执行，在编写标准化器的时候要牢记这一点。

在执行任何其它标准化之前，Slate 会遍历所有的 `Element` 节点并确保最少有一个子节点。如果没有，会创建一个空的 `Text` 子节点。

当 `Element` 没有子节点且进行自定处理时，可能会犯错。例如，如果表格元素没有记录，可能会希望删除表格；然而，这永远不会发生，因为在标准化运行之前会自动创建一个 `Text` 节点。

## 错误修复

需要避免的问题是创建标准化死循环。如果检查特定的无效结构，并实际上并**没有**通过更改节点来修复该结构，则可能会发生这种情况。这回导致死循环，因为该节点继续标记为无效，但它从未正确修复过。

例如，确保 `link` 元素具有有效 `url` 的标准化：

```javascript
// WARNING：这是一个不正确行为的示例！
const withLinks = editor => {
  const { normalizeNode } = editor

  editor.normalizeNode = entry => {
    const [node, path] = entry

    if (
      Element.isElement(node) &&
      node.type === 'link' &&
      typeof node.url !== 'string'
    ) {
      // ERROR：对 url 来说 null 不是有效值
      Transforms.setNodes(editor, { url: null }, { at: path })
      return
    }

    normalizeNode(entry)
  }

  return editor
}
```

此修复程序编写错误。它希望确保所有 `link` 元素都有 `url` 属性字符串。但是要修复无效链接，会将 `url` 设置为 `null`，这仍然不是字符串！

在这种情况下，可能想要解包链接，将其完全删除。_或者_扩展验证（`url == null`）以接受“空”链接 。

## 对其它代码的影响

如果节点树_不_应在转换之间规范化，则转换序列可能需要包装在 [`Editor.withoutNormalizing`](../api/nodes/editor.md#editorwithoutnormalizingeditor-editor-fn---void--void) 中。
当 `unwrapNodes` 后跟 `wrapNodes` 时，通常会出现这种情况。例如，可以编写汉书来改变块的类型，如下所示：

```javascript
const LIST_TYPES = ['numbered-list', 'bulleted-list']

function changeBlockType(editor, type) {
  Editor.withoutNormalizing(editor, () => {
    const isActive = isBlockActive(editor, type)
    const isList = LIST_TYPES.includes(type)

    Transforms.unwrapNodes(editor, {
      match: n =>
        LIST_TYPES.includes(
          !Editor.isEditor(n) && SlateElement.isElement(n) && n.type
        ),
      split: true,
    })
    const newProperties = {
      type: isActive ? 'paragraph' : isList ? 'list-item' : type,
    }
    Transforms.setNodes(editor, newProperties)

    if (!isActive && isList) {
      const block = { type: type, children: [] }
      Transforms.wrapNodes(editor, block)
    }
  })
}
```
