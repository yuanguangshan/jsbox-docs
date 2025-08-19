# type: "web"

`web` 用于创建一个简单的网页，在之后将会提供选项用于显示浏览器导航按钮：

```js
{
  type: "web",
  props: {
    url: "https://www.apple.com"
  },
  layout: $layout.fill
}
```

显示 Apple 首页。

也可以使用 `request` 参数附带更多信息：

```js
{
  type: "web",
  props: {
    request: {
      url: "https://www.apple.com",
      method: "GET",
      header: {},
      body: body // $data type
    }
  },
  layout: $layout.fill
}
```

# 加载本地文件

可以将 html, js 和 css 等文件都放在本地，然后用 html 属性加载：

```js
let html = $file.read("assets/index.html").string;
$ui.render({
  views: [
    {
      type: "web",
      props: {
        html
      },
      layout: $layout.fill
    }
  ]
});
```

```html
<html>
  <head>
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=no">
    <link rel="stylesheet" href="local://assets/index.css">
    <script src="local://assets/index.js"></script>
  </head>
  <body>
    <h1>Hey, there!</h1><img src="local://assets/icon.png">
  </body>
  <script>
    window.onload = () => {
      alert(sum(1, 1));
    }
  </script>
</html>
```

本地文件将会从安装包内寻找，同时也支持 `shared://` 和 `drive://` 等协议。

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
title | string | 只读 | 网页标题
url | string | 只写 | 地址
toolbar | bool | 只写 | 显示工具栏
html | string | 只写 | html
text | string | 只写 | text
loading | boolean | 只读 | 是否在加载
progress | number | 只读 | 加载进度
canGoBack | boolean | 只读 | 是否可以后退
canGoForward | boolean | 只读 | 是否可以前进
ua | string | 只写 | UserAgent(iOS 9+)
scrollEnabled | bool | 读写 | 是否可以滚动
bounces | bool | 读写 | 是否滚动回弹
transparent | bool | 读写 | 是否背景透明
showsProgress | bool | 只写 | 是否显示进度条
inlineMedia | bool | 只写 | 是否允许 inline video
airPlay | bool | 只写 | 是否允许 AirPlay
pictureInPicture | bool | 只写 | 是否允许画中画
allowsNavigation | bool | 读写 | 是否允许滑动返回
allowsLinkPreview | bool | 读写 | 是否允许链接预览

# goBack()

后退。

# goForward()

前进。

# reload()

重新加载。

# reloadFromOrigin()

从初始的 URL 重新加载。

# stopLoading()

停止加载。

# eval(object)

运行一段 JavaScript：

```js
webView.eval({
  script: "var sum = 1 + 2",
  handler: function(result, error) {

  }
})
```

# exec(script)

与 `eval` 类似，但此函数为 async 函数：

```js
const {result, error} = await webView.exec("1 + 1");
```

# events: didClose

`didClose` 会在网页关闭时回调：

```js
didClose: function(sender) {

}
```

# events: decideNavigation

`decideNavigation` 可以决定是否加载网页，用于拦截某些请求：

```js
decideNavigation: function(sender, action) {
  if (action.requestURL === "https://apple.com") {
    return false
  }
  return true
}
```

# events: didStart

`didStart` 在网页开始加载时回调：

```js
didStart: function(sender, navigation) {

}
```

# events: didReceiveServerRedirect

`didReceiveServerRedirect` 在收到服务器重定向时回调：

```js
didReceiveServerRedirect: function(sender, navigation) {

}
```

# events: didFinish

`didFinish` 在网页成功加载时调用：

```js
didFinish: function(sender, navigation) {

}
```

# events: didFail

`didFail` 在网页加载失败时调用：

```js
didFail: function(sender, navigation, error) {

}
```

# events: didSendRequest

`didSendRequest` 在页面 JavaScript 发送了 XMLHttpRequest 时候调用：

```js
didSendRequest: function(request) {
  var method = request.method
  var url = request.url
  var header = request.header
  var body = request.body
}
```

# js 注入

