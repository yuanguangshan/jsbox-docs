# navigationAction

`navigationAction` 在 `web` 控件的回调里面表示一个导航操作。

属性 | 类型 | 读写 | 说明
---|---|---|---
type | number | 只读 | 类型
sourceURL | string | 只读 | source url
targetURL | string | 只读 | target url
requestURL | string | 只读 | request url

---

## 文件内容解读与示例

### 用途说明

`navigationAction` 对象是 JSBox 中用于**控制 `web` 视图导航行为**的关键数据结构。它在 `web` 组件的 `decideNavigation` 事件中被传递。通过检查 `navigationAction` 对象的属性，你的脚本可以**拦截并决定**是否允许网页进行某个导航操作（例如，点击链接、提交表单、页面重定向等）。这对于实现自定义的链接处理、广告拦截、或限制用户访问特定网站等功能非常有用。

### 核心概念：导航守卫

`web` 组件的 `decideNavigation` 事件与 `navigationAction` 对象共同构成了“导航守卫”机制。在每次网页尝试加载新内容之前，这个事件都会被触发，为你提供一个机会来检查即将发生的导航，并返回 `true`（允许导航）或 `false`（阻止导航）。

### 属性详解

`navigationAction` 对象包含了关于即将发生的导航操作的详细信息：

-   **`navigationAction.type`**: 只读，表示导航的类型。例如，它可能是用户点击链接（`WKNavigationTypeLinkActivated` 对应的数值）、表单提交（`WKNavigationTypeFormSubmitted`）、页面重载（`WKNavigationTypeReload`）等。具体数值含义通常与 `WKNavigationType` 枚举对应。
-   **`navigationAction.sourceURL`**: 只读，发起导航的当前页面的 URL。即用户当前正在浏览的页面的 URL。
-   **`navigationAction.targetURL`**: 只读，导航的目标 URL。这是用户或页面试图跳转到的 URL。在大多数情况下，它与 `requestURL` 相同。
-   **`navigationAction.requestURL`**: 只读，实际发出的 HTTP 请求的 URL。在某些情况下（如重定向），`targetURL` 和 `requestURL` 可能不同。通常，这是你进行判断的主要依据。

### 示例代码：拦截特定网站的访问

下面的示例将创建一个 `web` 视图，加载一个包含多个链接的简单 HTML 页面。脚本会拦截所有导航尝试，只允许访问 `apple.com`，而阻止访问 `google.com` 和其他网站。

```javascript
const sampleHtml = `
<html>
<head><title>导航测试</title></head>
<body>
  <h1>导航拦截示例</h1>
  <p>点击以下链接：</p>
  <ul>
    <li><a href="https://www.apple.com">访问 Apple 官网</a></li>
    <li><a href="https://www.google.com">访问 Google 官网</a></li>
    <li><a href="https://www.jsbox.com">访问 JSBox 官网</a></li>
  </ul>
  <button onclick="window.location.href='https://www.bing.com'">跳转到 Bing</button>
</body>
</html>
`;

$ui.render({
  props: { title: "导航拦截" },
  views: [
    {
      type: "web",
      props: {
        html: sampleHtml
      },
      layout: $layout.fill,
      events: {
        // decideNavigation 事件接收 sender 和 action 参数
        decideNavigation: (sender, action) => {
          const requestURL = action.requestURL;
          console.log(`尝试导航到: ${requestURL}`);

          if (requestURL.includes("apple.com")) {
            $ui.toast("允许访问 Apple 官网");
            return true; // 允许导航
          } else if (requestURL.includes("google.com")) {
            $ui.toast("阻止访问 Google 官网！");
            return false; // 阻止导航
          } else {
            $ui.toast(`阻止访问 ${requestURL}！`);
            return false; // 默认阻止其他所有导航
          }
        },
        didFail: (sender, navigation, error) => {
          console.error(`导航失败: ${error.localizedDescription}`);
        }
      }
    }
  ]
});
```

**代码解读**：

1.  我们创建了一个 `web` 视图，并加载了一段包含多个链接和按钮的 HTML 内容。
2.  在 `web` 视图的 `decideNavigation` 事件中，我们接收到 `action` 参数，其中包含了即将发生的导航的详细信息。
3.  我们通过 `action.requestURL` 来判断目标 URL。根据 URL 的不同，我们返回 `true`（允许导航）或 `false`（阻止导航）。
4.  这个示例展示了如何利用 `navigationAction` 对象作为“守卫”，对 `web` 视图的导航行为进行精细的控制。 
