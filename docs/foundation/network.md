> 网络模块将是 JSBox 里面最重要的模块之一，他能帮助你完成很多需求。

# $http.request(object)

发起一个 HTTP 请求：

```js
$http.request({
  method: "POST",
  url: "https://apple.com",
  header: {
    k1: "v1",
    k2: "v2"
  },
  body: {
    k1: "v1",
    k2: "v2"
  },
  handler: function(resp) {
    const data = resp.data;
  }
})
```

参数 | 类型 | 说明
---|---|---
method | string | GET/POST/DELETE 等
url | string | 链接
header | object | http header
body | object | http body
timeout | number | 请求超时
form | object | form-data 参数
files | array | 文件列表
proxy | json | 代理设置
progress | function | upload/download 中进度回调
showsProgress | bool | 是否显示进度条
message | string | upload/download 中的提示语
handler | function | 回调函数

`body` 可以是一个 JSON 结构或是一个二进制数据，

如果 `body` 是一个 JSON 结构：

- 当 header 中的 `Content-Type` 为 `application/json` 时，body 将会以 `json` 的形式编码
- 当 header 中的 `Content-Type` 为 `application/x-www-form-urlencoded` 时，body 将会转换成 `a=b&c=d` 的 query 结构
- `Content-Type` 默认值是 `application/json`

如果 `body` 是一个二进制数据：

这 http body 不会进行编码，会直接使用传进来的数据。

`form` 和 `files` 样例见 `$http.upload`。

返回数据 `resp` 的结构：

参数 | 类型 | 说明
---|---|---
data | string | json 数据会自动 parse
rawData | data | 原始返回的二进制数据
response | response | [请参考](object/response.md)
error | error | [请参考](object/error.md)

代理设置：

```js
{
  proxy: {
    "HTTPEnable": true,
    "HTTPProxy": "",
    "HTTPPort": 0,
    "HTTPSEnable": true,
    "HTTPSProxy": "",
    "HTTPSPort": 0
  }
}
```

# $http.get(object)

发起一个 `GET` 请求，例如：

```js
$http.get({
  url: "https://apple.com",
  handler: function(resp) {
    const data = resp.data;
  }
})
```

除 `method` 为 `GET` 以外，参数和 `$http.request` 一致。

# $http.post(object)

发起一个 `POST` 请求，例如：

```js
$http.post({
  url: "https://apple.com",
  handler: function(resp) {
    const data = resp.data;
  }
})
```

除 `method` 为 `POST` 以外，参数和 `$http.request` 一致。

# $http.download(object)

参数和上述完全一样，不同之处在于此方法返回的 `data` 是一个二进制数据：

```js
$http.download({
  url: "https://images.apple.com/v/ios/what-is/b/images/performance_large.jpg",
  showsProgress: true, // Optional, default is true
  backgroundFetch: true, // Optional, default is false
  progress: function(bytesWritten, totalBytes) {
    const percentage = bytesWritten * 1.0 / totalBytes;
  },
  handler: function(resp) {
    $share.sheet(resp.data)
  }
})
```

# $http.upload(object)

