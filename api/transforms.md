# Transform API

转换是对文档进行操作的助手函数。可用于定义自己的命令。

- [Node 选项](transforms.md#node-options)
- [静态方法](transforms.md#static-methods)
  - [Node 转换](transforms.md#node-transforms)
  - [Selection 转换](transforms.md#selection-transforms)
  - [Text 转换](transforms.md#text-transforms)
  - [Editor 转换](transforms.md#editor-transforms)

## Node 选项

所有的转换都支持参数 `options`。 This includes options specific to the transform, and general `NodeOptions` to specify which Nodes in the document that the transform is applied to.

```typescript
interface NodeOptions {
  at?: Location
  match?: (node: Node, path: Location) => boolean
  mode?: 'highest' | 'lowest'
  voids?: boolean
}
```

- `at` 选项在编辑器中查找[位置](../concepts/03-locations.md)。默认是用户当前选区。[了解有关 `at` 选项的更多信息](../concepts/04-transforms.md#at-选项)

- `match` 选项使用自定义函数过滤节点集合。[了解有关 `match` 选项的更多信息](../concepts/04-transforms.md#match-选项)

- `mode` 选项也过滤节点集合。

- 当 `voids` 为 false, 将过滤掉 [void 元素](./nodes/editor.md#重写特定模式实例方法)。

## 静态方法

### Node 转换

在节点上操作转换。

#### `Transforms.insertFragment(editor: Editor, fragment: Node[], options?)`

Insert of fragment of nodes at the specified location or (if not defined) the current selection or (if not defined) the end of the document.

Options: `{at?: Location, hanging?: boolean, voids?: boolean}`

#### `Transforms.insertNodes(editor: Editor, nodes: Node | Node[], options?)`

Atomically inserts `nodes` at the specified location or (if not defined) the current selection or (if not defined) the end of the document.

Options supported: `NodeOptions & {hanging?: boolean, select?: boolean}`.

For example, to insert at the very end, without replacing the current selection and regardless of block nesting, use

```javascript
Transforms.insertNodes(
  editor,
  { type: targetType, children: [{ text: '' }] },
  { at: [editor.children.length] }
)
```

#### `Transforms.removeNodes(editor: Editor, options?)`

Remove nodes at the specified location in the document. If no location is specified, remove the nodes in the selection.

Options supported: `NodeOptions & {hanging?: boolean}`

#### `Transforms.mergeNodes(editor: Editor, options?)`

Merge a node at the specified location with the previous node at the same depth. If no location is specified, use the selection. Resulting empty container nodes are removed.

Options supported: `NodeOptions & {hanging?: boolean}`

#### `Transforms.splitNodes(editor: Editor, options?)`

Split nodes at the specified location. If no location is specified, split the selection.

Options supported: `NodeOptions & {height?: number, always?: boolean}`

#### `Transforms.wrapNodes(editor: Editor, element: Element, options?)`

Wrap nodes at the specified location in the `element` container. If no location is specified, wrap the selection.

Options supported: `NodeOptions & {split?: boolean}`.

- `options.mode`: `'all'` is also supported.
- `options.split` indicates that it's okay to split a node in order to wrap the location. For example, if `ipsum` was selected in a `Text` node with `lorem ipsum dolar`, `split: true` would wrap the word `ipsum` only, resulting in splitting the `Text` node. If `split: false`, the entire `Text` node `lorem ipsum dolar` would be wrapped.

#### `Transforms.unwrapNodes(editor: Editor, options?)`

Unwrap nodes at the specified location. If necessary, the parent node is split. If no location is specified, use the selection.

Options supported: `NodeOptions & {split?: boolean}`. For `options.mode`, `'all'` is also supported.

#### `Transforms.setNodes(editor: Editor, props: Partial<Node>, options?)`

Set properties of nodes at the specified location. If no location is specified, use the selection.

Options supported: `NodeOptions & {hanging?: boolean, split?: boolean}`. For `options.mode`, `'all'` is also supported.

#### `Transforms.unsetNodes(editor: Editor, props: string | string[], options?)`

Unset properties of nodes at the specified location. If no location is specified, use the selection.

Options supported: `NodeOptions & {hanging?: boolean, split?: boolean}`. For `options.mode`, `'all'` is also supported.

#### `Transforms.liftNodes(editor: Editor, options?)`

Lift nodes at the specified location upwards in the document tree. If necessary, the parent node is split. If no location is specified, use the selection.

Options supported: `NodeOptions`. For `options.mode`, `'all'` is also supported.

#### `Transforms.moveNodes(editor: Editor, options)`

Move the nodes from an origin to a destination. A destination must be specified in the `options`. If no origin is specified, move the selection.

Options supported: `NodeOptions & {to: Path}`. For `options.mode`, `'all'` is also supported.

### Selection 转换

Transforms that operate on the document's selection.

#### `Transforms.collapse(editor: Editor, options?)`

Collapse the selection to a single point.

Options: `{edge?: 'anchor' | 'focus' | 'start' | 'end'}`

#### `Transforms.select(editor: Editor, target: Location)`

Set the selection to a new value specified by `target`. When a selection already exists, this method is just a proxy for `setSelection` and will update the existing value.

For example, to set the selection to the entire contents of the editor:

```javascript
Transforms.select(editor, {
  anchor: Editor.start(editor, []),
  focus: Editor.end(editor, []),
})
```

#### `Transforms.deselect(editor: Editor)`

Unset the selection.

#### `Transforms.move(editor: Editor, options?)`

Move the selection's point forward or backward.

Options: `{distance?: number, unit?: 'offset' | 'character' | 'word' | 'line', reverse?: boolean, edge?: 'anchor' | 'focus' | 'start' | 'end'}`

#### `Transforms.setPoint(editor: Editor, props: Partial<Point>, options?)`

Set new properties on one of the selection's points.

Options: `{edge?: 'anchor' | 'focus' | 'start' | 'end'}`

#### `Transforms.setSelection(editor: Editor, props: Partial<Range>)`

Set new properties on an active selection. Since the value is a `Partial<Range>`, this method can only handle updates to an existing selection. If there is no active selection the operation will be void. Use `select` if you'd like to create a selection when there is none.

### Text 转换

Transforms that operate on text.

#### `Transforms.delete(editor: Editor, options?)`

Delete text in the document.

Options: `{at?: Location, distance?: number, unit?: 'character' | 'word' | 'line' | 'block', reverse?: boolean, hanging?: boolean, voids?: boolean}`

#### `Transforms.insertText(editor: Editor, text: string, options?)`

Insert a string of text at the specified location or (if not defined) the current selection or (if not defined) the end of the document.

Options: `{at?: Location, voids?: boolean}`

### Editor 转换

#### `Transforms.transform(editor: Editor, transform: Transform)`

Transform the `editor` by an `operation`.
