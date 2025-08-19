# $server

你可以使用 $http.startServer 来创建一个简单的 Web 服务器，可以将文件放到上面，然后通过 HTTP 链接下载。当你需要自定义 Request Handler 时，可以使用 $server 相关的接口：

```js
const server = $server.new();

const options = {
  port: 6060, // Required
  bonjourName: "", // Optional
  bonjourType: "", // Optional
};

server.start(options);
```

# $server.start(object)

使用简单的参数快速启动一个 HTTP 服务器：

```js
const server = $server.start({
  port: 6060,
  path: "assets/website",
  handler: () => {
    $app.openURL("http://localhost:6060/index.html");
  }
});
```

# server.stop()

停止 Web 服务器。

# server.listen(events)

通过这个接口可以监听事件：

```js
server.listen({
  didStart: server => {
    $delay(1, () => {
      $app.openURL(`http://localhost:${port}`);
    });
  },
  didConnect: server => {},
  didDisconnect: server => {},
  didStop: server => {},
  didCompleteBonjourRegistration: server => {}
});
```

# server.addHandler(object)

注册一个 Request Handler:

```js
const handler = {};

// Handler filter
handler.filter = rules => {
  const method = rules.method;
  const url = rules.url;
  // rules.headers, rules.path, rules.query;
  return "data"; // default, data, file, multipart, urlencoded
}

// Handler response
handler.response = request => {
  const method = request.method;
  const url = request.url;
  return {
    type: "data", // default, data, file, error
    props: {
      html: "<html><body style='font-size: 300px'>Hello!</body></html>"
      // json: {
      //   "status": 1,
      //   "values": ["a", "b", "c"]
      // }
    }
  };
}

// Handler async response
handler.asyncResponse = (request, completion) => {
const method = request.method;
  const url = request.url;
  completion({
    type: "data", // default, data, file, error
    props: {
      html: "<html><body style='font-size: 300px'>Hello!</body></html>"
      // json: {
      //   "status": 1,
      //   "values": ["a", "b", "c"]
      // }
    }
  });
}
```

 # handler.filter

 filter 是一个函数，在这里你可以根据传入的 rules 决定对这些规则使用什么样的 request，rules 结构：

```json
{
  "method": "",
  "url": "",
  "headers": {

  },
  "path": "",
  "query": {

  }
}
```

request 类型包括：`default`, `data`, `file`, `multipart`, `urlencoded`。

从 v1.51.0 开始，你可以返回一个字典，用来覆盖之前的规则，例如：

```js
handler.filter = rules => {
  return {
    "type": "data",
    "method": "GET"
  }
}
```

当你需要重定向一个请求，或是将 `POST` 改写为 `GET` 等需求时，可能会用得上。

# handler.response

这个函数传入一个 request，根据需要返回一个 response 用于完成对这个请求的响应，例如这是一种最简单的 response：

```js
{
  type: "data",
  props: {
    html: "<html><body style='font-size: 300px'>Hello!</body></html>"
  }
}
```

创建 response 可以指定类型为：`default`, `data`, `file`，通过不同的 props 来初始化一个 response。

`data` 类型支持通过 `html`, `text` 或 `json` 字段来构造。`file` 类型支持通过 `path` 来构造。

其他支持的参数：

属性 | 类型 | 说明
---|---|---
contentType | string | content type
contentLength | number | content length
statusCode | number | status code
cacheControlMaxAge | number | cache control max age
lastModifiedDate | Date | last modified date
eTag | string | E-Tag
gzipEnabled | bool | gzip enabled
headers | object | HTTP headers

# server.clearHandlers()

清除掉所有已注册的 Request Handlers。

完整例子请见：https://github.com/cyanzhong/xTeko/blob/master/extension-demos/server.js

Request 对象和 Response 对象包括多种类型，请参考[详细文档](object/server.md)。

---

## 文件内容解读与示例

### 用途说明

`$server` API 提供了在 JSBox 内部**运行一个本地 HTTP 服务器**的强大功能。与 `$http.startServer` 这种主要用于提供静态文件服务的简化接口不同，`$server` 允许你**自定义请求处理逻辑**，从而实现复杂的本地 Web 服务、API Mocking、或与本地网络设备进行交互。这对于开发需要模拟后端、提供本地数据接口或构建复杂本地 Web 应用的脚本非常有用。

### 核心概念：自定义请求处理

`$server` 的核心在于其**请求处理器（Request Handler）**机制。你可以注册一个或多个处理器，每个处理器都包含两个主要部分：

1.  **`filter` (过滤器)**: 决定当前处理器是否应该响应某个传入的 HTTP 请求。你可以根据请求的方法（GET/POST）、URL 路径、请求头等来设置过滤规则。
2.  **`response` 或 `asyncResponse` (响应)**: 当 `filter` 匹配成功后，这个部分负责生成并返回 HTTP 响应。你可以返回 HTML、JSON、纯文本，甚至直接返回本地文件。

### 主要方法与功能详解

#### 1. 服务器生命周期管理

-   **`$server.new()`**: 创建一个新的服务器实例。你可以创建多个服务器实例，但它们需要监听不同的端口。
-   **`server.start(options)`**: 启动服务器。`options` 包含 `port`（必填，服务器监听的端口号）、`bonjourName` 和 `bonjourType`（可选，用于局域网服务发现）。
-   **`server.stop()`**: 停止服务器。
-   **`server.listen(events)`**: 监听服务器的生命周期事件，如 `didStart`（服务器启动成功）、`didConnect`（有客户端连接）、`didStop`（服务器停止）。

**示例**：启动一个服务器并监听其状态

```javascript
const myServer = $server.new();
myServer.listen({
  didStart: (server) => {
    $ui.alert(`本地服务器已启动在: ${server.url}`);
    console.log(`服务器已启动在: ${server.url}`);
  },
  didStop: () => {
    $ui.toast("服务器已停止。");
    console.log("服务器已停止。");
  },
  didConnect: () => {
    console.log("有客户端连接到服务器。");
  }
});

