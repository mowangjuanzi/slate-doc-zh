# Span API

> Commit ID: [7d9d25e1790c6557e6ff2072f79f7904736aec65](https://github.com/ianstormtaylor/slate/blob/main/docs/api/locations/span.md)

A `Span` is a low-level way to refer to a `Range` using `Element` as the end points instead of a `Point` which requires the use of leaf text nodes.

```typescript
type Span = [Path, Path]
```

- [静态方法](span.md#static-methods)
  - [检测方法](span.md#check-methods)

## 静态方法

### 检测方法

#### `Span.isSpan(value: any) => value is Span`

检测 `value` 是否实现了 `Span` 接口。
