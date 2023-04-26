# Slate React Hooks

- [Check hooks](hooks.md#check-hooks)
- [Editor hooks](hooks.md#editor-hooks)
- [Selection hooks](hooks.md#selection-hooks)

### Check hooks

Slate 编辑器的 React hook

#### `useFocused(): boolean`

获取当前编辑器的 `focused` 状态。

#### `useReadOnly(): boolean`

获取当前编辑器的 `readOnly` 状态。

#### `useSelected(): boolean`

获取当前编辑器的 `selected` 状态。

### Editor hooks

#### `useSlate(): ReactEditor`

从 React 上下文中获取当前编辑器对象。每当编辑器中发生更改时重新渲染上下文。

#### `useSlateWithV(): { editor: ReactEditor, v: number }`

The same as `useSlate()` but includes a version counter which you can use to prevent re-renders.

#### `useSlateStatic(): ReactEditor`

从 React 上下文中获取当前编辑器对象。 该 useSlate 版本不会重新渲染上下文。之前的调用 `useEditor`。

### Selection hooks

#### `useSlateSelection(): (BaseRange & { placeholder?: string | undefined; onPlaceholderResize?: ((node: HTMLElement | null) => void) | undefined }) | null`

Get the current editor selection from the React context. Only re-renders when the selection changes.

#### `useSlateSelector<T>(selector: (editor: Editor): T, equalityFn?: (a: T, b: T) => boolean): T`

Similar to `useSlateSelection` but uses redux style selectors to prevent rerendering on every keystroke.

Returns a subset of the full selection value based on the `selector`.

Bear in mind rerendering can only prevented if the returned value is a value type or for reference types (e.g. objects and arrays) add a custom equality function for the `equalityFn` argument.

Example:

```typescript
const isSelectionActive = useSlateSelector(editor => Boolean(editor.selection))
```