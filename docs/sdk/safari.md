> 用于 Safari 交互，例如打开 Safari View Controller 或注入 JavaScript 到 Safari

# $safari.open(object)

通过 [Safari View Controller](https://developer.apple.com/documentation/safariservices/sfsafariviewcontroller) 打开网页：

```js
$safari.open({
  url: "https://www.apple.com",
  entersReader: true,
  height: 360,
  handler: function() {

  }
})
```

`entersReader` 表示在可能的情况下自动进入阅读模式。

`height` 可以设置在 Widget 上运行时的打开高度，可选项。

`handler` 为完成后的回调，可选项。

# $safari.items

当你使用 Action Extension 时，可以使用这个方法获得 Safari 环境里的数据：

```js
const items = $safari.items; // JSON format
```

# $safari.inject(script)

> 在 iOS 15 上已过时，请使用更好的 [Safari 浏览器扩展](safari-extension/intro.md)。

当你使用 Action Extension 时，可以使用这个方法向 Safari 注入 JavaScript:

```js
$safari.inject("window.location.href = 'https://apple.com';")
```

Action Extension 会被关闭，同时 JavaScript 会被运行在 Safari。

更多有用的例子：https://github.com/cyanzhong/xTeko/tree/master/extension-scripts/safari-extensions

# $safari.addReadingItem(object)

添加到 Safari 的阅读列表：

```js
$safari.addReadingItem({
  url: "https://sspai.com",
  title: "Title", // Optional
  preview: "Preview text" // Optional
})
```

---

## 文件内容解读与示例

### 用途说明

`$safari` API 提供了与 iOS 系统**Safari 浏览器**进行交互的能力。它主要用于通过 **Safari View Controller (SFSafariViewController)** 在你的脚本内部提供原生的网页浏览体验，以及一些与 Safari 扩展相关的功能。这使得你的脚本能够无缝地集成网页内容，并利用 Safari 的强大功能。

### 核心功能：`$safari.open(options)` - Safari View Controller

-   **用途**: 这是 `$safari` 模块最常用且最重要的功能。它会以模态弹窗的形式，在你的脚本内部打开一个完整的 Safari 浏览器实例（SFSafariViewController）。
-   **优势**: 
    -   **原生体验**: SFSafariViewController 拥有 Safari 的所有特性，如自动填充、内容拦截器、共享 Cookie 和数据，提供与 Safari 应用本身一致的浏览体验。
    -   **安全与隐私**: 它独立于你的脚本进程运行，确保用户数据安全，不会被脚本直接访问。
    -   **无需离开应用**: 用户可以在你的脚本内浏览网页，完成后自动返回，提升了用户体验的流畅性。
-   **参数**: 
    -   `url`: 必填，要打开的网址。
    -   `entersReader`: 布尔值，如果可能，自动进入 Safari 的阅读模式。
    -   `height`: 在 Widget 上运行时，可以设置打开的高度。
    -   `handler`: 可选，当 Safari View Controller 关闭时的回调函数。

**示例**：在 Safari View Controller 中打开网页

```javascript
$ui.render({
  props: { title: "Safari VC 示例" },
  views: [
    {
      type: "input",
      props: { id: "url-input", placeholder: "请输入网址", text: "https://www.apple.com" },
      layout: make => make.top.left.right.inset(20).height.equalTo(36)
    },
    {
      type: "button",
      props: { title: "在 Safari VC 中打开" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).centerX.equalTo(view.super).width.equalTo(200),
      events: {
        tapped: () => {
          const url = $("url-input").text;
          if (url) {
            $safari.open({
              url: url,
              entersReader: true, // 尝试进入阅读模式
              handler: () => {
                $ui.toast("Safari View Controller 已关闭。");
              }
            });
          } else {
            $ui.toast("请输入网址！");
          }
        }
      }
    },
    {
      type: "button",
      props: { title: "添加到阅读列表" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).centerX.equalTo(view.super).width.equalTo(200),
      events: {
        tapped: () => {
          const url = $("url-input").text;
          if (url) {
            $safari.addReadingItem({
              url: url,
              title: $("url-input").text, // 使用网址作为标题
              preview: "来自 JSBox 的阅读列表项"
            });
            $ui.toast("已添加到阅读列表。");
          } else {
            $ui.toast("请输入网址！");
          }
        }
      }
    }
  ]
});
```

### 其他功能

#### 1. `$safari.items`

-   **用途**: 当脚本作为 Safari Action Extension 运行时，用于获取 Safari 环境中的数据（如当前页面的 URL、标题、选中的文本等）。
-   **注意**: 推荐使用 `$context.safari.items` 来访问这些数据，因为它更符合 `$context` 模块的语义。

#### 2. `$safari.inject(script)`

-   **用途**: 向 Safari 浏览器注入 JavaScript 代码。这允许你对 Safari 中加载的网页进行自定义操作。
-   **重要提示**: **此方法在 iOS 15 及更高版本上已过时**。现在推荐使用更强大、更规范的 [Safari 浏览器扩展](safari-extension/intro.md) 来实现类似功能。

#### 3. `$safari.addReadingItem(options)`

-   **用途**: 将一个 URL 添加到 Safari 的阅读列表。阅读列表允许用户离线阅读网页内容。
-   **参数**: `url`（必填），`title`（可选），`preview`（可选）。

### 总结

`$safari` API 是在 JSBox 中提供原生网页浏览体验的最佳方式。它通过 Safari View Controller 提供了与 Safari 应用一致的功能和安全性。对于更复杂的 Safari 交互和自动化，应优先考虑使用 iOS 15+ 引入的 Safari 浏览器扩展。 
