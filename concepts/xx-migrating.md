# 迁移

Slate 从早期版本迁移到 `0.50.x` 版本并不是一项简单的任务。从头开始考虑整个框架。这导致了一组**很**好的抽象，将会编写更少的代码。但迁移过程并不简单。

强烈建议阅读本教程后能通读[演示](../walkthroughs/01-installing-slate.md)和其它[概念](01-interfaces.md)以了解如何应用所有新概念。

## 主要区别

下面从架构的角度阐述 Slate `0.50.x` 版本的_主要_区别。

### JSON！

数据模型现在由简单的 JSON 对象组成。之前使用 [Immutable.js](https://immutable-js.github.io/immutable-js/) 数据结构。这是巨大的改变，而且解锁了很多东西。希望也能提高使用 Slate 时的平均性能。也能让新手更加容易上手。这将是一个非常巨大的迁移变化，但这是值得的。

### 接口

数据模型基于接口。之前每个模型都是类的实例。现在数据不单单是对象了，而且 Slate 还期待对象实现了接口。因此过去存在于 `node.data` 的自定义属性可以可以存在于节点的顶级。

### 命名空间

许多助手作为命名空间中的助手函数集合公开。例如 `Node.get(root, path)` 或者 `Range.isCollapsed(range)`。这使得代码更加清晰，因为始终可以快速查看正在使用的接口。

### TypeScript

代码库现在使用 TypeScript。使用纯 JSON 作为数据模型，并且使用基于接口的 API 是迁移到 TypeScript 之后变得更容易的两件事。可以不使用它，但是在使用 API 时会如果使用它会获得更多的安全性。（如果使用 VS Code，无论如何都会获得很好的自动补全！）

### 概念更少

接口和命令的数量已减少。 之前 `Selection`, `Annotation` 和 `Decoration` 曾经都是单独的类。现在它们只是实现 `Range` 接口的对象。之前 `Block` 和 `Inline` 是分开的；现在它们是实现 `Element` 接口的对象。之前是 `Document` 和 `Value`，但现在顶级 `Editor` 包含文档本身的子节点。

命令的数量也减少了。之前对每种类型都有命令，比如 `insertText`、`insertTextAtRange`、`insertTextAtPath`。这些已经合并到一组更小的可定制的命令中，例如 `insertText` 可以在 `at: Path | Range | Point` 中使用。

### 包更少

为了降低维护负担，并且 Slate 核心包的抽象和 API 使事情更加容易，包的数量已经减少。像 `slate-plain-serializer`、`slate-base64-serializer`等等。已被删除，如果需要可以在用户层轻松实现。甚至 `slate-html-deserializer` 现在也可以在用户层实现（在 ~10 LOC 中利用 `slate-hyperscript`）。以及像 `slate-dev-environment`、`slate-dev-test-utils`等等内部包不再公开，因为它们是实现细节。

### 命令

引入了新的“命令”概念。（旧“命令”现在成为“转化”。） This new concept expresses the semantic intent of a user editing the document. And they allow for the right abstraction to tap into user behaviors—for example to change what happens when a user presses enter, or backspace, etc. Instead of using `keydown` events you should likely override command behaviors instead.

命令通过调用 `editor.*` 核心函数触发。 And they travel through a middleware-like stack, but built from composed functions. Any plugin can override the behaviors to augment an editor.

### 插件

插件现在是简单函数，可以增强接收到的 `Editor` 对象并再次将其返回。 For example, they can augment the command execution by composing the `editor.exec` function or listen to operations by composing `editor.apply`. Previously they relied on a custom middleware stack, and they were just bags of handlers that got merged onto an editor. Now we're using plain old function composition \(aka wrapping\) instead.

### 元素

块级和内联现在是运行时选择。 Previously it was baked into the data model with the `object: 'block'` or `object: 'inline'` attributes. Now, it checks whether an "element" is inline or not at runtime. For example, you might check to see that `element.type === 'link'` and treat it as inline.

### 更加 React 化

渲染和事件处理不再是插件关注的事。 Previously plugins had full control over the rendering and event-handling logic in the editor. This creates a bad incentive to start putting **all** rendering logic in plugins, which puts Slate in a position of being a wrapper around all of React, which is very hard to do well. Instead, the new architecture has plugins focused purely on the richtext aspects, and leaves the rendering and event handling aspects to React.

### 上下文

之前 `<Editor>` 组件具有“控制器”对象和 `contenteditable` DOM 元素两个指责。 In the new version, there is a new `<Slate>` context provider and a simpler `<Editable>` `contenteditable`-like component. By putting the `<Slate>` provider higher up in your component tree, you can share the editor directly with toolbars, buttons, etc. using the `useSlate` hook.

### Hook

除了 `useSlate` hook，还有一些其他 hook。 For example the `useSelected` and `useFocused` hooks help with knowing when to render selected states \(often for void nodes\). And since they use React's Context API they will automatically re-render when their state changes.

### `beforeinput`

现在几乎只使用 `beforeinput` 事件。 Instead of relying on a series of shims and the quirks of React synthetic events, we're now using the standardized `beforeinput` event as our baseline. It is fully supported in Safari and Chrome, will soon be supported in the new Chromium-based Edge, and is currently being worked on in Firefox. In the meantime there are a few patches to make Firefox work. Internet Explorer is no longer supported in core out of the box.

### 无历史

现在核心历史逻辑最终提取到一个独立插件中。 This makes it much easier for people to implement their own custom history behaviors. And it ensures that plugins have enough control to augment the editor in complex ways, because the history requires it.

### 无标记

标记已经从 Slate 数据模型中移除。 Now that we have the ability to define custom properties right on the nodes themselves, you can model marks as custom properties of text nodes. For example bold can be modelled simply as a `bold: true` property.

### 无注解

同样，注解已经从 Slate 核心中删除。 They can be fully implemented now in userland by defining custom operations and rendering annotated ranges using decorations. But most cases should be using custom text node properties or decorations anyways. There were not that many use cases that benefited from annotations.

## 减少体积

目标之一是简化 Slate 中的许多逻辑，使其易于维护和迭代。 This was done by refactoring to better base abstractions that can be built on, by leveraging modern DOM APIs, and by migrating to simpler React patterns.

To give you a sense for the change in total lines of code:

```text
slate                       8,436  ->  3,958  (47%)
slate-react                 3,905  ->  1,954  (50%)

slate-base64-serializer        38  ->      0
slate-dev-benchmark           340  ->      0
slate-dev-environment         102  ->      0
slate-dev-test-utils           44  ->      0
slate-history                   0  ->    211
slate-hotkeys                  62  ->      0
slate-html-serializer         253  ->      0
slate-hyperscript             447  ->    345
slate-plain-serializer         56  ->      0
slate-prop-types               62  ->      0
slate-react-placeholder        62  ->      0

total                      13,807  ->  6,468  (47%)
```

这是一个很大的不同！这甚至不包括在此关系中剥离的依赖关系。
