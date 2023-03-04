# Operation API

> Commit ID: [55effa953c353cd80fbc93cb47ef66a8ec2031e5](https://github.com/ianstormtaylor/slate/blob/main/docs/api/operation.md)

`Operation` 对象定义了底层指令，用来 Slate 编辑其应用更改到其内部状态。将所有的更改表示为操作是允许 Slate 编辑器轻松实现历史记录、写作和其他功能的原因。

### Check methods

#### `isNodeOperation(value: any) => boolean`

Check if a value is a `NodeOperation` object. Returns the value as a `NodeOperation` if it is one.

#### `isOperation(value: any) => boolean`

Check if a value is an `Operation` object. Returns the value as an `Operation` if it is one.

#### `isOperationList(value: any) => boolean`

Check if a value is a list of `Operation` objects. Returns the value as an `Operation[]` if it is one.

#### `isSelectionOperation(value: any) => boolean`

Check if a value is a `SelectionOperation` object. Returns the value as a `SelectionOperation` if it is one.

#### `isTextOperation(value: any) => boolean`

Check if a value is a `TextOperation` object. Returns the value as a `TextOperation` if it is one.

### Static methods

#### `inverse(op: Operation) => Operation`

Invert an operation, returning a new operation that will exactly undo the original when applied.
