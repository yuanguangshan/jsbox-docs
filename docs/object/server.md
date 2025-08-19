# request

属性 | 类型 | 读写 | 说明
---|---|---|---
method | string | 只读 | http method
url | string | 只读 | url
headers | json | 只读 | http headers
path | string | 只读 | path
query | json | 只读 | query
contentType | string | 只读 | Content-Type
contentLength | number | 只读 | Content-Length
ifModifiedSince | date | 只读 | If-Modified-Since
ifNoneMatch | bool | 只读 | If-None-Match
acceptsGzip | bool | 只读 | accepts Gzip encoding
localAddressData | data | 只读 | local address data
localAddress | string | 只读 | local address string
remoteAddressData | data | 只读 | remote address data
remoteAddress | string | 只读 | remote address string
hasBody | bool | 只读 | has body
hasByteRange | bool | 只读 | has byte range

# data request

data request 包含所有 request 的属性，同时有如下属性：

属性 | 类型 | 读写 | 说明
---|---|---|---
data | data | 只读 | data
text | string | 只读 | text
json | json | 只读 | json

# file request

file request 包含所有 request 的属性，同时有如下属性：

属性 | 类型 | 读写 | 说明
---|---|---|---
temporaryPath | string | 只读 | temporary file path

# multipart request

multipart request 包含所有 request 的属性，同时有如下属性：

属性 | 类型 | 读写 | 说明
---|---|---|---
arguments | array | 只读 | arguments
files | array | 只读 | files
mimeType | string | 只读 | MIME Type

arguments 包含的数据结构：

属性 | 类型 | 读写 | 说明
---|---|---|---
controlName | string | 只读 | control name
contentType | string | 只读 | Content-Type
mimeType | string | 只读 | MIME Type
data | data | 只读 | data
string | string | 只读 | string
fileName | string | 只读 | file name
temporaryPath | string | 只读 | temporary file path

# response

response 是 handler.response 返回的对象：

属性 | 类型 | 读写 | 说明
---|---|---|---
redirect | string | 读写 | redirect url
permanent | bool | 读写 | permanent
statusCode | number | 读写 | http status code
contentType | string | 读写 | Content-Type
contentLength | string | 读写 | Content-Length
cacheControlMaxAge | number | 读写 | Cache-Control
lastModifiedDate | date | 读写 | Last-Modified
eTag | string | 读写 | ETag
gzipEnabled | bool | 读写 | gzip enabled
hasBody | bool | 读写 | has body

例如你能这样构造一个 response：

```js
return {
  type: "default",
  props: {
    statusCode: 404
  }
}
```

# data response

data response 包含所有 response 的属性，同时有如下属性：

属性 | 类型 | 读写 | 说明
---|---|---|---
text | string | 读写 | text
html | string | 读写 | html
json | json | 读写 | json

# file response

file response 包含所有 response 的属性，同时有如下属性：

属性 | 类型 | 读写 | 说明
---|---|---|---
path | string | 读写 | file path
isAttachment | bool | 读写 | is attachment
byteRange | range | 读写 | byte range

---

## 文件内容解读与示例

### 用途说明

本文档详细描述了 `$server` API 中用于自定义请求处理器（`addHandler`）的**`request` 和 `response` 对象的结构**。理解这些对象的属性，是编写能够精确处理传入 HTTP 请求和生成自定义 HTTP 响应的关键。它们是构建本地 Web 服务、API Mocking 或与本地网络设备进行交互的基础。

### `request` 对象：传入请求的详细信息

`request` 对象封装了所有关于客户端发起的 HTTP 请求的详细信息。它有多种子类型，根据请求体的不同而有所区别。

#### 1. 通用属性

所有 `request` 对象都包含以下通用属性：

-   **`request.method`**: HTTP 请求方法（`GET`, `POST`, `PUT`, `DELETE` 等）。
-   **`request.url`**: 完整的请求 URL。
-   **`request.path`**: 请求的路径部分（不包含查询参数）。
-   **`request.query`**: 请求的查询参数，一个 JSON 对象。
-   **`request.headers`**: 请求头，一个 JSON 对象。
-   **`request.remoteAddress`**: 客户端的 IP 地址。
-   **`request.contentType`**: 请求体的 `Content-Type`。
-   **`request.contentLength`**: 请求体的长度（字节）。

