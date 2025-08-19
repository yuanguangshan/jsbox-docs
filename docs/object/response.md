# response

`response` 是 http 请求里面返回的内容，例如：

```js
$http.get({
  url: "",
  handler: function(resp) {
    const response = resp.response;
  }
})
```

属性 | 类型 | 读写 | 说明
---|---|---|---
url | string | 只读 | url
MIMEType | string | 只读 | MIME 类型
expectedContentLength | number | 只读 | 长度
textEncodingName | string | 只读 | 编码
suggestedFilename | string | 只读 | 建议的文件名
statusCode | number | 只读 | HTTP 状态码
headers | object | 只读 | HTTP header

---

## 文件内容解读与示例

### 用途说明

`response` 对象是 JSBox 中用于封装 **HTTP 响应详细信息**的标准数据类型。当你通过 `$http` 模块发起网络请求并收到服务器的响应时，`resp` 对象中会包含一个 `response` 属性，它就是这个 `response` 对象。它提供了关于 HTTP 状态、响应头、内容类型等关键信息，对于正确处理服务器返回的数据和调试网络问题至关重要。

### 核心概念：HTTP 响应的结构化表示

`response` 对象提供了一个结构化的方式来访问 HTTP 响应的各个部分，这比直接处理原始响应数据更方便和可靠。它与 iOS 原生的 `HTTPURLResponse` 对象相对应。

### 属性详解

-   **`response.url`**: 只读，实际响应的 URL。在发生重定向后，这可能与请求的 URL 不同。
-   **`response.MIMEType`**: 只读，响应内容的 MIME 类型（例如 `"application/json"`, `"text/html"`, `"image/jpeg"`）。这对于判断如何解析响应体非常重要。
-   **`response.expectedContentLength`**: 只读，响应体的预期长度（字节）。
-   **`response.textEncodingName`**: 只读，响应内容的文本编码（例如 `"utf-8"`）。
-   **`response.suggestedFilename`**: 只读，系统根据响应头（如 `Content-Disposition`）建议的文件名。
-   **`response.statusCode`**: 只读，HTTP 状态码（例如 `200` 表示成功，`404` 表示未找到，`500` 表示服务器错误）。这是判断请求是否成功的首要依据。
-   **`response.headers`**: 只读，一个 JavaScript 对象，包含了所有的 HTTP 响应头。你可以通过 `response.headers["Content-Type"]` 等方式访问。

### 示例代码：检查网络请求的响应详情

下面的示例将向一个公共 API 发起 GET 请求，并显示其 `response` 对象的详细信息。

```javascript
$ui.render({
  props: { title: "Response 对象示例" },
  views: [
    {
      type: "label",
      props: { id: "status-label", text: "点击按钮发起请求...", lines: 0 },
      layout: make => make.top.left.right.inset(20).height.equalTo(100)
    },
    {
      type: "button",
      props: { title: "发起请求" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(20).centerX.equalTo(view.super).width.equalTo(150),
      events: {
        tapped: () => {
          $("status-label").text = "请求中...";
          $http.get({
            url: "https://api.github.com/users/jsbox-app", // 一个公共 API
            handler: (resp) => {
              if (resp.error) {
                $("status-label").text = `请求失败: ${resp.error.localizedDescription}`; 
              } else if (resp.response) {
                const response = resp.response;
                let responseInfo = "";
                responseInfo += `URL: ${response.url}\n`;
                responseInfo += `状态码: ${response.statusCode}\n`;
                responseInfo += `MIME 类型: ${response.MIMEType}\n`;
                responseInfo += `预期长度: ${response.expectedContentLength} 字节\n`;
                responseInfo += `编码: ${response.textEncodingName || "未知"}\n`;
                responseInfo += `建议文件名: ${response.suggestedFilename || "无"}\n`;
                responseInfo += `部分响应头:\n`;
                responseInfo += `  Content-Type: ${response.headers["Content-Type"] || "无"}\n`;
                responseInfo += `  Server: ${response.headers["Server"] || "无"}\n`;
                responseInfo += `  Date: ${response.headers["Date"] || "无"}\n`;
                
                $("status-label").text = responseInfo;
                console.log("完整的响应头:", response.headers);
                console.log("响应数据:", resp.data);
              }
            }
          });
        }
      }
    }
  ]
});
```

**代码解读**：

1.  我们向 GitHub API 发起一个 GET 请求。
2.  在 `handler` 回调中，我们通过 `resp.response` 访问 `response` 对象。
3.  然后，我们访问 `response` 对象的各个属性（`url`, `statusCode`, `MIMEType`, `headers` 等），并将它们显示在 `status-label` 上。
4.  `console.log("完整的响应头:", response.headers);` 会将所有响应头打印到控制台，方便调试。

`response` 对象是理解和调试网络请求结果的关键。通过检查其属性，你可以全面了解服务器的响应，从而进行正确的数据处理和错误判断。 
