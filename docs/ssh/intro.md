# Secure Shell

从 v1.14.0 开始，JSBox 支持 SSH，你可以通过简单的 JavaScript 连接至你的服务器，这里有一个完整的例子来帮你有个宏观的了解：[ssh-example](https://github.com/cyanzhong/xTeko/tree/master/extension-demos/ssh-example)。

JSBox 提供的 SSH 相关接口主要包括三个方面：

- session
- channel
- sftp

通过这些接口你可以连接服务器，执行脚本，也能上传下载文件。

JSBox 的 SSH 接口基于 [NMSSH](https://github.com/NMSSH/NMSSH) 的实现，你可以参考 NMSSH 的文档来了解整体设计。

# $ssh.connect(object)

通过密码或公钥私钥连接至服务器，同时能执行一段脚本：

```js
$ssh.connect({
  host: "",
  port: 22,
  username: "",
  public_key: "",
  private_key: "",
  // password: "",
  script: "ls -l /var/lib/",
  handler: function(session, response) {
    console.log(`connect: ${session.connected}`)
    console.log(`authorized: ${session.authorized}`)
    console.log(`response: ${response}`)
  }
})
```

这个例子通过 `public_key` 和 `private_key` 来授权，你可以将密钥文件打包放在安装包里，正如 [ssh-example](https://github.com/cyanzhong/xTeko/tree/master/extension-demos/ssh-example) 里面所示例的那样。

这个接口会返回一个 session 实例和 response，session 对象结构：

参数 | 类型 | 说明
---|---|---
host | string | 主机
port | number | 端口
username | string | 用户名
timeout | number | 超时时间
lastError | error | 错误
fingerprintHash | string | fingerprint
banner | string | banner
remoteBanner | string | remote banner
connected | bool | 是否连接
authorized | bool | 是否授权
channel | channel | channel 实例
sftp | sftp | sftp 实例

通过 session 对象中的 channel 和 sftp，我们可以完成后续的各种操作。

# $ssh.disconnect()

关掉 JSBox 开启的所有 SSH session:

```js
$ssh.disconnect()
```

---

## 文件内容解读与示例

### 用途说明

本文档是 JSBox 中 **SSH（Secure Shell）** 功能的入门介绍。SSH 是一种网络协议，用于在不安全的网络上安全地操作网络服务。通过 JSBox 的 SSH 功能，开发者可以直接从脚本中连接到远程服务器，执行命令，并传输文件，极大地扩展了 JSBox 的自动化能力，使其能够与后端服务器进行深度交互。

### 核心概念

JSBox 的 SSH 功能主要围绕三个核心对象展开：

1.  **Session (会话)**: 代表了与远程服务器的一次完整连接。你需要先建立一个 `session`，然后才能进行后续操作。它包含了连接状态、授权信息以及 `channel` 和 `sftp` 的实例。

2.  **Channel (通道)**: 在一个已建立的 `session` 之上，`channel` 用于**执行命令和脚本**。你可以通过它向服务器发送指令（如 `ls`, `pwd`），并获取返回结果。

3.  **SFTP (SSH File Transfer Protocol)**: 同样建立在 `session` 之上，`sftp` 专门用于**文件传输**。它提供了一套完整的文件系统操作，如上传、下载、移动、删除文件和目录等。

JSBox 的实现基于 **NMSSH** 这个 Objective-C 库，为熟悉 iOS 原生开发的开发者提供了额外的背景信息。

### API 详解

1.  **`$ssh.connect(options)`**
    -   **作用**: 这是发起 SSH 连接的入口点。它会尝试连接到指定的服务器，并进行身份验证。
    -   **认证方式**: 支持两种主流的认证方法：
        -   **密码认证**: 提供 `password` 属性。
        -   **密钥认证**: 提供 `public_key` 和 `private_key` 的内容（通常是读取密钥文件获得）。这种方式更安全，也是服务器上推荐的配置。
    -   **直接执行脚本**: `script` 参数是一个方便的快捷方式，可以在连接成功后立即执行一条命令，其输出会通过 `handler` 的 `response` 参数返回。
    -   **回调函数 `handler(session, response)`**: 
        -   `session`: 连接成功后返回的会话对象。这是后续所有操作（通过 `channel` 和 `sftp`）的起点。
        -   `response`: `script` 参数执行后返回的文本结果。

2.  **`$ssh.disconnect()`**
    -   **作用**: 一个全局的断开连接方法。它会关闭所有由 JSBox 建立的 SSH 会话，用于在脚本结束时清理连接资源。

### 示例：连接服务器并列出根目录文件

```javascript
// 服务器连接信息
const connectionInfo = {
  host: "your_server_ip",
  port: 22,
  username: "your_username",
  // 推荐使用密钥认证
  // private_key: $file.read("assets/id_rsa").string,
  // public_key: $file.read("assets/id_rsa.pub").string,
  password: "your_password", // 或者使用密码
  script: "ls -l /" // 连接成功后执行的命令
};

$ui.loading(true);

// 发起连接
$ssh.connect({
  ...connectionInfo,
  handler: (session, response) => {
    $ui.loading(false);

    // 检查连接和授权状态
    if (session.connected && session.authorized) {
      $ui.success("连接成功!");
      console.log("服务器根目录文件列表:");
      console.log(response); // 打印 script 命令的返回结果

      // 你可以在这里继续使用 session.channel 或 session.sftp 进行更多操作
      // 例如：session.channel.execute(...)

    } else {
      $ui.error("连接失败");
      console.error("Last Error:", session.lastError);
    }

    // 完成操作后，可以选择断开连接
    // $ssh.disconnect();
  }
});
```

**准备工作**:

-   将 `your_server_ip`, `your_username`, `your_password` 替换为你的真实服务器信息。
-   如果使用密钥认证，你需要先生成密钥对，并将公钥 `id_rsa.pub` 的内容添加到服务器的 `~/.ssh/authorized_keys` 文件中。然后将私钥 `id_rsa` 文件添加到脚本的 `assets` 目录中，并通过 `$file.read` 读取。

### 总结

`$ssh.connect` 是打通 JSBox 与远程服务器的第一步。通过它建立一个安全的 `session`，开发者就获得了在服务器上执行命令和管理文件的能力，为开发诸如服务器监控、自动化部署、文件同步等高级功能的脚本奠定了基础。