web 组件通过向 WebView 注入 JavaScript 来控制页面的行为，只需在 `props` 里面定义：

```js
props: {
  script: function() {
    var images = document.getElementsByTagName("img")
    for (var i=0; i<images.length; ++i) {
      var element = images[i]
      element.onclick = function(event) {
        var source = event.target || event.srcElement
        $notify("share", {"url": source.getAttribute("data-original")})
        return false
      }
    }
  }
}
```

script 是一个 `function`，function 里面所有的代码会在网页加载完成之后执行。

或者 script 内容也可以是一个字符串字面量，可以通过 [Uglify](https://skalman.github.io/UglifyJS-online/) 和 [Escape](https://www.freeformatter.com/json-escape.html) 的方式转换成这样一个字符串：

```js
props: {
  script: "for(var images=document.getElementsByTagName(\"img\"),i=0;i<images.length;++i){var element=images[i];element.onclick=function(e){var t=e.target||e.srcElement;return $notify(\"share\",{url:t.getAttribute(\"data-original\")}),!1}}"
}
```

# $notify(event, message)

在 script 的内容里面，可以通过 `$notify` 方法来将事件传递到 `events` 里面去处理：

```js
props: {
  script: function() {
    $notify("customEvent", {"key": "value"})
  }
},
events: {
  customEvent: function(object) {
    // object = {"key": "value"}
  }
}
```

这样的话页面加载完成之后将会调用 `notify` 将事件传递到 `events` 里面的 `test` 事件，从而可以在这里进一步和 native 代码互通。

这套机制有一点点复杂，请参考下面这个完整的例子：

```js
$ui.render({
  props: {
    title: "斗图啦"
  },
  views: [
    {
      type: "button",
      props: {
        title: "搜索"
      },
      layout: function(make) {
        make.right.top.inset(10)
        make.size.equalTo($size(64, 32))
      },
      events: {
        tapped: function(sender) {
          search()
        }
      }
    },
    {
      type: "input",
      props: {
        placeholder: "输入关键字"
      },
      layout: function(make) {
        make.top.left.inset(10)
        make.right.equalTo($("button").left).offset(-10)
        make.height.equalTo($("button"))
      },
      events: {
        returned: function(sender) {
          search()
        }
      }
    },
    {
      type: "web",
      props: {
        script: "for(var images=document.getElementsByTagName(\"img\"),i=0;i<images.length;++i){var element=images[i];element.onclick=function(e){var t=e.target||e.srcElement;return $notify(\"share\",{url:t.getAttribute(\"data-original\")}),!1}}"
      },
      layout: function(make) {
        make.left.bottom.right.equalTo(0)
        make.top.equalTo($("input").bottom).offset(10)
      },
      events: {
        share: function(object) {
          $http.download({
            url: `http:${object.url}`,
            handler: function(resp) {
              $share.universal(resp.data)
            }
          })
        }
      }
    }
  ]
})

function search() {
  const keyword = $("input").text;
  const url = `https://www.doutula.com/search?keyword=${encodeURIComponent(keyword)}`;
  $("input").blur()
  $("web").url = url
}

