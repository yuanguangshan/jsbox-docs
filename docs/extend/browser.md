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

ima解读

`$browser.exec` 是 JSBox 环境中的一个高级 API，用于在**无头浏览器（Headless Browser）**环境中执行 JavaScript 代码。它通过创建一个隐藏的 WebView 实例，提供对 Web API 的访问能力，从而扩展 JSBox 脚本的功能边界。以下是其核心用途和实际应用场景的详细分析：

---

### 🌐 **一、核心功能：提供浏览器环境的能力**
JSBox 原生运行时缺乏完整的 Web API 支持（如 DOM 操作、XML 解析等），而 `$browser.exec` 填补了这一空白：
1. **支持 Web API**  
   - 在脚本中可直接使用 `DOMParser`、`fetch`、`XMLHttpRequest`、`localStorage` 等浏览器专属 API[1](@ref)。
   - 示例：解析 HTML/XML 字符串并提取数据：
     ```javascript
     $browser.exec({
       script: `return new DOMParser().parseFromString("<a>Hello</a>", "text/xml").body.innerHTML;`,
       handler: result => $ui.alert(result) // 输出 "<a>Hello</a>"
     });
     ```

2. **独立沙箱环境**  
   - 代码在隔离的 WebView 中运行，不影响 JSBox 主线程，避免全局变量污染[1](@ref)。

---

### ⚙️ **二、典型应用场景**
#### 1. **解析 HTML/XML 内容**
   - 从网页源码或 API 响应中提取结构化数据（如链接、文本、图片）。
   - **示例**：抓取网页中所有链接的文本和 URL：
     ```javascript
     const html = `<a href="https://jsbox.com">官网</a><a href="https://docs.com">文档</a>`;
     const links = await $browser.exec({
       script: html => [...document.querySelectorAll('a')].map(a => ({ text: a.textContent, href: a.href })),
       args: [html]
     });
     ```

#### 2. **模拟浏览器行为**
   - 执行依赖 DOM 的操作（如渲染 SVG、操作虚拟 DOM）。
   - **示例**：动态生成 SVG 并获取 Base64 编码：
     ```javascript
     const svgCode = `<svg xmlns="http://www.w3.org/2000/svg" width="100" height="100"><circle cx="50" cy="50" r="40" fill="red"/></svg>`;
     const base64 = await $browser.exec({
       script: svg => {
         const img = new Image();
         img.src = 'data:image/svg+xml,' + encodeURIComponent(svg);
         return img.src;
       },
       args: [svgCode]
     });
     ```

#### 3. **与 JSBox 环境交互**
   - 通过 `$notify` 发送事件和 `handler` 回调实现双向通信。
   - **示例**：异步处理数据后返回结果：
     ```javascript
     $browser.exec({
       script: async () => {
         const data = await fetch('https://api.example.com/data').then(res => res.json());
         $notify("dataReady", data); // 触发自定义事件
         return "Done";
       },
       dataReady: data => $ui.alert(`收到数据: ${JSON.stringify(data)}`),
       handler: msg => console.log(msg) // 输出 "Done"
     });
     ```

---

### 🚀 **三、技术优势**
| **能力**                | **JSBox 原生环境** | **`$browser.exec` 环境** |
|-------------------------|---------------------|--------------------------|
| DOM 操作（如 `querySelector`） | ❌ 不支持           | ✅ 支持                  |
| Web API（`fetch`、`DOMParser`）| ❌ 不支持           | ✅ 支持                  |
| 多线程/异步任务           | ✅ 支持             | ✅ 支持（独立 WebView）  |
| 与 iOS 原生模块交互        | ✅ 直接调用         | 🔶 需通过事件通信         |

---

### 🧩 **四、实际案例**
#### 1. **网页爬虫工具**
   ```javascript
   async function scrape(url) {
     return $browser.exec({
       script: async (url) => {
         const res = await fetch(url);
         const html = await res.text();
         const parser = new DOMParser();
         const doc = parser.parseFromString(html, "text/html");
         return [...doc.querySelectorAll("h1, h2")].map(el => el.textContent);
       },
       args: [url]
     });
   }
   scrape("https://example.com").then(headings => console.log(headings));
   ```

#### 2. **XML 数据转换**
   ```javascript
   const xmlData = `<items><item id="1">Apple</item><item id="2">Banana</item></items>`;
   const items = await $browser.exec({
     script: xml => {
       const doc = new DOMParser().parseFromString(xml, "application/xml");
       return [...doc.querySelectorAll("item")].map(item => ({
         id: item.getAttribute("id"),
         name: item.textContent
       }));
     },
     args: [xmlData]
   });
   ```

---

### 💡 **五、进阶技巧**
1. **动态注入变量**  
   通过字符串模板或 `args` 参数将 JSBox 环境中的变量传递到浏览器环境：
   ```javascript
   const userName = "JSBoxUser";
   $browser.exec({
     script: `return "Hello, ${userName}";`, // 直接拼接变量
     handler: greeting => $ui.alert(greeting)
   });
   ```

2. **Promise 简化语法**  
   使用 `await` 直接获取返回值（需在 `async` 函数中）：
   ```javascript
   const result = await $browser.exec(`return navigator.userAgent;`);
   $ui.alert(`UserAgent: ${result}`);
   ```

3. **错误处理**  
   通过 `try/catch` 捕获执行异常：
   ```javascript
   try {
     await $browser.exec(`invalidSyntax!`);
   } catch (e) {
     $ui.alert(`执行错误: ${e.message}`);
   }
   ```

---

### 💎 **总结**
`$browser.exec` 的核心价值在于**弥合了 JSBox 原生环境与浏览器能力之间的鸿沟**。它适用于：
- 需要操作 DOM 或解析 HTML/XML 的场景；
- 调用浏览器专属 API（如 `fetch`、`DOMParser`）；
- 执行复杂异步任务且需隔离运行环境的场景。

通过结合 JSBox 的原生 iOS 功能（如通知、文件操作）和 `$browser.exec` 的 Web 能力，开发者可以构建更强大的自动化脚本或工具应用[1](@ref)[5](@ref)。