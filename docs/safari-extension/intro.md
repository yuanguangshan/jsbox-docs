# Safari 浏览器扩展

在 iOS 15 及以上，JSBox 为 [Safari 浏览器扩展](https://support.apple.com/zh-cn/guide/iphone/iphab0432bf6/ios) 提供了全面支持，您可以使用 JavaScript 定制自己的 Safari，并且支持自动运行和手动运行两种形式。

关于 Safari 扩展的使用方法，请参考上述 Apple 官方文档，本文档主要讲解开发相关内容。

## JavaScript Web API

与 JSBox 脚本不同的是，Safari 扩展运行在浏览器环境，脚本可以使用所有的 Web 接口，但不能使用 JSBox 专属的接口。

与之相关的开发文档，请查询 Mozilla 的 [Web API 接口参考](https://developer.mozilla.org/zh-CN/docs/Web/API)。

> 您可以在桌面端浏览器或应用内的 WebView 环境测试 Safari 扩展，但在真实的 Safari 环境中测试仍然是必要的步骤。

## Safari 工具

Safari 工具旨在为 Safari 提供扩展功能，需用户手动点击运行，创建方式：

- 新建项目
- Safari 工具

如使用 VS Code 插件创建项目，文件名需以 `.safari-tool.js` 结尾，同步到 JSBox 后会被识别为 Safari 工具。

样例代码：

```js
const video = document.querySelector("video");
if (video) {
  video.webkitSetPresentationMode("picture-in-picture");
} else {
  alert("No videos found.");
}
```

此工具运行后，将会让 YouTube 页面的视频进入画中画模式。

## Safari 规则

Safari 规则旨在为 Safari 提供自动运行的插件，无需用户手动点击运行，创建方式：

- 新建项目
- Safari 规则

如使用 VS Code 插件创建项目，文件名需以 `.safari-rule.js` 结尾，同步到 JSBox 后会被识别为 Safari 规则。

样例代码：

```js
const style = document.createElement("style");
const head = document.head || document.getElementsByTagName("head")[0];
head.appendChild(style);
style.appendChild(document.createTextNode("._9AhH0 { display: none }"));
```

此规则运行后，将会允许用户下载 Instagram 页面的图片。

> 出于安全考虑，Safari 规则默认禁止自动运行，需用户在应用设置中手动开启。

---

## 文件内容解读与示例

### 用途说明

本文档介绍了 JSBox 对 **iOS 15 及更高版本 Safari 浏览器扩展**的全面支持。它允许开发者使用纯 JavaScript 来定制 Safari 浏览器的行为，实现网页内容的修改、自动化操作等。这些扩展可以手动触发（“Safari 工具”）或自动运行（“Safari 规则”），极大地增强了 Safari 的功能和用户体验。

### 核心概念：浏览器环境与 JSBox API 限制

-   **运行环境**: Safari 扩展运行在**浏览器环境**中。这意味着你的脚本可以访问和使用所有标准的 Web API（如 `document`、`window`、`fetch`、`DOMParser` 等），但**不能使用 JSBox 专属的 API**（如 `$ui`、`$http`、`$file` 等）。
-   **开发与测试**: 你可以在桌面浏览器或 JSBox 内的 `web` 组件中进行初步测试，但最终仍需在真实的 Safari 环境中进行验证，因为不同环境的兼容性可能存在差异。

### 两种 Safari 扩展类型

#### 1. Safari 工具 (Safari Tool)

-   **用途**: 提供用户**手动触发**的辅助功能。用户需要在 Safari 中点击扩展图标来运行它。
-   **创建方式**: 在 JSBox 中选择“新建项目” -> “Safari 工具”。
-   **文件命名约定**: 脚本文件名需以 `.safari-tool.js` 结尾。同步到 JSBox 后会被识别为 Safari 工具。

**示例**：提取当前页面所有图片 URL 的工具

```javascript
// my_image_extractor.safari-tool.js

const imageUrls = [];
document.querySelectorAll('img').forEach(img => {
  if (img.src) {
    imageUrls.push(img.src);
  }
});

if (imageUrls.length > 0) {
  alert("当前页面图片 URL:\n" + imageUrls.join('\n'));
} else {
  alert("当前页面未找到图片。");
}
```

#### 2. Safari 规则 (Safari Rule)

-   **用途**: 提供**自动运行**的网页修改或自动化功能。无需用户手动点击，当满足特定条件时（例如访问某个网站），它会自动执行。
-   **创建方式**: 在 JSBox 中选择“新建项目” -> “Safari 规则”。
-   **文件命名约定**: 脚本文件名需以 `.safari-rule.js` 结尾。同步到 JSBox 后会被识别为 Safari 规则。
-   **安全考量**: **重要提示**：出于安全考虑，Safari 规则默认是禁止自动运行的。用户需要在 iOS 系统设置中手动为 JSBox 开启 Safari 扩展的自动运行权限。

**示例**：自动隐藏特定网页元素

```javascript
// my_clean_page.safari-rule.js

// 假设某个网站的广告弹窗有 class="ad-popup"
const adPopup = document.querySelector('.ad-popup');
if (adPopup) {
  adPopup.style.display = 'none';
  console.log("广告弹窗已隐藏！");
}

// 假设某个网站的固定导航栏有 id="fixed-header"
const fixedHeader = document.getElementById('fixed-header');
if (fixedHeader) {
  fixedHeader.style.position = 'static'; // 取消固定定位
  console.log("固定头部已取消固定！");
}
```

### 总结

JSBox 的 Safari 扩展功能为开发者提供了强大的能力，可以深度定制 Safari 浏览体验。理解“浏览器环境”的限制（不能使用 JSBox API）以及“工具”和“规则”两种类型的区别，是开发高效 Safari 扩展的关键。 
