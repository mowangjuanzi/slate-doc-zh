# Slate React

> Commit ID: [9bc0b6132aa288a37ae9a85d0e59a9d5a75ebdd7](https://github.com/ianstormtaylor/slate/blob/main/docs/libraries/slate-react.md)

该子库包含 Slate 特定于 React 的相关逻辑。

## 组件

用于渲染 Slate 编辑器的 React 组件

### `RenderElementProps`

`RenderElementProps` 传递给 `renderElement` 处理程序。

### `RenderLeafProps`

`RenderLeafProps` 传递给 `renderLeaf` 处理程序。

### `Editable`

主要的 Slate 编辑器。

#### 事件处理

默认情况下，`Editable` 组件带有一组事件处理程序，用于处理典型的富文本编辑行为（例如实现了自己的 `onCopy`、`onPaste`、`onDrop`、 `onKeyDown` 处理程序。

在某些情况下会想要扩展或者重写 Slate 的默认行为，可以通过将自定义的事件处理程序传递给 `Editable` 组件来实现。

Your custom event handler can control whether or not Slate should execute its own event handling for a given event after your handler runs depending on the return value of your event handler as described below.

```jsx
import {Editable} from 'slate-react';

function MyEditor() {
  const onClick = event => {
    // 实现自定义事件逻辑。。。

    // When no value is returned, Slate will execute its own event handler when
    // neither isDefaultPrevented nor isPropagationStopped was set on the event
  };

  const onDrop = event => {
    // 实现自定义事件逻辑。。。

    // No matter the state of the event, treat it as being handled by returning
    // true here, Slate will skip its own event handler
    return true;
  };

  const onDragStart = event => {
    // 实现自定义事件逻辑。。。

    // No matter the status of the event, treat event as *not* being handled by
    // returning false, Slate will execute its own event handler afterward
    return false;
  };

  return (
    <Editable
      onClick={onClick}
      onDrop={onDrop}
      onDragStart={onDragStart}
      {/*...*/}
    />
  )
}
```

### `DefaultElement(props: RenderElementProps)`

默认元素

### `DefaultLeaf(props: RenderLeafProps)`

The default custom leaf renderer.

### `Slate(editor: ReactEditor, value: Node[], children: React.ReactNode, onChange: (value: Node[]) => void, [key: string]: any)`

A wrapper around the provider to handle `onChange` events, because the editor is a mutable singleton so it won't ever register as "changed" otherwise.

## Hooks

Slate 编辑器的 React hook

### `useFocused`

获取当前编辑器的 `focused` 状态。

### `useReadOnly`

获取当前编辑器的 `readOnly` 状态。

### `useSelected`

获取当前编辑器的 `selected` 状态。

### `useSlate`

从 React 上下文中获取当前编辑器对象。每当编辑器中发生更改时重新渲染上下文。

### `useSlateWithV`

The same as `useSlate()` but includes a version counter which you can use to prevent re-renders.

### `useSlateStatic`

从 React 上下文中获取当前编辑器对象。 该 useSlate 版本不会重新渲染上下文。之前的调用 `useEditor`。

### `useSlateSelection`

Get the current editor selection from the React context. Only re-renders when the selection changes.

## ReactEditor

A React and DOM-specific version of the `Editor` interface. All about translating between the DOM and Slate.

### `isComposing(editor: ReactEditor)`

Check if the user is currently composing inside the editor.

### `findKey(editor: ReactEditor, node: Node)`

Find a key for a Slate node.

### `findPath(editor: ReactEditor, node: Node)`

Find the path of Slate node.

### `isFocused(editor: ReactEditor)`

Check if the editor is focused.

### `isReadOnly(editor: ReactEditor)`

Check if the editor is in read-only mode.

### `blur(editor: ReactEditor)`

Blur the editor.

### `focus(editor: ReactEditor)`

Focus the editor.

### `deselect(editor: ReactEditor)`

Deselect the editor.

### `hasDOMNode(editor: ReactEditor, target: DOMNode, options: { editable?: boolean } = {})`

Check if a DOM node is within the editor.

### `insertData(editor: ReactEditor, data: DataTransfer)`

Insert data from a `DataTransfer` into the editor. This is a proxy method to call in this order `insertFragmentData(editor: ReactEditor, data: DataTransfer)` and then `insertTextData(editor: ReactEditor, data: DataTransfer)`.

### `insertFragmentData(editor: ReactEditor, data: DataTransfer)`

Insert fragment data from a `DataTransfer` into the editor. Returns true if some content has been effectively inserted.

### `insertTextData(editor: ReactEditor, data: DataTransfer)`

Insert text data from a `DataTransfer` into the editor. Returns true if some content has been effectively inserted.

### `setFragmentData(editor: ReactEditor, data: DataTransfer, originEvent?: 'drag' | 'copy' | 'cut')`

Sets data from the currently selected fragment on a `DataTransfer`.

### `toDOMNode(editor: ReactEditor, node: Node)`

Find the native DOM element from a Slate node.

### `toDOMPoint(editor: ReactEditor, point: Point)`

Find a native DOM selection point from a Slate point.

### `toDOMRange(editor: ReactEditor, range: Range)`

Find a native DOM range from a Slate `range`.

### `toSlateNode(editor: ReactEditor, domNode: DOMNode)`

Find a Slate node from a native DOM `element`.

### `findEventRange(editor: ReactEditor, event: any)`

从 DOM `event` 中获取目标范围。

### `toSlatePoint(editor: ReactEditor, domPoint: DOMPoint)`

从 DOM 选区的 `domNode` 和 `domOffset` 中查找 Slate 点。

### `toSlateRange(editor: ReactEditor, domRange: DOMRange | DOMStaticRange | DOMSelection, options?: { exactMatch?: boolean } = {})`

从 DOM 范围或者选区中查找 Slate 范围。

## 插件

Slate 编辑器特定于React 插件

### `withReact(editor: Editor)`

添加 React and DOM 特定行为到编辑器。

当与 `withHistory` 一起使用时， `withReact` 应该应用于外部。例如：

```javascript
const [editor] = useState(() => withReact(withHistory(createEditor())))
```

## 工具

私有便利模块。
