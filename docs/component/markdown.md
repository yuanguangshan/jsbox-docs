# type: "markdown"

用来显示 Markdown 格式的文本，当你需要显示富文本内容时，除了使用 HTML 以外，Markdown 是一个不错的选择：

```js
$ui.render({
  views: [
    {
      type: "markdown",
      props: {
        content: "# Hello, *World!",
        style: // optional, custom style sheet
        `
        body {
          background: #f0f0f0;
        }
        `
      },
      layout: $layout.fill
    }
  ]
})
```

属性 | 类型 | 读写 | 说明
---|---|---|---
webView | $view | r | webView
content | string | rw | 内容
scrollEnabled | bool | rw | 是否滚动
style | string | rw | 自定义样式

---

## 文件内容解读与示例

### 组件用途

`markdown` 组件是一个专门用于解析和显示 [Markdown](https://www.markdownguide.org/basic-syntax/) 格式文本的视图。对于需要展示富文本（如标题、列表、粗体、链接等）的场景，使用 `markdown` 组件通常比手动拼接 `styledText` 或编写完整的 HTML 要简单和快捷得多。它非常适合用于制作应用的“关于”页面、帮助文档、文章阅读器等。

### 核心概念

1.  **工作原理**: `markdown` 组件的本质是一个预先配置好的 `webView`（网页视图）。当你给它提供 Markdown 文本时，JSBox 会在内部将其转换为 HTML，然后在这个 `webView` 中显示出来。理解这一点有助于你更好地掌握它的 `style` 属性。

2.  **`content` 属性**: 这是最核心的属性，它的值就是你想要显示的、未经处理的 Markdown 原文字符串。

3.  **`style` 属性 (自定义样式)**: 这是 `markdown` 组件最强大的功能之一。它允许你提供一段标准的 **CSS (层叠样式表)** 字符串，来完全自定义渲染后的 HTML 的外观。你可以修改背景色、字体、字号、标题样式、链接颜色、代码块样式等几乎所有视觉元素。

### 示例代码：渲染一篇带自定义样式的文章

下面的示例将演示如何渲染一篇包含多种 Markdown 元素的文章，并使用 `style` 属性为其应用一套自定义的视觉主题。

```javascript
// 1. 准备 Markdown 内容
const markdownContent = `
# 关于 JSBox

JSBox 是一个创造工具，它提供了强大的 API，让你可以通过 JavaScript 构建自己的原生应用。

## 核心特性

* **易用**: 简洁的 API 设计，上手快。
* **强大**: 支持原生 UI、网络请求、数据存储等。
* **灵活**: 可通过[社区](https://jsboxbbs.com/)获取和分享脚本。

## 代码示例

\`\`\`javascript
// 弹出一个简单的提示
$ui.alert("Hello, JSBox!");
\`\`\`
`;

// 2. 准备自定义 CSS 样式
const customStyle = `
body {
  font-family: -apple-system, sans-serif;
  background-color: #f8f8f8;
  color: #333;
  padding: 15px;
}
h1 {
  color: #007aff;
  border-bottom: 2px solid #007aff;
  padding-bottom: 5px;
}
pre {
  background-color: #282c34;
  color: #abb2bf;
  padding: 10px;
  border-radius: 5px;
}
`;

// 3. 渲染 markdown 视图
$ui.render({
  props: {
    title: "Markdown 组件示例"
  },
  views: [
    {
      type: "markdown",
      props: {
        content: markdownContent,
        style: customStyle
      },
      layout: $layout.fill
    }
  ]
});
```

**代码解读**：

1.  我们在 `markdownContent` 中定义了各种 Markdown 元素，包括一级标题 (`#`)、二级标题 (`##`)、无序列表 (`*`)、链接和代码块 (\`\`\`)。
2.  在 `customStyle` 中，我们编写了标准的 CSS 规则：
    - `body`: 设置了页面的背景色、默认字体和内边距。
    - `h1`: 将一级标题的颜色改为了蓝色，并添加了下划线。
    - `pre`: 为代码块（在 HTML 中通常是 `<pre>` 标签）设置了深色的背景和浅色的文字，模拟了一个常见的代码编辑器主题。
3.  最后，我们将这两个字符串分别传给 `content` 和 `style` 属性，`markdown` 组件便会为我们呈现出一个经过精美排版的富文本页面。

`markdown` 组件是展示格式化文本的绝佳选择，它在 Markdown 的简洁性与 CSS 的灵活性之间取得了完美的平衡. 
