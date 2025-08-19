> 以下内容是通过一些简单的样例展示 JSBox 的接口设计，可以对 JSBox 有个概括性的了解。

# Hello, World!

```js
// 弹出 alert
$ui.alert("Hello, World!")
```

```js
// 打印到控制台
console.info("Hello, World!")
```

# 获取并展示剪贴板内容

```js
$ui.preview({
  text: JSON.stringify($clipboard.items)
})
```

# 简单的 HTTP 请求

```js
$http.get({
  url: 'https://docs.xteko.com',
  handler: function(resp) {
    const data = resp.data;
  }
})
```

# 创建一个按钮

```js
$ui.render({
  views: [
    {
      type: "button",
      props: {
        title: "Button"
      },
      layout: function(make, view) {
        make.center.equalTo(view.super)
        make.width.equalTo(64)
      },
      events: {
        tapped: function(sender) {
          $ui.toast("Tapped")
        }
      }
    }
  ]
})
```

以上几个例子可以简单的认识一下接口的基本设计，在之后的文档里面有更详尽的介绍。

---

## 文件内容解读与示例

### 用途说明

本文档提供了一系列**简单而典型的 JSBox 脚本示例**，旨在帮助新用户快速了解 JSBox API 的基本设计和使用方式。这些示例涵盖了 UI 交互、系统功能调用和网络请求等核心方面，是快速入门 JSBox 开发的绝佳起点。

### 示例详解

#### 1. Hello, World!

这是编程世界的传统开端，展示了如何在 JSBox 中输出信息。

-   **`$ui.alert("Hello, World!")`**: 
    -   **用途**: 弹出一个原生的系统提示框，显示一条消息。
    -   **概念**: 引入了 `$ui`（User Interface）模块，它是 JSBox 中与用户界面交互的核心模块。

-   **`console.info("Hello, World!")`**: 
    -   **用途**: 将消息打印到 JSBox 的控制台（可在脚本编辑器底部查看），用于调试和日志输出。
    -   **概念**: 引入了 `console` 对象，它是 JavaScript 中进行调试的常用工具。

**示例代码**：

```javascript
$ui.alert("Hello, World! (UI 提示)");
console.info("Hello, World! (控制台输出)");
```

#### 2. 获取并展示剪贴板内容

这个示例展示了脚本如何与 iOS 系统服务进行交互，获取剪贴板中的数据。

-   **`$clipboard.items`**: 
    -   **用途**: 获取 iOS 系统剪贴板中的所有内容。它返回一个数组，每个元素代表剪贴板中的一个项目，可能包含文本、图片、链接等不同类型的数据。
    -   **概念**: 引入了 `$clipboard` 模块，用于与系统剪贴板进行交互。
-   **`JSON.stringify($clipboard.items)`**: 
    -   **用途**: 将 JavaScript 对象（这里是剪贴板项目数组）转换为 JSON 格式的字符串，以便于显示或传输。
-   **`$ui.preview({ text: ... })`**: 
    -   **用途**: 在一个临时的预览窗口中显示文本内容。这对于快速查看复杂数据结构非常方便。

**示例代码**：

```javascript
// 假设你的剪贴板中已经复制了一些内容
$ui.preview({
  text: JSON.stringify($clipboard.items, null, 2), // null, 2 用于格式化 JSON，使其更易读
  title: "剪贴板内容"
});
```

#### 3. 简单的 HTTP 请求

这个示例展示了脚本如何进行网络通信，从互联网获取数据。

-   **`$http.get({ url: ..., handler: ... })`**: 
    -   **用途**: 发起一个 HTTP GET 请求到指定的 URL。这是从 Web API 获取数据最常用的方法。
    -   **`$http`**: 引入了 `$http` 模块，它是 JSBox 中进行网络请求的核心模块。
    -   **`handler`**: 一个回调函数，当网络请求完成并收到响应时，这个函数会被调用。`resp` 参数包含了响应的所有信息，`resp.data` 通常是服务器返回的已解析数据（如 JSON 对象）。

**示例代码**：

```javascript
$ui.toast("正在获取 Chuck Norris 的随机笑话...");
$http.get({
  url: "https://api.chucknorris.io/jokes/random",
  handler: function(resp) {
    if (resp.data && resp.data.value) {
      $ui.alert({
        title: "Chuck Norris Joke",
        message: resp.data.value
      });
    } else {
      $ui.alert("获取笑话失败，请检查网络或 API。");
    }
  }
});
```

#### 4. 创建一个按钮

这个示例展示了如何在 JSBox 中创建自定义的用户界面，并响应用户的交互。

-   **`$ui.render({ views: [...] })`**: 
    -   **用途**: 这是 JSBox 中渲染自定义 UI 的核心函数。它接受一个包含视图定义的对象作为参数。
-   **`type: "button"`**: 
    -   **用途**: 定义一个按钮组件。
-   **`props`**: 
    -   **用途**: 设置组件的各种属性，如 `title`（按钮上显示的文本）。
-   **`layout`**: 
    -   **用途**: 定义组件在父视图中的位置和大小。`make.center.equalTo(view.super)` 使按钮居中，`make.width.equalTo(64)` 设置宽度。
-   **`events: { tapped: ... }`**: 
    -   **用途**: 定义当用户点击按钮时要执行的动作。`sender` 参数指向被点击的按钮本身。
-   **`$ui.toast("Tapped")`**: 
    -   **用途**: 显示一个短暂的、非阻塞的提示消息，通常用于用户操作的即时反馈。

**示例代码**：

```javascript
$ui.render({
  props: {
    title: "UI 按钮示例"
  },
  views: [
    {
      type: "button",
      props: {
        title: "点击我"
      },
      layout: function(make, view) {
        make.center.equalTo(view.super);
        make.size.equalTo($size(100, 40)); // 设置按钮大小
      },
      events: {
        tapped: function(sender) {
          $ui.toast("按钮被点击了！");
        }
      }
    }
  ]
});
```

### 总结

这些简单的示例为你提供了 JSBox API 设计的概览。它们展示了如何使用以 `$` 开头的 API 来进行 UI 交互、系统功能调用和网络请求。在后续的文档中，你将找到这些模块更详尽的介绍和更多高级用法。 
