# 安装 Slate

Slate 是分为多个 npm 包的单一代码库，所以你需要使用如下方式安装它：

```text
yarn add slate slate-react
```

你也需要确保安装 Slate 同级依赖：

```text
yarn add react react-dom
```

_注意，如果你更愿意使用预打包的 Slate，你可以 `yarn add slate` 并检索已打包的 `dist/slate.js` 文件！查看[使用已打包源](xx-using-the-bundled-source.md)教程获取更多信息。_

一旦你安装了 Slate，你需要导入它。

```jsx
// 导入 React 依赖。
import React, { useState } from 'react'
// 导入 Slate 编辑器工厂。
import { createEditor } from 'slate'

// 导入 Slate 组件和 React 插件。
import { Slate, Editable, withReact } from 'slate-react'
```

在我们导入之前，让我们先从空 `<App>` 组件开始：

```jsx
// 定义应用程序 。。。
const App = () => {
  return null
}
```

下一步是创建一个新的 `Editor` 对象。我们希望编辑器在渲染过程中保持稳定，因此我们使用 `useState` 钩子[而不是 setter](https://github.com/ianstormtaylor/slate/pull/3925#issuecomment-781179930)：

```jsx
const App = () => {
  // 创建一个不会随着渲染而改变的 Slate 编辑器对象。
  const [editor] = useState(() => withReact(createEditor()))
  return null
}
```

当然我们没有渲染任何东西，所以你不会看到任何改变。

> 如果使用 TypeScript，还需要使用 `ReactEditor` 扩展 `Editor` 并根据 [TypeScript](../concepts/12-typescript.md) 的文档添加注解。下面的示例还包括此示例剩余部分所需要的自定义类型。

```typescript
// 仅 TypeScript 用户添加此代码
import { BaseEditor, Descendant } from 'slate'
import { ReactEditor } from 'slate-react'

type CustomElement = { type: 'paragraph'; children: CustomText[] }
type CustomText = { text: string }

declare module 'slate' {
  interface CustomTypes {
    Editor: BaseEditor & ReactEditor
    Element: CustomElement
    Text: CustomText
  }
}
```

接下来渲染 `<Slate>` 内容提供者。

提供者组件会保持对 Slate 编辑器、插件、值、选择以及发生任何更改的追踪。它**必须**在 `<Editable>` 组件上渲染。但是它也可以使用 `useSlate` 钩子将编辑器状态提供给工具栏、菜单等其它组件。

```jsx
const initialValue = [
  {
    type: 'paragraph',
    children: [{ text: 'A line of text in a paragraph.' }],
  },
]

const App = () => {
  const [editor] = useState(() => withReact(createEditor()))
  // 渲染 Slate 上下文。
  return <Slate editor={editor} initialValue={initialValue} />
}
```

你可以将 `<Slate>` 组件为下面的每个组件提供上下文。

> Slate 提供者的 “value” prop 仅用作 editor.children 的初始状态。如果你的代码依赖替换 editor.children 你应该直接替换它，而不是依赖 “value” prop 去执行此操作。有关更深入的讨论请参阅 [Slate PR 4540](https://github.com/ianstormtaylor/slate/pull/4540)。

这与 `<input>` 或 `<textarea>` 之类的心智模型略有不同，因为富文本文档更加复杂。你通常希望在可编辑内容旁边包含工具栏，实时预览或者其它复杂组件。

通过共享上下文，其它组件可以执行命令，查询编辑器状态等。

好，下一步是渲染 `<Editable>` 组件。组件的作用类似于 `contenteditable`；在任何地方渲染都会为最近的编辑器上下文渲染一个可编辑的富文本文档。

```jsx
const initialValue = [
  {
    type: 'paragraph',
    children: [{ text: 'A line of text in a paragraph.' }],
  },
]

const App = () => {
  const [editor] = useState(() => withReact(createEditor()))
  return (
    // Add the editable component inside the context.
    <Slate editor={editor} initialValue={initialValue}>
      <Editable />
    </Slate>
  )
}
```

这很简单！

这是 Slate 最基本的例子。如果将其渲染到页面上，将会看到带有文本 `A line of text in a paragraph.` 的段落，当你输入时，你会看到文本发生了变化！
