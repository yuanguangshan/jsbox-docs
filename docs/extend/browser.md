# $browser.exec

用来提供一个基于 webView 的 js 环境，这样的话你可以可以使用一些 Web APIs：

```js
$browser.exec({
  script: function() {
    const parser = new DOMParser();
    const doc = parser.parseFromString("<a>hey</a>", "application/xml");
    // $notify("customEvent", {"key": "value"})
    return doc.children[0].innerHTML;
  },
  handler: function(result) {
    $ui.alert(result);
  },
  customEvent: function(message) {

  }
})
```

# 方便的简写

你可以通过 Promise 写成非常简洁的形式：

```js
var result = await $browser.exec("return 1 + 1;");
```

# 如何访问 native 环境的变量

你可以通过类似这样的方式动态创建：

```js
const name = "JSBox";
$browser.exec({
  script: `
  var parser = new DOMParser();
  var doc = parser.parseFromString("<a>hey ${name}</a>", "application/xml");
  return doc.children[0].innerHTML;`,
  handler: function(result) {
    $ui.alert(result);
  }
})
```

通过字符串拼接的方法，name 会被填充为 native 环境的 `var name`。

关于 `$notify` 的使用，可以参考 [web 组件](component/web.md?id=notifyevent-message)。

---

## 文件内容解读与示例

### 用途说明：无头浏览器环境

`$browser.exec` 是一个非常强大且高级的 API，它允许你在一个**没有可见界面的 WebView 环境中执行 JavaScript 代码**。这被称为“无头浏览器”（Headless Browser）功能。它主要用于执行那些需要 Web API（如 `DOMParser`、`fetch`、`XMLHttpRequest`）或需要模拟浏览器行为（如处理 HTML/XML 字符串）的任务，而这些 API 在 JSBox 的标准 JavaScript 运行时中是不可用的。

### 核心概念

1.  **独立环境**: `$browser.exec` 创建的是一个独立的、轻量级的浏览器环境。它不会显示任何 UI，只负责执行你提供的 JavaScript 代码。
2.  **Web API 支持**: 在这个环境中，你可以使用标准的 Web API，例如：
    -   `DOMParser`: 用于将 HTML 或 XML 字符串解析成可操作的 DOM 对象。
    -   `fetch` / `XMLHttpRequest`: 进行网络请求，有时可以绕过 `$http` 可能遇到的 CORS 限制。
    -   其他浏览器环境特有的全局对象和方法。
3.  **与 JSBox 环境的交互**: 
    -   **返回结果**: 你在 `script` 中 `return` 的任何值，都会通过 `handler` 回调函数或 `await` 语法返回给你的 JSBox 脚本。
    -   **`$notify`**: 类似于 `web` 组件，你可以在 `script` 中调用 `$notify(eventName, message)` 来向 JSBox 脚本发送自定义事件和数据。
    -   **变量注入**: 通过 JavaScript 的模板字符串（反引号 ` `` `），你可以方便地将 JSBox 脚本中的变量值注入到 `script` 中执行的 JavaScript 代码里。

### `$browser.exec(options)` 参数详解

-   **`script`**: 必需。要在这个无头浏览器环境中执行的 JavaScript 代码。可以是一个函数，也可以是一个字符串。
-   **`handler`**: 可选。一个回调函数，用于接收 `script` 执行后的返回值。如果 `script` 是一个 `async` 函数，`handler` 会接收到 Promise resolve 的值。
-   **自定义事件**: 你可以在 `options` 中定义与 `script` 中 `$notify` 对应的事件处理函数。

### 示例代码：解析 HTML 字符串

下面的示例将演示如何使用 `$browser.exec` 来解析一个 HTML 字符串，并从中提取所有链接的文本和 URL。

```javascript
async function parseHtmlLinks(htmlString) {
  const result = await $browser.exec({
    script: function(html) {
      // 在无头浏览器环境中执行的 JavaScript
      const parser = new DOMParser();
      const doc = parser.parseFromString(html, "text/html");
      const links = [];
      doc.querySelectorAll("a").forEach(a => {
        links.push({
          text: a.textContent,
          href: a.href
        });
      });
      return links; // 返回解析结果
    },
    args: [htmlString] // 将外部的 htmlString 作为参数传递给 script 函数
  });
  return result;
}

// 模拟一个 HTML 字符串
const sampleHtml = `
<html>
<body>
  <h1>欢迎</h1>
  <p>这是一个 <a href="https://www.jsbox.com">JSBox 官网</a> 的链接。</p>
  <p>访问 <a href="https://docs.jsbox.com">JSBox 文档</a> 获取更多信息。</p>
</body>
</html>
`;

// 调用解析函数并显示结果
parseHtmlLinks(sampleHtml).then(links => {
  let message = "";
  if (links && links.length > 0) {
    message = "解析到的链接:\n";
    links.forEach(link => {
      message += `- ${link.text}: ${link.href}\n`;
    });
  } else {
    message = "未找到链接。";
  }
  $ui.alert(message);
}).catch(error => {
  $ui.alert(`解析失败: ${error}`);
});
```

**代码解读**：

1.  我们定义了一个 `parseHtmlLinks` 异步函数，它接收一个 HTML 字符串作为参数。
2.  在 `script` 内部，我们使用了 `DOMParser` 来将 HTML 字符串转换为一个可操作的 DOM 文档对象。
3.  然后，通过 `doc.querySelectorAll("a")` 找到所有的 `<a>` 标签，并遍历它们，提取 `textContent` 和 `href`。
4.  最后，将提取到的链接信息作为数组 `return`。这个数组会通过 Promise 返回给 `parseHtmlLinks` 函数的调用者。
5.  `args: [htmlString]` 演示了如何将 JSBox 环境中的变量作为参数传递给 `script` 函数。

`$browser.exec` 是一个高级工具，它为 JSBox 脚本打开了与 Web 技术深度交互的大门，是实现网页内容抓取、自动化操作等复杂功能的利器。 
