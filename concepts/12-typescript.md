# 使用 TypeScript

> Commit ID: [c8c75e9e2d6317383d2517c26110b971dec027fe](https://github.com/ianstormtaylor/slate/blob/main/docs/concepts/12-typescript.md)

Slate 支持 Slate 文档模型类型（即一组自定义 `Editor`、`Element`、`Text` 类型）。如果需要支持多个文档模型，参阅多文档模型章节。

**警告：** 使用 TypeScript 时必须定义 `CustomTypes`, 注解 `useState` 并注解编辑器的初始状态，否则 Slate 会显示类型错误。

## 从 0.47.x 迁移

从 0.47.x 迁移时，首先阅读以下教程。同时还要记住这些常见的迁移问题：

- 当引用 `node.type` 时，可能会看到错误 `Property 'type' does not exist on type 'Node'`。要解决此问题，需要添加类似 `Element.isElement(node) && node.type === 'paragraph'` 之类的代码。这很有必要，因为 `Node` 可以是 `Element` 或者 `Text` 并且 `Text` 没有 `type` 属性。
- 为 `Editor` 定义 CustomType 时要小心。确保 `Editor` 定义的 CustomType 是 `BaseEditor & ...`。不是 `Editor & ...`

## 定义 `Editor`, `Element` and `Text` 类型

要定义自定义 `Element` 或者 `Text` 类型，要像这样在 `slate` 模块中扩展 `CustomTypes` 接口。

```typescript
// 此示例适用于使用 `ReactEditor` 和 `HistoryEditor` 的编辑器
import { BaseEditor } from 'slate'
import { ReactEditor } from 'slate-react'
import { HistoryEditor } from 'slate-history'

type CustomElement = { type: 'paragraph'; children: CustomText[] }
type CustomText = { text: string; bold?: true }

declare module 'slate' {
  interface CustomTypes {
    Editor: BaseEditor & ReactEditor & HistoryEditor
    Element: CustomElement
    Text: CustomText
  }
}
```

## 编辑器中的注解

使用 `Descendant[]` 注解编辑器的初始值.

```tsx
import React, { useState } from 'react'
import { createEditor, Descendant } from 'slate'
import { Slate, Editable, withReact } from 'slate-react'

const initialValue: Descendant[] = [
  {
    type: 'paragraph',
    children: [{ text: 'A line of text in a paragraph.' }],
  },
]

const App = () => {
  const [editor] = useState(() => withReact(createEditor()))

  return (
    <Slate editor={editor} value={initialValue}>
      <Editable />
    </Slate>
  )
}
```

## `Element` 和 `Text` 类型最佳实践

虽然可以直接在 `CustomTypes` 接口中定义类型，但最佳做法是分别定义和导出每种类型以便可以引用单个类型，例如 `ParagraphElement`.

使用最佳实践，自定义类型看起来就像是这样：

```typescript
// 此示例适用于使用 `ReactEditor` 和 `HistoryEditor` 的编辑器
import { BaseEditor } from 'slate'
import { ReactEditor } from 'slate-react'
import { HistoryEditor } from 'slate-history'

export type CustomEditor = BaseEditor & ReactEditor & HistoryEditor

export type ParagraphElement = {
  type: 'paragraph'
  children: CustomText[]
}

export type HeadingElement = {
  type: 'heading'
  level: number
  children: CustomText[]
}

export type CustomElement = ParagraphElement | HeadingElement

export type FormattedText = { text: string; bold?: true }

export type CustomText = FormattedText

declare module 'slate' {
  interface CustomTypes {
    Editor: CustomEditor
    Element: CustomElement
    Text: CustomText
  }
}
```

在本示例中 `CustomText` 等于 `FormattedText`，但在真正的编辑器中，可能有更多类型的文本，像是不允许格式化的代码块文本。

## 为什么类型定义不寻常

因为经常会问这个问题，所以本节解释一下为什么 Slate 的类型定义是非典型。

Slate 需要支持称为类型推断的功能，该功能可以在联合类型中使用（例如 `ParagraphElement | HeadingElement`）。这允许用户使用的类型减少。如果在块内部显示类似 `if (node.type === 'paragraph') { ... }` 类似的代码，会将节点的类型减少到 `ParagraphElement`。

Slate 还需要一种方式允许开发者将自定义类型放入 Slate。这是通过合并声明来完成的，这是 `interface` 的功能。

Slate 结合联合类型和接口来使用这两个功能。

参阅[提案：添加自定义 TypeScript 类型到 Slate](https://github.com/ianstormtaylor/slate/issues/3725) 获取更多信息。

## 多文档模型

目前， Slate 一次支持单个文档模型类型。例如，它不能支持具有不同文档架构的两个不同的富文本编辑器。

Slate 的 TypeScript 支持是这样设计的，因为文档模式类型总比没有强。目标是支持为多个编辑器定义类型，目前 Slate 的创建者正在构建一个正在进行的 PR。

支持多个文档模型的一种解决办法是在单独的包中创建每个编辑器，然后导入。这尚未经过测试，但应该可以工作。

## 扩展其它类型

当前还支持扩展其它类型，但这些类型并没有像上面记录的那样经过彻底的测试：

- `Selection`
- `Range`
- `Point`

可以随意扩展这些类型，但扩展这些类型应该实验性的，请在 GitHub issues 中报告 bug。

## TypeScript 示例

有关如何使用类型的示例，请参阅 Slate 存储库中的 `packages/slate-react/src/custom-types.ts`。
