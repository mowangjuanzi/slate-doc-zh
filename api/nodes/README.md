# Node 类型 API

> Commit ID: [7d9d25e1790c6557e6ff2072f79f7904736aec65](https://github.com/ianstormtaylor/slate/blob/main/docs/api/nodes/README.md)

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
