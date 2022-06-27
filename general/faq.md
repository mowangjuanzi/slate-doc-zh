# 常见问题

人们对 Slate 的一些常见问题：

- [为什么内容粘贴为纯文本？](faq.md#为什么内容粘贴为纯文本？)
- [Slate 支持哪些浏览器和设备？](faq.md#Slate 支持哪些浏览器和设备？)

## 为什么内容粘贴为纯文本？

Slate 的核心原则之一是，与大多数编辑器不同，**不会**为正在编辑的内容规定指定的“模式”。这意味着 Slate 核心没有“引用”或者“粗体格式”的概念。

大部分情况会提升灵活性而没有多少缺点，但在某些情况下必须做更多的工作。粘贴就是其中之一。

由于 Slate 对域一无所知，因此不知道如何解析粘贴的 HTML（或者其它内容）。所以默认情况下，每当内容粘贴到 Slate 编辑器中时，都会将其解析为纯文本。如果想要对粘贴的内容更智能，则需要重写 `insert_data` 命令和反序列化 `DataTransfer` 对象的 `text/html` 数据。

## Slate 支持哪些浏览器和设备？

Slate 的目标是支持桌面和移动设备上的所有现代化浏览器。

Slate 处于测试阶段且由社区驱动，因此它的支持并没有这么强大。

在桌面上，仅对当前 Chrome、 Edge、 Firefox、 Safari 最新的几个版本进行了测试，并不支持 Internet Explorer。

在移动设备上，支持 IOS 设备且不定期测试。 Chrome on Android was until recently unsupported except for in older versions of Slate (0.47) but has recently been added. For clarity, due to the differences in Android's support of the `beforeInput` event, Android input uses compositions and mutations which is different from other browsers. This means that Android support progresses separately from other browsers and due to it being new, may have more bugs.

如果想添加或者提升浏览器或者设备支持，请提交 PR！或者在浏览器不兼容时构建一个插件。

对于 IE11 之类的旧浏览器，一些新原生标准 API 不能用。 Slate's position on this is that it is up to the user to bring polyfills \(like [https://polyfill.io](https://polyfill.io)\) when needed for things like `el.closest`, etc. Otherwise we'd have to bundle and maintain lots of polyfills that others may not even need in the first place. For clarity, Slate makes no guarantees that it will work with older browsers, even with polyfills and at present, there are still unresolved issues with IE11.