文件上传接口，此处以[七牛云](https://www.qiniu.com/)为例：

```js
$photo.pick({
  handler: function(resp) {
    const image = resp.image;
    if (image) {
      $http.upload({
        url: "http://upload.qiniu.com/",
        form: {
          token: "<token>"
        },
        files: [
          {
            "image": image,
            "name": "file",
            "filename": "file.png"
          }
        ],
        progress: function(percentage) {

        },
        handler: function(resp) {
          
        }
      })
    }
  }
})
```

从相册选取一张图片上传到七牛云，需要填入你自己的 token，此处仅为示例。

`form` 参数表示 form-data 中其他的参数，例如上述的 `token`。

`files` 参数文件结构定义：

参数 | 类型 | 说明
---|---|---
image | object | 图片
data | object | 二进制数据
name | string | 上传表单中的名称
filename | string | 上传之后的文件名
content-type | string | 文件格式

# $http.startServer(object)

启动一个 Web 服务器，将本地目录放到服务器以提供 HTTP 接口访问：

```js
$http.startServer({
  port: 5588, // port number
  path: "", // script root path
  handler: function(result) {
    const url = result.url;
  }
})
```

这将会把脚本的根目录作为 server 的根目录，你可以通过获取到的 url 访问这个目录，我们提供了一个 Web 界面供用户在浏览器中访问。

同时这个 Web 服务器提供以下几个 HTTP 接口来进行操作：

- `GET` /list?path=path 获得文件列表
- `GET` /download?path=path 下载文件
- `POST` /upload `{"files[]": file}` 上传文件
- `POST` /move `{"oldPath": "", "newPath": ""}` 移动文件
- `POST` /delete `{"path": ""}` 删除文件
- `POST` /create `{"path": ""}` 创建文件夹

请使用你的 HTTP 相关知识理解上述内容，在此不再赘述。

# $http.stopServer()

将启动的 Web 服务器停掉：

```js
$http.stopServer()
```

# $http.shorten(object)

生成短连接：

```js
$http.shorten({
  url: "https://apple.com",
  handler: function(url) {

  }
})
```

# $http.lengthen(object)

展开一个短连接：

```js
$http.lengthen({
  url: "http://t.cn/RJZxkFD",
  handler: function(url) {

  }
})
```

# $network.ifa_data

获得网卡`接收/发送`数据：

```js
const ifa_data = $network.ifa_data;
console.log(ifa_data)
```

样例输出：

```json
{
  "en0" : {
    "received" : 2581234688,
    "sent" : 469011456
  },
  "awdl0" : {
    "received" : 2231296,
    "sent" : 8180736
  },
  "utun0" : {
    "received" : 0,
    "sent" : 0
  },
  "pdp_ip1" : {
    "received" : 2048,
    "sent" : 2048
  },
  "pdp_ip0" : {
    "received" : 2215782400,
    "sent" : 211150848
  },
  "en2" : {
    "received" : 5859328,
    "sent" : 6347776
  },
  "utun2" : {
    "received" : 0,
    "sent" : 0
  },
  "lo0" : {
    "received" : 407684096,
    "sent" : 407684096
  },
  "utun1" : {
    "received" : 628973568,
    "sent" : 35312640
  }
}
```

# $network.interfaces

获取当前设备所有的网络接口：

```js
const interfaces = $network.interfaces;

// E.g. { 'en0/ipv4': 'x.x.x.x' }
```

# $network.startPinging(object)

开始 Ping:

```js
$network.startPinging({
  host: "google.com",
  timeout: 2.0, // default
  period: 1.0, // default
  payloadSize: 56, // default
  ttl: 49, // default
  didReceiveReply: function(summary) {

  },
  didReceiveUnexpectedReply: function(summary) {

  },
  didSendPing: function(summary) {

  },
  didTimeout: function(summary) {

  },
  didFail: function(error) {

  },
  didFailToSendPing: function(response) {

  }
})
```

`summary` 结构：

```json
{
  "sequenceNumber": 0,
  "payloadSize": 0,
  "ttl": 0,
  "host": "",
  "sendDate": null,
  "receiveDate": null,
  "rtt": 0,
  "status": 0
}
```

更多信息请参考：https://github.com/lmirosevic/GBPing

# $network.stopPinging()

停止 Ping。

# $network.proxy_settings

与 [CFNetworkCopySystemProxySettings](https://developer.apple.com/documentation/cfnetwork/1426754-cfnetworkcopysystemproxysettings) 效果相同。

---

## 文件内容解读与示例

### 用途说明

`network.md` 文档涵盖了 JSBox 中两个至关重要的网络相关 API：

-   **`$http`**: 用于发起 HTTP/HTTPS 网络请求，包括获取数据、上传文件、甚至在本地运行一个 Web 服务器。
-   **`$network`**: 用于获取设备网络状态信息和执行低级别的网络诊断（如 Ping）。

这两个模块是任何需要与互联网或局域网交互的 JSBox 脚本的基石。

### `$http` API：HTTP/HTTPS 请求

`$http` 模块是你的脚本与 Web 服务进行数据交换的核心。所有请求都是异步的，通常通过 `handler` 回调函数处理结果。

#### 1. 核心请求方法：`$http.request(options)`

-   **用途**: 最通用、最灵活的 HTTP 请求方法，支持所有 HTTP 方法（GET, POST, PUT, DELETE 等）、自定义请求头、请求体、超时设置、代理等。
-   **`body` 处理**: 
    -   如果 `Content-Type` 是 `application/json`，`body` 对象会被自动序列化为 JSON 字符串。
    -   如果 `Content-Type` 是 `application/x-www-form-urlencoded`，`body` 对象会被转换为 URL 编码的查询字符串。
    -   如果 `body` 是 `$data` 对象，则直接作为原始二进制数据发送。
-   **`resp` 返回结构**: `resp` 对象包含 `data`（已解析的 JSON 或字符串）、`rawData`（原始二进制数据）、`response`（HTTP 响应对象）和 `error`（错误信息）。

**示例**：发起一个 POST 请求

```javascript
$http.request({
  method: "POST",
  url: "https://jsonplaceholder.typicode.com/posts", // 这是一个模拟 API
  header: {
    "Content-Type": "application/json"
  },
  body: {
    title: "JSBox Post",
    body: "This is a test post from JSBox.",
    userId: 1
  },
  handler: (resp) => {
    if (resp.data) {
      $ui.alert(`POST 请求成功！\nID: ${resp.data.id}\n标题: ${resp.data.title}`);
    } else {
      $ui.alert(`POST 请求失败: ${resp.error.localizedDescription}`);
    }
  }
});
```

#### 2. 快捷请求方法：`$http.get(options)` / `$http.post(options)`

-   **用途**: `$http.request` 的简化版本，分别用于发起 GET 和 POST 请求。参数与 `$http.request` 类似，只是 `method` 已固定。

**示例**：获取公共 IP 地址

```javascript
$http.get({
  url: "https://api.ipify.org?format=json",
  handler: (resp) => {
    if (resp.data && resp.data.ip) {
      $ui.alert(`你的公共 IP 地址是: ${resp.data.ip}`);
    } else {
      $ui.alert("获取 IP 失败。");
    }
  }
});
```

#### 3. 文件下载：`$http.download(options)`

-   **用途**: 下载文件。返回的 `resp.data` 是一个 `$data` 对象。
-   **特性**: 支持 `showsProgress`（显示系统进度条）、`progress` 回调（实时获取下载进度）、`backgroundFetch`（后台下载）。

**示例**：下载一张图片并显示

```javascript
$http.download({
  url: "https://picsum.photos/id/237/400/300", // 随机图片
  showsProgress: true, // 显示下载进度条
  progress: (bytesWritten, totalBytes) => {
    const percentage = Math.round((bytesWritten / totalBytes) * 100);
    console.log(`下载进度: ${percentage}%`);
  },
  handler: (resp) => {
    if (resp.data) {
      $ui.alert({
        title: "图片下载完成",
        message: "点击查看图片",
        actions: [
          { title: "查看", handler: () => $quicklook.open({ image: resp.data }) }
        ]
      });
    } else {
      $ui.alert(`图片下载失败: ${resp.error.localizedDescription}`);
    }
  }
});
```

#### 4. 文件上传：`$http.upload(options)`

-   **用途**: 上传文件（通常是 `multipart/form-data` 格式）。
-   **参数**: `form`（表单字段）、`files`（文件列表，每个文件可以是 `$image` 或 `$data` 对象，并指定 `name` 和 `filename`）。

**示例**：上传图片（概念性，需真实上传服务器）

```javascript
// 假设 imageToUpload 是一个 $image 对象，例如通过 $photo.pick() 获取
// $http.upload({
//   url: "你的上传服务器URL",
//   form: { user_id: "123", description: "我的照片" },
//   files: [
//     { image: imageToUpload, name: "photo_file", filename: "my_photo.png", "content-type": "image/png" }
//   ],
//   progress: (percentage) => { console.log(`上传进度: ${percentage}%`); },
//   handler: (resp) => { /* 处理上传结果 */ }
// });
```

#### 5. 本地 Web 服务器：`$http.startServer(options)` / `$http.stopServer()`

-   **用途**: 在 JSBox 内部启动一个轻量级的 HTTP 服务器，可以将脚本目录下的文件通过 HTTP 协议提供访问。这对于本地调试、API Mocking 或创建基于 Web 的 UI 非常有用。

**示例**：启动本地服务器并访问

```javascript
$http.startServer({
  port: 8080, // 指定端口
  path: "", // 将脚本根目录作为服务器根目录
  handler: (result) => {
    $ui.alert(`本地服务器已启动: ${result.url}`);
    $app.openURL(result.url); // 在浏览器中打开
  }
});
// $http.stopServer(); // 在需要时停止服务器
```

#### 6. 短链接服务：`$http.shorten(options)` / `$http.lengthen(options)`

-   **用途**: 分别用于生成短链接和展开短链接。

### `$network` API：网络信息与诊断

`$network` 模块提供了获取设备网络状态和执行低级别网络诊断的功能。

#### 1. 网络流量统计：`$network.ifa_data`

-   **用途**: 获取各个网络接口（如 Wi-Fi, 蜂窝数据）的接收和发送字节数统计。

**示例**：

```javascript
const trafficData = $network.ifa_data;
console.log("网络流量数据:", JSON.stringify(trafficData, null, 2));
// 你可以遍历 trafficData 来获取每个接口的 received 和 sent 字节数
```

#### 2. 网络接口信息：`$network.interfaces`

-   **用途**: 获取当前设备所有网络接口的 IP 地址信息。

**示例**：

```javascript
const interfaces = $network.interfaces;
console.log("网络接口信息:", JSON.stringify(interfaces, null, 2));
// 例如: { "en0/ipv4": "192.168.1.100" }
```

#### 3. Ping 工具：`$network.startPinging(options)` / `$network.stopPinging()`

-   **用途**: 执行 Ping 操作，用于测试网络连通性和延迟。支持详细的回调事件来监控 Ping 过程。

**示例**：Ping Google.com

```javascript
$network.startPinging({
  host: "google.com",
  count: 5, // Ping 5次
  didReceiveReply: (summary) => {
    console.log(`收到回复: ${summary.host} RTT: ${summary.rtt}ms`);
  },
  didTimeout: (summary) => {
    console.log(`请求超时: ${summary.host}`);
  },
  didFail: (error) => {
    console.log(`Ping 失败: ${error.localizedDescription}`);
  },
  didFinish: () => {
    console.log("Ping 结束。");
  }
});
// $network.stopPinging(); // 在需要时停止 Ping
```

### 总结

`$http` 和 `$network` 模块是 JSBox 脚本与外部世界进行数据交换和网络诊断的基石。无论是从 Web API 获取数据、上传文件、运行本地服务器，还是监控网络状态、执行 Ping 操作，这两个模块都提供了强大而灵活的工具。
