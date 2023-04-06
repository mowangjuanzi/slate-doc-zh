# Range API

`Range` 对象是一组指向 Slate 文档特定范围的点。它们可以在单个节点内定义，也可以跨越多个节点定义。编辑器的 `selection` 存储的是范围。

```typescript
interface Range {
  anchor: Point
  focus: Point
}
```

- [静态方法](range.md#静态方法)
  - [检索方法](range.md#检索方法)
  - [检查方法](range.md#检查方法)
  - [Transform methods](range.md#transform-methods)

## 静态方法

### 检索方法

#### `Range.edges(range: Range, options?) => [Point, Point]`

按照在文档中出现的顺序获取 `range` 的起点和终点。

选项: `{reverse?: boolean}`

#### `Range.end(range: Range) => Point`

根据在文档中出现的顺序获取 `range` 的终点。

#### `Range.intersection(range: Range, another: Range) => Range | null`

获取 `range` 与 `another` 的交集。如果两个范围不重叠，则返回 `null`。

#### `Range.points(range: Range) => Generator<PointEntry>`

Iterate through the two point entries in a `Range`. First it will yield a `PointEntry` representing the `anchor`, then it will yield a `PointEntry` representing the `focus`.

#### `Range.start(range: Range) => Point`

根据在文档中出现的顺序获取 `range` 的起点。

### 检查方法

检查 Range 的某些属性。始终返回布尔值。

#### `Range.equals(range: Range, another: Range) => boolean`

检查 `range` 是否完全等于 `another`。

#### `Range.includes(range: Range, target: Path | Point | Range) => boolean`

检查 `range` 是否包含路径、点或另一个范围的一部分。

For clarity the definition of `includes` can mean partially includes. Another way to describe this is if one Range intersectns the other Range.

#### `Range.isBackward(range: Range) => boolean`

Check if a `range` is backward, meaning that its anchor point appears _after_ its focus point in the document.

#### `Range.isCollapsed(range: Range) => boolean`

Check if a `range` is collapsed, meaning that both its anchor and focus points refer to the exact same position in the document.

#### `Range.isExpanded(range: Range) => boolean`

Check if a `range` is expanded. This is the opposite of `Range.isCollapsed` and is provided for legibility.

#### `Range.isForward(range: Range) => boolean`

Check if a `range` is forward. This is the opposite of `Range.isBackward` and is provided for legibility.

#### `Range.isRange(value: any) => value is Range`

Check if a `value` implements the `Range` interface.

### Transform methods

#### `Range.transform(range: Range, op: Operation, options) => Range | null`

Transform a `range` by an `op`.

Options: `{affinity: 'forward' | 'backward' | 'outward' | 'inward' | null}`
