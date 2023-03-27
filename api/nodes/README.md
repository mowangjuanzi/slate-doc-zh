# Node 类型 API

`Node` 联合类型代表 Slate 文档树中出现的所有不同节点类型。

```typescript
type Node = Editor | Element | Text

type Descendant = Element | Text
type Ancestor = Editor | Element
```

- [Node](./node.md)
- [NodeEntry](./node-entry.md)
- [Editor](./editor.md)
- [Element](./element.md)
- [Text](./text.md)
