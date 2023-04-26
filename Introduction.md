# 简介

> 本项目地址：https://github.com/mowangjuanzi/slate-doc-zh
> 
> 英文官网项目地址：https://github.com/ianstormtaylor/slate
>
> 目前的版本是：[slate@0.94.0](https://github.com/ianstormtaylor/slate/releases/tag/slate%400.94.0)

[Slate](http://slatejs.org) 是_完全_可定制的框架，用于构建富文本编辑器。

Slate 可以构建丰富直观的编辑器，像 [Medium](https://medium.com/)、[Dropbox Paper](https://www.dropbox.com/paper)、[Google Docs](https://www.google.com/docs/about/) —— 这些正成为网络应用程序的中流砥柱 —— 不会让代码库陷入复杂。

因为它的所有逻辑都是用一系列插件实现的，所以无论_是否_是“核心”都不会受到限制。你可以想象它是基于 [React](https://facebook.github.io/react/) `contenteditable` 的可拔插实现。它的灵感来自于 [Draft.js](https://facebook.github.io/draft-js/)、[Prosemirror](http://prosemirror.net/)、[Quill](http://quilljs.com/) 等库.

> 🤖 **Slate 目前处于测试阶段**。核心 API 现在可以使用，但你也可以为高级用例 PR 修复。一些 API 没有“最终确定”且会随着时间的推移找到更好的方案后进行（破坏性）更新。

## 为什么？

为什么创建 Slate？ 嗯。。。_（注意：本节有[作者](https://github.com/ianstormtaylor)的一些意见！）_

在创建 Slate 之前，我尝试了很多富文本类库 —— [**Draft.js**](https://facebook.github.io/draft-js/)、[**Prosemirror**](http://prosemirror.net/)、[**Quill**](http://quilljs.com/)等。我发现简单的示例运行起来很容易，一旦你尝试构建像 [Medium](https://medium.com/)、[Dropbox Paper](https://www.dropbox.com/paper)、[Google Docs](https://www.google.com/docs/about/)，你就会遇到更深层次的问题。。。

- **编辑器的“模式”是硬编码的，难以修改。**像是粗体斜体是开箱即用的，但是评论，嵌入，甚至更多特定领域需要的呢？
- **以编程的方式转换文档非常复杂。**作为用户编写可能会奏效，但是对构建高级行为至关重要的程序化更改是不必要的复杂。
- **似乎是后来才想到序列化为 HTML、Markdown等。**像是将文档转换为 HTML 或者 Markdown 这种简单的事情都需要编写大量的模板代码，这似乎是非常常见的用例。
- **重新做视图层似乎效率低下且受到限制。**大部分编辑器都使用了自己的视图，而不是使用像 React 这样现有的技术，所以你必须学习一个新的有“问题”的全新系统。
- **没有预先设计协作编辑。**通常编辑器的内部数据表示无法用于实时、协作编辑用例，除非是重写编辑器。
- **存储库巨大，并非小而可复用。**许多编辑器的代码库通常没有公开可用于开发人员复用的内部工具，导致不得不重新发明轮子。
- **不能构建复杂嵌套文档。**许多编辑器都是围绕“内部层次不多的”文档进行设计，使得表格、嵌入、字幕等内容难以实现，甚至不可能实现。

当然并不是所有的编辑器都会有这些问题，但是你尝试使用其它编辑器，你可能会碰到类似的问题。为了绕过它们的 API 限制且实现你追求的用户体验，你必须借助非常 hack 的方式。有些经验是完全不可能实现的。

如果听起来很熟悉，你可能会喜欢 Slate。

这让我想到了如何 Slate 如何解决这些问题。。。

## 原则

Slate 尝试解决“[为什么？](Introduction.md#为什么？)”提出的问题。有几个原则：

1. **插件是一等公民。**Slate 最重要的部分是插件是一等公民。这意味着你_完全_可以自定义编辑体验，构建像 Medium 或 Dropbox 一样复杂的编辑器，而无需猜测类库的呈现。
2. **无模式核心。**Slate 的核心逻辑对要编写的数据模式几乎没有假设，这意味着当你需要编写复杂用例时，不会被任何库中的任何假设阻碍。
3. **嵌套文档模型。**Slate 使用的文档模型是嵌套递归树，就像 DOM 一样。这意味着可以为高级用例创建复杂组件，如表格或者嵌套引用。但是也可以通过单个层级来保持简单。
4. **与 DOM 平等。**Slate 的数据模型基于 DOM —— 文档是嵌套树，用于选择和范围且公开所有标准事件处理程序。这意味表格或嵌套引用之类的高级行为是可能的。在 DOM 中执行的任何操作，几乎都可以在 Slate 中执行。
5. **命令直观。**Slate 文档使用“命令”编辑，它被设计为高级且非常直观的读写，因此自定义功能必须尽可能的直观。这极大的提高了你推断代码的能力。
6. **协作式数据模型。**Slate 使用的数据模型 —— 特别是如何将操作应用于文档 —— 设计为允许在顶层进行协作编辑，所以当决定让编辑器进行协作时无需重新考虑任何内容。
7. **清晰“核心”界限。**借助插件优先架构和无模式核心，“核心”和“自定义”之间的界限变得更加清晰，这意味着核心体验不会在边缘情况下陷入困境。

## Demo

查看所有示例的[**在线 demo**](http://slatejs.org)！

## 示例

要了解如何使用 Slate，请查看这些示例：

- [**纯文本**](https://www.slatejs.org/examples/plaintext) —— 展示最基本的情况：美化过的 `<textarea>`。
- [**富文本**](https://www.slatejs.org/examples/richtext) —— 展示你期望从基础编辑器中获得的行为。
- [**Markdown 预览**](https://www.slatejs.org/examples/markdown-preview) — 展示如何为类 Markdown 结构体添加快捷键行处理程序。
- [**内联**](https://www.slatejs.org/examples/inlines) —— 展示如何使用关联数据在内联节点文本换行。
- [**图像**](https://www.slatejs.org/examples/images) —— 展示如何使用 void（无文本）节点添加图像。
- [**悬停工具栏**](https://www.slatejs.org/examples/hovering-toolbar) —— 展示如何实现上下文悬停菜单。
- [**表格**](https://www.slatejs.org/examples/tables) —— 展示如何嵌套块以渲染更多高级组件。
- [**粘贴 HTML**](https://www.slatejs.org/examples/paste-html) —— 展示如何使用 HTML 序列化程序处理粘贴的 HTML。
- [**提及**](https://www.slatejs.org/examples/mentions) —— 展示如何使用行内 void 节点实现简单的 @ 提及。

每个示例都包含代码实现的 **View Source** 链接。我们还有[其它示例](https://github.com/ianstormtaylor/slate/tree/master/site/examples)。

如果你对展示的常见示例有想法，请 PR！

## 文档

如果你是第一次使用 Slate，请查看[入门](http://docs.slatejs.org/walkthroughs/01-installing-slate)演示和[概念](http://docs.slatejs.org/concepts)去熟悉 Slate 的架构和思维模型。

- [**演示**](http://docs.slatejs.org/walkthroughs)
- [**概念**](http://docs.slatejs.org/concepts)
- [**常见问题**](http://docs.slatejs.org/general/faq)
- [**资源**](http://docs.slatejs.org/general/resources)

如果还不够，你可以随时[阅读源码](https://github.com/ianstormtaylor/slate/tree/master/packages)，它包含了大量注释。

也有将文档翻译成其它语言：

- [中文](https://doodlewind.github.io/slate-doc-cn/)

如果你正在维护翻译，请随时在此提交 PR！

## 贡献！

超级欢迎所有贡献！请查看[贡献说明](general/contributing.md)获取更多信息！

Slate 是 [MIT 许可](https://github.com/ianstormtaylor/slate/tree/f6bfe034d707693488c38da77537fd36cb8856cf/License.md)。
