# Location 类型 API

`Location` 接口是引用 Slate 文档中特定位置的多种方式的联合：路径、点或范围。方法通常会接受 `Location` 而不是只需要 `Path`、`Point` 或 `Range`。

```typescript
type Location = Path | Point | Range
```

- [Location](./location.md)
- [Path](./path.md)
- [PathRef](./path-ref.md)
- [Point](./point.md)
- [PointEntry](./point-entry.md)
- [PointRef](./point-ref.md)
- [Range](./range.md)
- [RangeRef](./range-ref.md)
- [Span](./span.md)
