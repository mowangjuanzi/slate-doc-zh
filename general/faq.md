# 常见问题

人们对 Slate 的一些常见问题：

- [为什么内容粘贴为纯文本？](faq.md#why-is-content-is-pasted-as-plaintext)
- [Slate 支持哪些浏览器和设备？](faq.md#what-browsers-and-devices-does-slate-support)

## Why is content pasted as plain text?

Slate 的核心原则之一是，与大多数编辑器不同，**不会**为正在编辑的内容规定指定的“模式”。这意味着 Slate 核心没有“引用”或者“粗体格式”的概念。

大部分情况会提升灵活性而没有多少缺点，但在某些情况下必须做更多的工作。粘贴就是其中之一。

Since Slate knows nothing about your domain, it can't know how to parse pasted HTML content \(or other content\). So, by default whenever a user pastes content into a Slate editor, it will parse it as plain text. If you want it to be smarter about pasted content, you need to override the `insert_data` command and deserialize the `DataTransfer` object's `text/html` data as you wish.

## What browsers and devices does Slate support?

Slate's goal is to support all the modern browsers on both desktop and mobile devices.

Slate is in beta and is community-driven and so its support is not as robust as it could be.

On the desktop, it's currently tested against the latest few versions of Chrome, Edge, Firefox and Safari on desktops. And it does not work in Internet Explorer.

On mobile, iOS devices are supported but not regularly tested. Chrome on Android was until recently unsupported except for in older versions of Slate (0.47) but has recently been added. For clarity, due to the differences in Android's support of the `beforeInput` event, Android input uses compositions and mutations which is different from other browsers. This means that Android support progresses separately from other browsers and due to it being new, may have more bugs.

If you want to add or improve browser or device support, we'd love for you to submit a pull request! Or in the case of incompatible browsers, build a plugin.

For older browsers, such as IE11, a lot of the now standard native APIs aren't available. Slate's position on this is that it is up to the user to bring polyfills \(like [https://polyfill.io](https://polyfill.io)\) when needed for things like `el.closest`, etc. Otherwise we'd have to bundle and maintain lots of polyfills that others may not even need in the first place. For clarity, Slate makes no guarantees that it will work with older browsers, even with polyfills and at present, there are still unresolved issues with IE11.