$("input").focus()
```

# 注入 css

与注入 script 类似的，style 也可以注入：

```js
props: {
  style: ""
}
```

当然，你也可以通过 js 注入的方式执行实现上述效果。

# Native 调用 Web 组件

Native 代码可以通过 `notify(event, message)` 给 web 组件发送消息：

```js
const webView = $("webView");
webView.notify({
  "event": "foobar",
  "message": {"key": "value"}
});
```

这将可以实现 Native 调用网页上面的方法。

---

## 文件内容解读与示例

### 组件用途

`web` 组件远不止是一个简单的网页查看器，它是在 JSBox 中嵌入一个功能完整的**浏览器核心**（基于 `WKWebView`）。这使它成为构建混合应用（Hybrid App）的基石，你可以无缝地融合 Web 技术的灵活性与 JSBox 原生能力的强大。

### 核心概念：双向通信的桥梁

`web` 组件最强大的地方在于它建立了一座连接 **JSBox 原生环境**与 **WebView 网页环境**的双向桥梁。

#### 1. 从 Web 到 JSBox：`$notify`

- **原理**: 在网页的 JavaScript 中，可以调用一个由 JSBox 注入的全局函数 `$notify(eventName, message)`。
- **作用**: 这个函数会向 JSBox 的 `web` 组件发送一个信号，触发其 `events` 中对应 `eventName` 的事件处理器。
- **示例**: 网页中的一个按钮被点击 -> 调用 `$notify("buttonClicked", { some: "data" })` -> 触发 JSBox 中 `events: { buttonClicked: data => { ... } }` 的逻辑。

#### 2. 从 JSBox 到 Web：`eval`/`exec`

- **原理**: 在 JSBox 的原生代码中，可以调用 `web` 视图实例的 `eval()` 或 `exec()` 方法。
- **作用**: 这两个方法可以在 `web` 组件当前加载的页面上**执行任意的 JavaScript 代码**。
- **示例**: JSBox 代码执行 `$("my-web").exec("document.body.style.backgroundColor = 'blue'")` -> 网页背景变为蓝色。

### 关键特性

- **加载方式**: 你可以通过 `url` 加载远程网页，也可以通过 `html` 加载一个完整的本地或内存中的 HTML 字符串。
- **脚本注入 (`script` 属性)**: 这是一个非常实用的功能。通过 `script` 属性提供的 JavaScript 代码（或函数）会在**每一个页面加载完成时**自动注入并执行。这非常适合用来对网页进行通用改造，例如修改页面样式、或为页面元素批量绑定 `$notify` 事件。
- **导航控制 (`decideNavigation` 事件)**: 这是一个强大的“守卫”事件。在页面将要跳转到新链接之前，此事件会被触发。你可以检查 `action.requestURL`，并返回 `true`（允许跳转）或 `false`（阻止跳转），从而实现对 WebView 内部导航行为的完全控制。

### 示例代码：一个简单的混合应用

下面的示例将加载一段自定义 HTML，其中包含一个按钮。当用户点击网页中的按钮时，会通过 `$notify` 通知 JSBox，JSBox 则会更新一个原生的 `label` 组件作为响应。

```javascript
// 1. 定义要加载的 HTML 内容
const myHTML = `
<html>
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>body { font-family: -apple-system, sans-serif; text-align: center; padding-top: 50px; }</style>
</head>
<body>
  <h2>这是 Web 视图内部</h2>
  <button id="my-button">点击我</button>

  <script>
    // 2. 在网页的 JS 中，为按钮添加点击事件
    document.getElementById("my-button").onclick = () => {
      // 3. 调用 $notify 将消息发送回 JSBox
      $notify("webEvent", { message: "Hello from Web!" });
    };
  </script>
</body>
</html>
`;

// 4. 渲染 UI
$ui.render({
  props: { title: "Web 组件示例" },
  views: [
    {
      type: "label",
      props: {
        id: "status-label",
        text: "等待来自 Web 的消息...",
        align: $align.center
      },
      layout: make => {
        make.top.left.right.inset(20);
        make.height.equalTo(40);
      }
    },
    {
      type: "web",
      props: {
        html: myHTML
      },
      layout: make => {
        make.top.equalTo(view.prev.bottom).offset(10);
        make.left.right.bottom.equalTo(0);
      },
      events: {
        // 5. 在 JSBox 中定义事件处理器，名称与 $notify 的第一个参数对应
        webEvent: (data) => {
          // 6. 更新原生 label 的文本
          $("status-label").text = `收到消息: ${data.message}`;
          $ui.toast("消息已收到！");
        }
      }
    }
  ]
});
```

**代码解读**：

这个例子清晰地展示了从 Web 到 JSBox 的单向通信流程。网页中的按钮通过 `$notify` 发送了一个名为 `webEvent` 的事件和一些数据。JSBox 端的 `web` 组件通过在 `events` 中定义同名函数 `webEvent` 来接收这个事件和数据，并据此更新了原生 `label` 的状态。这就是混合应用开发的核心模式。
