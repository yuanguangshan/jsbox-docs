# error

`error` 类型用于封装一个出错的信息。

属性 | 类型 | 读写 | 说明
---|---|---|---
domain | string | 只读 | domain
code | number | 只读 | code
userInfo | object | 只读 | userInfo
localizedDescription | string | 只读 | 描述
localizedFailureReason | string | 只读 | 原因
localizedRecoverySuggestion | string | 只读 | 建议

---

## 文件内容解读与示例

### 用途说明

`error` 对象是 JSBox 中用于表示**错误信息**的标准数据类型。当 JSBox 的 API 调用失败时（例如网络请求失败、文件操作失败、权限不足等），它通常会返回一个 `error` 对象，而不是简单地抛出 JavaScript 异常。这个对象封装了错误的详细信息，帮助开发者诊断问题，并向用户提供有意义的反馈。

### 核心概念：结构化错误处理

`error` 对象提供了一种结构化的方式来处理错误，这比仅仅依靠错误消息字符串更可靠和方便。它与 iOS 原生的 `NSError` 对象相对应，包含了错误发生的原因、代码和可能的解决方案。

### 属性详解

-   **`error.domain`**: 错误所属的领域或类别。通常是一个字符串，用于标识错误的来源（例如，`NSURLErrorDomain` 表示网络错误，`NSPOSIXErrorDomain` 表示文件系统错误）。
-   **`error.code`**: 错误的数字代码。在特定 `domain` 下，每个代码都有特定的含义。开发者可以根据 `domain` 和 `code` 来精确判断错误类型。
-   **`error.localizedDescription`**: 最常用的属性，提供一个用户友好的、本地化的错误描述字符串。这是通常直接显示给用户的错误信息。
-   **`error.localizedFailureReason`**: 提供错误发生原因的本地化描述。比 `localizedDescription` 更具体，解释了为什么会失败。
-   **`error.localizedRecoverySuggestion`**: 提供解决错误的建议的本地化描述。指导用户或开发者如何尝试恢复或解决问题。
-   **`error.userInfo`**: 一个包含额外错误信息的字典（JavaScript 对象）。通常包含更详细的调试信息，例如导致错误的 URL、文件路径等。

### 示例代码：处理网络请求错误

下面的示例将尝试向一个不存在的 URL 发起网络请求，并捕获返回的 `error` 对象，然后显示其详细信息。

```javascript
$ui.render({
  props: { title: "错误对象示例" },
  views: [
    {
      type: "label",
      props: { id: "status-label", text: "点击按钮发起请求...", lines: 0 },
      layout: make => make.top.left.right.inset(20).height.equalTo(80)
    },
    {
      type: "button",
      props: { title: "发起错误请求" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(20).centerX.equalTo(view.super).width.equalTo(150),
      events: {
        tapped: () => {
          $("status-label").text = "请求中...";
          $http.get({
            url: "https://this-url-does-not-exist-12345.com/api/data", // 一个不存在的URL
            timeout: 5, // 设置超时时间
            handler: (resp) => {
              if (resp.error) {
                // 请求失败，resp.error 存在
                const error = resp.error;
                let errorMessage = "请求失败！\n";
                errorMessage += `描述: ${error.localizedDescription || "无"}\n`;
                errorMessage += `原因: ${error.localizedFailureReason || "无"}\n`;
                errorMessage += `建议: ${error.localizedRecoverySuggestion || "无"}\n`;
                errorMessage += `领域: ${error.domain || "无"}\n`;
                errorMessage += `代码: ${error.code || "无"}\n`;
                errorMessage += `详细信息: ${JSON.stringify(error.userInfo || {}, null, 2)}`;

                $("status-label").text = errorMessage;
                console.error("完整的错误对象:", error);
              } else if (resp.data) {
                // 请求成功
                $("status-label").text = `请求成功！\n数据: ${JSON.stringify(resp.data)}`;
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

1.  我们向一个故意设置的、不存在的 URL 发起 `$http.get` 请求，以确保它会失败。
2.  在 `handler` 回调中，我们首先检查 `resp.error` 是否存在。如果存在，说明请求失败。
3.  然后，我们访问 `error` 对象的各个属性（`localizedDescription`, `domain`, `code` 等），将它们组合成一个详细的错误消息，并显示在 `status-label` 上。
4.  `console.error(error)` 会将完整的错误对象打印到控制台，这对于开发者进行更深入的调试非常有用。

`error` 对象是 JSBox 中进行健壮错误处理的关键。通过检查和解析 `error` 对象的属性，你的脚本可以更精确地判断错误类型，并向用户提供更友好的反馈或尝试进行恢复操作。 
