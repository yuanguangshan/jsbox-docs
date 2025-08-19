# $quicklook

通过 $quicklook.open 我们可以通过 iOS 内置的文件查看器打开一些常见的文件格式。

通过 URL:

```js
$quicklook.open({
  url: "",
  handler: function() {
    // Handle dismiss action, optional
  }
})
```

通过 Data:

```js
$quicklook.open({
  type: "jpg",
  data
})
```

预览 image 对象：

```js
$quicklook.open({
  image
})
```

预览纯文本内容：

```js
$quicklook.open({
  text: "Hello, World!"
})
```

预览 JSON 对象：

```js
$quicklook.open({
  json: "{\"a\": [1, 2, true]}"
})
```

预览 HTML 内容：

```js
$quicklook.open({
  html: "<p>HTML</p>"
})
```

预览多个文件（list 内容必须全部为 data 或者全部是链接）：

```js
$quicklook.open({
  list: ["", "", ""]
})
```

请注意，可以传递一个 `type` 用于指定文件类型，该参数是可选的，指定的话会让结果更精确，否则则会猜测文件类型。

---

## 文件内容解读与示例

### 用途说明

`$quicklook` API 提供了一个便捷的方式来**快速预览**各种文件格式，而无需在你的脚本中实现复杂的渲染逻辑。它利用 iOS 系统内置的 Quick Look 框架，能够以原生、高效的方式展示图片、PDF、文本、视频、Office 文档等多种类型的文件。这对于文件管理、内容预览或调试脚本生成的文件非常有用。

### 核心方法：`$quicklook.open(options)`

这是 `$quicklook` 模块的唯一核心方法，用于打开一个模态的 Quick Look 预览器。`options` 对象定义了要预览的内容及其类型。

#### 1. 内容来源

你需要在 `options` 对象中提供**且仅提供一个**以下属性来指定要预览的内容：

-   **`url`**: 预览一个远程文件或本地文件的 URL。例如 `"https://example.com/document.pdf"` 或 `"local://assets/video.mp4"`。
-   **`data`**: 预览一个 `$data` 对象（二进制数据）。**当使用 `data` 时，必须同时提供 `type` 属性来指定文件类型**（如 `"pdf"`, `"docx"`, `"mp4"` 等），否则系统无法识别文件格式。
-   **`image`**: 预览一个 `$image` 对象。
-   **`text`**: 预览纯文本内容。
-   **`json`**: 预览 JSON 字符串。系统会对其进行格式化显示，方便调试。
-   **`html`**: 预览 HTML 字符串。
-   **`list`**: 预览多个文件。`list` 中的每个元素可以是 URL 字符串或 `$data` 对象。如果包含 `$data` 对象，同样需要为每个 `$data` 对象指定 `type`。

#### 2. 可选参数

-   **`handler`**: 预览器关闭时的回调函数。你可以用它来执行一些清理工作或在用户关闭预览后继续脚本流程。

### 示例代码：一个多功能文件预览器

下面的示例将创建一个简单的 UI，包含多个按钮，演示如何使用 `$quicklook.open()` 预览不同类型的内容。

```javascript
$ui.render({
  props: {
    title: "QuickLook 示例"
  },
  views: [
    {
      type: "button",
      props: { title: "预览图片" },
      layout: make => make.top.left.right.inset(20).height.equalTo(40),
      events: {
        tapped: () => {
          // 假设 assets 目录下有 sample.png
          const image = $image("assets/sample.png"); 
          if (image) {
            $quicklook.open({ image: image });
          } else {
            $ui.toast("图片文件不存在。");
          }
        }
      }
    },
    {
      type: "button",
      props: { title: "预览 PDF 文件" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.right.inset(20).height.equalTo(40),
      events: {
        tapped: () => {
          // 假设 assets 目录下有 sample.pdf
          const pdfData = $file.read("assets/sample.pdf");
          if (pdfData) {
            $quicklook.open({ data: pdfData, type: "pdf" }); // 必须指定 type
          } else {
            $ui.toast("PDF 文件不存在。");
          }
        }
      }
    },
    {
      type: "button",
      props: { title: "预览纯文本" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.right.inset(20).height.equalTo(40),
      events: {
        tapped: () => {
          $quicklook.open({ text: "Hello, World!\n\nThis is a plain text preview." });
        }
      }
    },
    {
      type: "button",
      props: { title: "预览 JSON 数据" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.right.inset(20).height.equalTo(40),
      events: {
        tapped: () => {
          const sampleJson = {
            name: "JSBox",
            version: "3.0.0",
            features: ["UI", "Network", "File"]
          };
          $quicklook.open({ json: JSON.stringify(sampleJson, null, 2) }); // 格式化后预览
        }
      }
    },
    {
      type: "button",
      props: { title: "预览 HTML 内容" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.right.inset(20).height.equalTo(40),
      events: {
        tapped: () => {
          const sampleHtml = "<h1>Hello HTML!</h1><p style=\"color: blue;\">This is a <i>simple</i> HTML preview.</p>";
          $quicklook.open({ html: sampleHtml });
        }
      }
    }
  ]
});
```

**代码解读**：

这个示例展示了 `$quicklook.open()` 如何根据传入的 `options` 对象中的不同属性来预览各种类型的内容。特别注意，当预览 `$data` 对象时，`type` 属性是必不可少的，它告诉 Quick Look 应该如何解释这些二进制数据。

`$quicklook` 是一个非常实用的工具，它能让你在脚本中快速为用户提供文件内容的预览，而无需自己实现复杂的渲染逻辑，极大地提升了开发效率和用户体验。 
