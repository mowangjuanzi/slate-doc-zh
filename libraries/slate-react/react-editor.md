# ReactEditor

`ReactEditor` is added to `Editor` when it is instantiated using the `withReact` method.

```typescript
const [editor] = useState(() => withReact(withHistory(createEditor())))
```

- [Static methods](react-editor.md#static-methods)
  - [Check methods](react-editor.md#check-methods)
  - [Focus and selection methods](react-editor.md#focus-and-selection-methods)
  - [DOM translation methods](react-editor.md#dom-translation-methods)
  - [DataTransfer methods](react-editor.md#datatransfer-methods)

## Static methods

### Check methods

#### `ReactEditor.isComposing(editor: ReactEditor): boolean`

Check if the user is currently composing inside the editor.

#### `ReactEditor.isFocused(editor: ReactEditor): boolean`

Check if the editor is focused.

#### `ReactEditor.isReadOnly(editor: ReactEditor): boolean`

Check if the editor is in read-only mode.

### Focus and selection methods

#### `ReactEditor.blur(editor: ReactEditor): void`

Blur the editor.

#### `ReactEditor.focus(editor: ReactEditor): void`

Focus the editor.

#### `ReactEditor.deselect(editor: ReactEditor): void`

Deselect the editor.

### DOM translation methods

#### `ReactEditor.findKey(editor: ReactEditor, node: Node): Key`

Find a key for a Slate node.

Returns an instance of `Key` which looks like `{ id: string }`

#### `ReactEditor.findPath(editor: ReactEditor, node: Node): Path`

Find the path of Slate node.

#### `ReactEditor.hasDOMNode(editor: ReactEditor, target: DOMNode, options: { editable?: boolean } = {}): boolean`

Check if a DOM node is within the editor.

#### `ReactEditor.toDOMNode(editor: ReactEditor, node: Node): HTMLElement`

Find the native DOM element from a Slate node.

#### `ReactEditor.toDOMPoint(editor: ReactEditor, point: Point): DOMPoint`

Find a native DOM selection point from a Slate point.

#### `ReactEditor.toDOMRange(editor: ReactEditor, range: Range): DOMRange`

Find a native DOM range from a Slate `range`.

#### `ReactEditor.toSlateNode(editor: ReactEditor, domNode: DOMNode): Node`

Find a Slate node from a native DOM `element`.

#### `ReactEditor.findEventRange(editor: ReactEditor, event: any): Range`

从 DOM `event` 中获取目标范围。

#### `ReactEditor.toSlatePoint(editor: ReactEditor, domPoint: DOMPoint): Point | null`

从 DOM 选区的 `domNode` 和 `domOffset` 中查找 Slate 点。

#### `ReactEditor.toSlateRange(editor: ReactEditor, domRange: DOMRange | DOMStaticRange | DOMSelection, options?: { exactMatch?: boolean } = {}): Range | null`

从 DOM 范围或者选区中查找 Slate 范围。

### DataTransfer methods

#### `ReactEditor.insertData(editor: ReactEditor, data: DataTransfer): void`

Insert data from a `DataTransfer` into the editor. This is a proxy method to call in this order `insertFragmentData(editor: ReactEditor, data: DataTransfer)` and then `insertTextData(editor: ReactEditor, data: DataTransfer)`.

#### `ReactEditor.insertFragmentData(editor: ReactEditor, data: DataTransfer): true`

Insert fragment data from a `DataTransfer` into the editor. Returns true if some content has been effectively inserted.

#### `ReactEditor.insertTextData(editor: ReactEditor, data: DataTransfer): true`

Insert text data from a `DataTransfer` into the editor. Returns true if some content has been effectively inserted.

#### `ReactEditor.setFragmentData(editor: ReactEditor, data: DataTransfer, originEvent?: 'drag' | 'copy' | 'cut'): void`

Sets data from the currently selected fragment on a `DataTransfer`.