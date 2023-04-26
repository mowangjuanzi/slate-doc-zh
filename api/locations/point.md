# Point API

`Point` 对象是指 Slate 文档中文本节点中的特定位置。它的 `path` 是指节点在树中的位置，其 `offset` 是指到节点文本字符串的距离。Points 可能仅指 `Text` 节点。

```typescript
interface Point {
  path: Path
  offset: number
}
```

- [静态方法](point.md#静态方法)
  - [Retrieval methods](point.md#retrieval-methods)
  - [Check methods](point.md#check-methods)
  - [Transform methods](point.md#transform-methods)

## 静态方法

### Retrieval methods

#### `Point.compare(point: Point, another: Point) => -1 | 0 | 1`

Compare a `point` to `another`, returning an integer indicating whether the point was before, at or after the other.

### Check methods

#### `Point.isAfter(point: Point, another: Point) => boolean`

Check if a `point` is after `another`.

#### `Point.isBefore(point: Point, another: Point) => boolean`

Check if a `point` is before `another`.

#### `Point.equals(point: Point, another: Point) => boolean`

Check if a `point` is exactly equal to `another`.

#### `Point.isPoint(value: any) => value is Point`

Check if a `value` implements the `Point` interface.

### Transform methods

#### `Point.transform(point: Point, op: Operation, options?) => Point | null`

Transform a `point` by an `op`.

Options: `{affinity?: 'forward' | 'backward' | null}`
