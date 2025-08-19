# WebSocket

JSBox 支持类似 [WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) 的接口，用于与服务器之间提供 socket 连接。

# $socket.new(object)

创建一个新的 socket 对象：

```js
const socket = $socket.new("wss://echo.websocket.org");
```

你也可以指定一些参数：

```js
const socket = $socket.new({
  url: "wss://echo.websocket.org",
  protocols: [],
  allowsUntrustedSSLCertificates: true
});
```

# socket.listen(object)

用户监听 socket 消息：

```js
socket.listen({
  didOpen: (sock) => {
    console.log("Websocket Connected");
  },
  didFail: (sock, error) => {
    console.error(`:( Websocket Failed With Error: ${error}`);
  },
  didClose: (sock, code, reason, wasClean) => {
    console.log("WebSocket closed");
  },
  didReceiveString: (sock, string) => {
    console.log(`Received: ${string}`);
  },
  didReceiveData: (sock, data) => {
    console.log(`Received: ${data}`);
  },
  didReceivePing: (sock, data) => {
    console.log("WebSocket received ping");
  },
  didReceivePong: (sock, data) => {
    console.log("WebSocket received pong");
  }
});
```

# socket.open()

打开 WebSocket。

# socket.close(object)

关闭 WebSocket:

```js
socket.close({
  code: 1000, // Optional, see: https://developer.mozilla.org/en-US/docs/Web/API/CloseEvent
  reason: "reason", // Optional
});
```

# socket.send(object)

发送内容，例如：

```js
const object = socket.send("Message");
const result = object.result;
const error = object.error;
```

也可以通过 data 发送：

```js
socket.send({
  data,
  noCopy: true, // Optional
});
```

# socket.ping(data)

```js
const object = socket.ping(data);
const result = object.result;
const error = object.error;
```

# socket.readyState

获取状态：

```js
const readyState = socket.readyState;
// 0: connecting, 1: open, 2: closing, 3: closed
```

完整例子请见：https://github.com/cyanzhong/xTeko/blob/master/extension-demos/socket.js

---

## 文件内容解读与示例

### 用途说明

`$socket` API 提供了在 JSBox 脚本中**建立和管理 WebSocket 连接**的能力。WebSocket 是一种在客户端和服务器之间进行**全双工（双向）**、**持久性**通信的协议。它非常适合需要实时数据传输的应用，如聊天应用、实时数据仪表盘、在线游戏、或任何需要服务器主动推送数据的场景。

### 核心概念：全双工与事件驱动

-   **全双工**: 与传统的 HTTP 请求（客户端发送请求，服务器响应）不同，WebSocket 连接一旦建立，客户端和服务器可以**在任何时候互相发送数据**，无需等待对方的请求或响应。
-   **持久连接**: 连接一旦建立，就会一直保持开放，直到一方主动关闭。这减少了每次通信的开销。
-   **事件驱动**: 所有的通信和连接状态变化都通过事件回调来处理，这使得编写异步网络代码更加直观。

### 主要方法与功能详解

#### 1. 创建与连接

-   **`$socket.new(options)`**: 创建一个 WebSocket 实例。`options` 对象包含：
    -   `url`: 必填，WebSocket 服务器的 URL，必须以 `ws://` 或 `wss://` 开头。
    -   `protocols`: 可选，一个字符串数组，表示客户端支持的子协议。
    -   `allowsUntrustedSSLCertificates`: 可选，布尔值，如果为 `true`，则允许连接到使用自签名 SSL 证书的服务器（仅用于开发测试，生产环境慎用）。
-   **`socket.open()`**: 启动连接过程。连接是异步建立的，结果通过 `didOpen` 或 `didFail` 事件通知。

**示例**：创建并打开一个 WebSocket 连接

```javascript
const ECHO_SERVER_URL = "wss://echo.websocket.org"; // 一个公共的 WebSocket Echo 服务器
let ws = null; // 用于存储 WebSocket 实例

function connectWebSocket() {
  if (ws && ws.readyState === 1) { // 如果已经连接，则不重复连接
    $ui.toast("已连接。");
    return;
  }

  ws = $socket.new(ECHO_SERVER_URL);

  ws.listen({
    didOpen: (sock) => {
      console.log("WebSocket 连接已建立！");
      $("status-label").text = "状态: 已连接";
      $ui.toast("连接成功！");
    },
    didFail: (sock, error) => {
      console.error(`WebSocket 连接失败: ${error}`);
      $("status-label").text = `状态: 连接失败 (${error})`;
      $ui.alert(`连接失败: ${error}`);
    },
    didClose: (sock, code, reason, wasClean) => {
      console.log(`WebSocket 连接关闭: ${code} - ${reason}`);
      $("status-label").text = "状态: 已关闭";
      $ui.toast("连接已关闭。");
    },
    didReceiveString: (sock, message) => {
      console.log(`收到消息: ${message}`);
      $("log-text").text += `
收到: ${message}`;
      $("log-text").scrollRangeToVisible($range($("log-text").text.length, 0)); // 滚动到底部
    },
    didReceiveData: (sock, data) => {
      console.log(`收到二进制数据: ${data.info.size} 字节`);
      $("log-text").text += `
收到二进制数据 (${data.info.size} 字节)`;
      $("log-text").scrollRangeToVisible($range($("log-text").text.length, 0));
    }
  });

  ws.open(); // 启动连接
  $("status-label").text = "状态: 连接中...";
}
```

#### 2. 发送消息

-   **`socket.send(data)`**: 发送文本或二进制数据到服务器。`data` 可以是字符串或 `$data` 对象。
-   **`socket.ping(data)`**: 发送 Ping 帧到服务器，用于保持连接活跃或测量延迟。

**示例**：发送文本消息

```javascript
function sendMessage() {
  const message = $("message-input").text;
  if (ws && ws.readyState === 1 && message) {
    ws.send(message);
    $("log-text").text += `
发送: ${message}`;
    $("message-input").text = ""; // 清空输入框
    $("log-text").scrollRangeToVisible($range($("log-text").text.length, 0));
  } else if (!ws || ws.readyState !== 1) {
    $ui.toast("请先连接 WebSocket！");
  }
}
```

#### 3. 关闭连接

-   **`socket.close(options)`**: 关闭 WebSocket 连接。`options` 可以包含 `code`（关闭状态码）和 `reason`（关闭原因）。

**示例**：

```javascript
function disconnectWebSocket() {
  if (ws && ws.readyState === 1) {
    ws.close({ code: 1000, reason: "User disconnected" }); // 1000 是正常关闭码
  } else {
    $ui.toast("未连接或已关闭。");
  }
}
```

#### 4. 连接状态

-   **`socket.readyState`**: 获取当前连接的状态。
    -   `0`: `connecting` (连接中)
    -   `1`: `open` (已打开)
    -   `2`: `closing` (正在关闭)
    -   `3`: `closed` (已关闭)

### 示例代码：WebSocket Echo 客户端

下面的示例将创建一个简单的 WebSocket Echo 客户端，连接到一个公共的 Echo 服务器，发送文本消息并显示收到的回显消息。

```javascript
$ui.render({
  props: { title: "WebSocket 客户端" },
  views: [
    {
      type: "label",
      props: { id: "status-label", text: "状态: 未连接", align: $align.center },
      layout: make => make.top.left.right.inset(10).height.equalTo(30)
    },
    {
      type: "button",
      props: { title: "连接" },
      layout: make => make.top.equalTo($("status-label").bottom).offset(10).left.inset(10).width.equalTo(80).height.equalTo(36),
      events: { tapped: connectWebSocket }
    },
    {
      type: "button",
      props: { title: "断开" },
      layout: make => make.top.equalTo($("status-label").bottom).offset(10).left.equalTo(view.prev.right).offset(10).width.equalTo(80).height.equalTo(36),
      events: { tapped: disconnectWebSocket }
    },
    {
      type: "input",
      props: { id: "message-input", placeholder: "输入消息" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.right.inset(10).height.equalTo(36)
    },
    {
      type: "button",
      props: { title: "发送" },
      layout: make => make.top.equalTo($("message-input").bottom).offset(10).left.right.inset(10).height.equalTo(36),
      events: { tapped: sendMessage }
    },
    {
      type: "text",
      props: { id: "log-text", editable: false, bgcolor: $color("#F0F0F0"), lines: 0 },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.right.bottom.inset(10)
    }
  ]
});

// 初始连接 (可选，也可以只通过按钮连接)
// connectWebSocket();
```

**代码解读**：

1.  `connectWebSocket()` 函数负责创建 `WebSocket` 实例并设置所有事件监听器，然后调用 `ws.open()` 启动连接。
2.  `sendMessage()` 函数检查连接状态，然后通过 `ws.send()` 发送输入框中的文本。
3.  `disconnectWebSocket()` 函数通过 `ws.close()` 关闭连接。
4.  `log-text` 文本视图用于显示收发的消息，`status-label` 显示连接状态。

`$socket` API 是构建实时应用的关键。理解其事件驱动的特性和连接生命周期，是编写健壮 WebSocket 客户端的基础。 