// 启动服务器
myServer.start({ port: 8080 });

// 可以在其他地方调用 myServer.stop() 来停止它
// $delay(10, () => myServer.stop()); // 10秒后自动停止
```

#### 2. 请求处理：`server.addHandler(handler)`

这是 `$server` 最强大的功能。你可以注册一个或多个 `handler` 对象来处理不同的请求。

-   **`handler.filter(rules)`**: 
    -   `rules` 对象包含 `method` (请求方法), `url`, `path`, `query` (查询参数), `headers` (请求头)。
    -   返回 `true` 表示该 `handler` 处理此请求；返回 `false` 表示不处理。
    -   从 v1.51.0 开始，你也可以返回一个对象来修改请求规则，例如重定向或修改请求方法。

-   **`handler.response(request)`**: 
    -   **同步**响应。接收 `request` 对象，返回一个响应对象。
    -   响应对象包含 `type` (`data`, `file`, `error`) 和 `props`（具体内容，如 `html`, `text`, `json` 或 `path`）。

-   **`handler.asyncResponse(request, completion)`**: 
    -   **异步**响应。适用于需要进行网络请求或耗时操作才能返回响应的场景。
    -   `completion` 是一个回调函数，在异步操作完成后调用它来发送响应。

**示例**：一个简单的 API Mocking 服务器

```javascript
const mockServer = $server.new();
mockServer.start({ port: 8081 });

mockServer.addHandler({
  // 过滤器：只处理 GET /api/time 的请求
  filter: (rules) => {
    return rules.method === "GET" && rules.path === "/api/time";
  },
  // 响应：返回当前时间戳的 JSON
  response: (request) => {
    return {
      type: "data",
      props: {
        json: { timestamp: Date.now(), message: "Hello from JSBox local server!" }
      },
      statusCode: 200,
      headers: { "Content-Type": "application/json" }
    };
  }
});

mockServer.addHandler({
  // 过滤器：处理 GET /static/hello.html 的请求
  filter: (rules) => {
    return rules.method === "GET" && rules.path === "/static/hello.html";
  },
  // 响应：返回一个本地 HTML 文件
  response: (request) => {
    // 假设你的脚本 assets 目录下有一个 hello.html 文件
    const htmlContent = $file.read("assets/hello.html");
    if (htmlContent) {
      return {
        type: "data",
        props: { html: htmlContent.string },
        statusCode: 200,
        headers: { "Content-Type": "text/html" }
      };
    } else {
      return { type: "error", props: { statusCode: 404, message: "File not found" } };
    }
  }
});

// 可以在浏览器中访问 http://localhost:8081/api/time 和 http://localhost:8081/static/hello.html
// 记得在脚本结束时调用 mockServer.stop()
// $delay(30, () => mockServer.stop()); // 30秒后自动停止
```

#### 3. 清除处理器：`server.clearHandlers()`

-   **用途**: 移除所有已注册的请求处理器。这在需要动态改变服务器行为时非常有用。

### 总结

`$server` API 是 JSBox 中一个非常高级且强大的模块，它为开发者提供了在设备本地运行自定义 HTTP 服务器的能力。通过灵活配置 `filter` 和 `response`，你可以实现各种复杂的本地网络服务，极大地扩展了 JSBox 脚本的应用场景。 