#### 2. 请求体子类型

-   **`data request`**: 当请求体是纯文本、JSON 或其他二进制数据时，`request` 对象会额外包含：
    -   `request.data`: 原始的 `$data` 对象。
    -   `request.text`: 请求体文本内容（如果 `Content-Type` 是文本类型）。
    -   `request.json`: 请求体 JSON 内容（如果 `Content-Type` 是 `application/json`）。

-   **`file request`**: 当请求体是单个文件上传时，`request` 对象会额外包含：
    -   `request.temporaryPath`: 上传文件的临时路径。

-   **`multipart request`**: 当请求体是 `multipart/form-data` 格式时（例如，包含文件上传和普通表单字段），`request` 对象会额外包含：
    -   `request.arguments`: 表单字段，一个数组，每个元素包含 `controlName`, `string` 等。
    -   `request.files`: 上传的文件列表，一个数组，每个元素包含 `fileName`, `temporaryPath` 等。

**示例**：在请求处理器中访问 `request` 对象

```javascript
mockServer.addHandler({
  filter: rules => rules.path === "/api/data",
  response: request => {
    console.log("请求方法:", request.method);
    console.log("请求路径:", request.path);
    console.log("查询参数:", request.query.id);
    console.log("请求头 Host:", request.headers["Host"]);

    if (request.method === "POST" && request.json) {
      console.log("接收到 JSON 数据:", request.json);
      return { type: "data", props: { json: { status: "received", data: request.json } } };
    } else if (request.method === "GET") {
      return { type: "data", props: { text: "Hello GET!" } };
    }
    return { type: "default", props: { statusCode: 400 } };
  }
});
```

### `response` 对象：生成传出响应

`response` 对象用于构建服务器返回给客户端的 HTTP 响应。它也有多种子类型，根据响应体的不同而有所区别。

#### 1. 通用属性

所有 `response` 对象都包含以下通用属性：

-   **`response.statusCode`**: HTTP 状态码（如 `200` OK, `404` Not Found, `500` Internal Server Error）。
-   **`response.contentType`**: 响应内容的 MIME 类型（如 `"application/json"`, `"text/html"`）。
-   **`response.headers`**: 响应头，一个 JSON 对象。
-   **`response.redirect`**: 如果需要重定向，指定重定向的 URL。
-   **`response.permanent`**: 布尔值，与 `redirect` 配合使用，表示是否是永久重定向（301）。

#### 2. 响应体子类型

-   **`data response`**: 用于发送文本、HTML 或 JSON 数据。`props` 中包含：
    -   `text`: 纯文本内容。
    -   `html`: HTML 文本内容。
    -   `json`: JSON 对象，会被自动序列化为 JSON 字符串。

-   **`file response`**: 用于发送本地文件作为响应。`props` 中包含：
    -   `path`: 本地文件的路径。
    -   `isAttachment`: 布尔值，如果为 `true`，浏览器会提示下载文件而不是直接打开。
    -   `byteRange`: 可选，用于支持 HTTP Range 请求，发送文件的一部分。

**示例**：构造不同类型的 `response`

```javascript
// 返回一个简单的文本响应
return { type: "data", props: { text: "Hello, Client!" }, statusCode: 200, contentType: "text/plain" };

// 返回一个 JSON 响应
return { type: "data", props: { json: { message: "Success", code: 0 } }, statusCode: 200, contentType: "application/json" };

// 返回一个 HTML 页面
return { type: "data", props: { html: "<h1>Welcome!</h1>" }, statusCode: 200, contentType: "text/html" };

// 返回一个本地文件
// return { type: "file", props: { path: "assets/image.png", isAttachment: false }, statusCode: 200, contentType: "image/png" };

// 重定向到另一个 URL
// return { type: "default", redirect: "https://www.jsbox.com", permanent: false };
```

### 总结

`request` 和 `response` 对象是 `$server` API 中构建自定义 HTTP 处理器的核心数据结构。通过深入理解它们的属性和子类型，你可以创建出功能强大、灵活多变的本地 Web 服务，实现各种复杂的网络交互逻辑。 
