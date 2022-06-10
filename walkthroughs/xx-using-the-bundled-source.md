# 使用已编译源

> Commit ID: [a47c35cd953ea6f67cd8c4670a36a8beaee0d9fc](https://github.com/ianstormtaylor/slate/blob/main/docs/walkthroughs/xx-using-the-bundled-source.md)

对大部分人来说，会希望通过 `npm` 安装 Slate，在这种情况下，可以按照常规[安装 Slate](01-installing-slate.md)教程去操作。

但是，如果希望通过简单的添加 `<script>` 标记来安装 Slate 到应用程序，那么本教程会有帮助。为了使“已捆绑”用例更简单，每个版本的 Slate 都附带一个叫做 `slate.js` 的已编译源代码的文件。

要获取 `slate.js` 的副本，请从 npm 下载所需的 Slate 版本：

```text
npm install slate@latest
```

然后在 `node_modules` 文件夹中查找已编译的 `slate.js` 文件：

```text
node_modules/
  slate/
    dist/
      slate.js
      slate.min.js
```

为了方便起见，还包括一个压缩版本的 `slate.min.js`。

在添加 `slate.js` 到页面之前，需要添加 `react`, `react-dom` 和 `react-dom-server` 副本，如下所示：

```html
<script src="./vendor/react.js"></script>
<script src="./vendor/react-dom.js"></script>
<script src="./vendor/react-dom-server.js"></script>
```

这确保了 Slate 不会打包 React 副本，因为这会大大增加应用程序的文件大小。

然后你可以在添加这些之后继续添加 `slate.js`：

```html
<script src="./vendor/slate.js"></script>
```

为了更容易以及快速原型开发，也可以使用 [`unpkg.com`](https://unpkg.com/#/) 分发网络，可以更容易的使用已打包的 npm 模块：

```html
<script src="https://unpkg.com/react/umd/react.production.min.js"></script>
<script src="https://unpkg.com/react-dom/umd/react-dom.production.min.js"></script>
<script src="https://unpkg.com/react-dom/umd/react-dom-server.browser.production.min.js"></script>
<script src="https://unpkg.com/slate/dist/slate.js"></script>
<script src="https://unpkg.com/slate-react/dist/slate-react.js"></script>
```

就是这些，即可准备运行！
