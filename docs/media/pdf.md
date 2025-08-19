> 此接口用于创建 PDF 文档

# $pdf.make(object)

通过 URL 或者 html 来创建 PDF 文档：

```js
$pdf.make({
  html: "<p>Hello, World!</p><h1 style='background-color: red;'>xTeko</h1>",
  handler: function(resp) {
    const data = resp.data;
    if (data) {
      $share.sheet(["sample.pdf", data])
    }
  }
})
```

也可以使用图片来创建 PDF 文件：

```js
let {data} = await $pdf.make({"images": images});
```

支持通过 `pageSize` 来设置页面大小，例如：

```js
$pdf.make({
  url: "https://github.com",
  pageSize: $pageSize.A5,
  handler: function(resp) {
    const data = resp.data;
    if (data) {
      $share.sheet(["sample.pdf", data])
    }
  }
})
```

`$pageSize` 支持 A0 ~ A10, B0 ~ B10, C0 ~ C10 等取值，请参考：http://en.wikipedia.org/wiki/Paper_size

# $pdf.toImages(data)

将 PDF 文档渲染成一个 image 数组：

```js
const images = $pdf.toImages(pdf);
```

# $pdf.toImage(data)

将 PDF 文档渲染成一个 image 对象：

```js
const image = $pdf.toImage(pdf);
```

---

## 文件内容解读与示例

### 用途说明

`$pdf` API 提供了在 JSBox 脚本中**创建和转换 PDF 文档**的强大功能。它允许你将 HTML 内容、网页或图片集合转换为 PDF 文件，也能将现有的 PDF 文件渲染成图片。这对于生成报告、保存网页内容、创建电子书或处理文档非常有用。

### 核心方法：`$pdf.make(options)` - 创建 PDF

这是生成 PDF 文档的核心方法。`options` 对象定义了 PDF 的内容来源和属性。

#### 1. 内容来源

-   **`html`**: 将 HTML 字符串转换为 PDF。你可以使用完整的 HTML 结构，包括 CSS 样式。
-   **`url`**: 将一个网页（通过其 URL）转换为 PDF。这对于保存网页内容为离线文档非常方便。
-   **`images`**: 将一个 `$image` 对象的数组转换为多页 PDF。每张图片将成为 PDF 的一页。

#### 2. 页面设置

-   **`pageSize`**: 使用 `$pageSize` 常量（如 `$pageSize.A4`, `$pageSize.Letter`）来定义 PDF 页面的尺寸。这确保了生成的 PDF 符合标准的纸张大小。

#### 3. 异步处理

-   `$pdf.make()` 是一个异步操作，结果通过 `handler` 回调函数或 `await` Promise 返回。返回的结果是一个 `$data` 对象，代表生成的 PDF 文件内容。

**示例**：将 HTML 转换为 PDF 并分享

```javascript
const htmlContent = `
  <h1>JSBox PDF 示例</h1>
  <p>这是一个由 JSBox 脚本生成的 PDF 文档。</p>
  <img src="https://picsum.photos/200/100" style="width: 100%; height: auto;">
  <p style="color: blue;">感谢您的阅读！</p>
`;

$pdf.make({
  html: htmlContent,
  pageSize: $pageSize.A4, // 设置页面大小为 A4
  handler: (resp) => {
    if (resp.data) {
      $ui.alert({
        title: "PDF 生成成功",
        message: "是否分享？",
        actions: [
          { title: "分享", handler: () => $share.sheet(["sample.pdf", resp.data]) },
          { title: "取消" }
        ]
      });
    } else {
      $ui.alert("PDF 生成失败！");
    }
  }
});
```

### PDF 转换为图片：`$pdf.toImages(data)` 和 `$pdf.toImage(data)`

-   **`$pdf.toImages(data)`**: 将一个 PDF 的 `$data` 对象渲染成一个 `$image` 对象的数组。PDF 的每一页都会被转换为数组中的一张图片。
-   **`$pdf.toImage(data)`**: 将一个 PDF 的 `$data` 对象渲染成一个单独的 `$image` 对象（通常是 PDF 的第一页）。

**示例**：将 PDF 转换为图片并显示第一页

```javascript
async function convertPdfToImage() {
  // 假设你有一个本地的 PDF 文件，例如 "assets/document.pdf"
  const pdfData = $file.read("assets/sample.pdf"); // 确保 sample.pdf 存在

  if (pdfData) {
    const images = $pdf.toImages(pdfData); // 将 PDF 转换为图片数组
    if (images && images.length > 0) {
      $ui.alert({ title: "PDF 第一页", image: images[0] }); // 显示第一页
    } else {
      $ui.alert("PDF 转换图片失败或 PDF 为空。");
    }
  } else {
    $ui.alert("未找到 PDF 文件。");
  }
}
// convertPdfToImage();
```

### 总结

`$pdf` API 为你的 JSBox 脚本提供了强大的文档处理能力。无论是将动态内容生成为标准 PDF，还是将现有 PDF 转换为图片进行预览，它都提供了便捷的接口。这对于自动化报告生成、内容归档等场景非常有用。 
