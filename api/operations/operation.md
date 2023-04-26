# Operation API

`Operation` 对象定义了底层指令，用来 Slate 编辑其应用更改到其内部状态。将所有的更改表示为操作是允许 Slate 编辑器轻松实现历史记录、写作和其他功能的原因。

- [Static methods](operation.md#static-methods)
  - [Manipulation methods](operation.md#manipulation-methods)
  - [Check methods](operation.md#check-methods)

## Static methods

### Manipulation methods

#### `Operation.inverse(op: Operation) => Operation`

Invert an operation, returning a new operation that will exactly undo the original when applied.

### Check methods

#### `Operation.isNodeOperation(value: any) => boolean`

Check if a value is a `NodeOperation` object. Returns the value as a `NodeOperation` if it is one.

#### `Operation.isOperation(value: any) => boolean`

Check if a value is an `Operation` object. Returns the value as an `Operation` if it is one.

#### `Operation.isOperationList(value: any) => boolean`

Check if a value is a list of `Operation` objects. Returns the value as an `Operation[]` if it is one.

#### `Operation.isSelectionOperation(value: any) => boolean`

Check if a value is a `SelectionOperation` object. Returns the value as a `SelectionOperation` if it is one.

#### `Operation.isTextOperation(value: any) => boolean`

Check if a value is a `TextOperation` object. Returns the value as a `TextOperation` if it is one.
