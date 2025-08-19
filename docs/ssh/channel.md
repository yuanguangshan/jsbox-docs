# session.channel

session 中的 channel 对象提供执行脚本等操作，结构如下：

参数 | 类型 | 说明
---|---|---
session | session | session 实例
bufferSize | number | buffer size
type | number | 类型
lastResponse | string | 响应
requestPty | bool | request pty
ptyTerminalType | number | pty terminal type
environmentVariables | json | 环境变量

# channel.execute(object)

执行脚本：

```js
channel.execute({
  script: "ls -l /var/lib/",
  timeout: 0,
  handler: function(result) {
    console.log(`response: ${result.response}`)
    console.log(`error: ${result.error}`)
  }
})
```

# channel.write(object)

执行 command:

```js
channel.write({
  command: "",
  timeout: 0,
  handler: function(result) {
    console.log(`success: ${result.success}`)
    console.log(`error: ${result.error}`)
  }
})
```

# channel.upload(object)

上传本地文件到服务器：

```js
channel.upload({
  path: "resources/notes.md",
  dest: "/home/user/notes.md",
  handler: function(success) {
    console.log(`success: ${success}`)
  }
})
```

# channel.download(object)

从服务器下载文件到本地：

```js
channel.download({
  path: "/home/user/notes.md",
  dest: "resources/notes.md",
  handler: function(success) {
    console.log(`success: ${success}`)
  }
})
```

---

## 文件内容解读与示例

### 用途说明

本文档详细介绍了 `session.channel` 对象的功能。一旦通过 `$ssh.connect` 建立了与服务器的会话（Session），`channel` 对象就是你用来**与服务器进行命令交互和基本文件传输**的主要工具。它封装了在远程服务器上执行命令、读写数据流以及上传下载文件的核心操作。

### 核心概念：SSH 通道

在一个 SSH 会话中，可以打开多个通道（Channel）。你可以将每个通道想象成在已建立的安全连接内部的一条独立的通信管道。JSBox 中的 `channel` 对象主要用于两种场景：

1.  **执行 Shell 命令**: 这是最常见的用途，比如运行 `ls`, `docker ps`, `git pull` 等。
2.  **SCP 文件传输**: `upload` 和 `download` 方法实际上是使用 SCP (Secure Copy Protocol) 协议，它在 SSH 连接之上提供了一种简单的文件传输方式。

### API 详解

1.  **`channel.execute(options)`**
    -   **作用**: 在远程服务器上执行一条完整的 shell 命令，并等待其完成后返回结果。
    -   **`script`**: 要执行的命令字符串。
    -   **`handler(result)`**: 命令执行结束后的回调函数。`result` 对象包含两个关键属性：
        -   `result.response`: 命令的标准输出（stdout）。
        -   `result.error`: 命令的标准错误输出（stderr）。
    -   **适用场景**: 适用于执行那些会立即返回结果的、非交互式的命令。

2.  **`channel.write(options)`**
    -   **作用**: 向通道写入数据。这通常用于需要与正在运行的进程进行交互的场景，但在此文档中用法较为简化。
    -   **适用场景**: 在 JSBox 的上下文中，它更像是一个简化的命令发送方式，适用于那些你不需要关心其输出的命令。

3.  **`channel.upload(options)`**
    -   **作用**: 将一个本地文件**上传**到远程服务器。
    -   **`path`**: 本地文件的路径（相对于脚本）。
    -   **`dest`**: 远程服务器上的目标路径和文件名。
    -   **`handler(success)`**: 上传完成后的回调，`success` 是一个布尔值，表示是否成功。

4.  **`channel.download(options)`**
    -   **作用**: 从远程服务器**下载**一个文件到本地。
    -   **`path`**: 远程服务器上的文件路径。
    -   **`dest`**: 下载后保存在本地的路径和文件名。
    -   **`handler(success)`**: 下载完成后的回调，`success` 是一个布尔值。

### 示例：执行命令并根据结果上传文件

假设我们需要检查服务器上某个目录是否存在，如果不存在，就创建一个，然后上传一个配置文件。

```javascript
// (需先通过 $ssh.connect 获得 session 对象)

const remoteDir = "/home/user/app_config";
const localConfigFile = "assets/config.json";
const remoteConfigFile = `${remoteDir}/config.json`;

const channel = session.channel;

// 1. 检查远程目录是否存在
channel.execute({
  script: `if [ -d "${remoteDir}" ]; then echo "exists"; else echo "not_exists"; fi`,
  handler: (result) => {
    if (result.error) {
      $ui.error(`检查目录失败: ${result.error}`);
      return;
    }

    const dirExists = result.response.trim() === "exists";

    if (dirExists) {
      console.log("目录已存在，直接上传文件。");
      uploadConfig();
    } else {
      console.log("目录不存在，正在创建...");
      // 2. 如果目录不存在，则创建目录
      channel.execute({
        script: `mkdir -p ${remoteDir}`,
        handler: (mkdirResult) => {
          if (mkdirResult.error) {
            $ui.error(`创建目录失败: ${mkdirResult.error}`);
          } else {
            console.log("目录创建成功，开始上传文件。");
            uploadConfig();
          }
        }
      });
    }
  }
});

// 3. 封装的上传函数
function uploadConfig() {
  $ui.loading("正在上传配置文件...");
  channel.upload({
    path: localConfigFile,
    dest: remoteConfigFile,
    handler: (success) => {
      $ui.loading(false);
      if (success) {
        $ui.success("配置文件上传成功！");
      } else {
        $ui.error("上传失败。");
      }
    }
  });
}
```

### 总结

`session.channel` 是执行远程操作的核心。通过 `execute` 方法，你可以将服务器的 shell 能力集成到你的脚本中，实现复杂的自动化任务。而 `upload` 和 `download` 则为脚本与远程服务器之间的文件交换提供了便捷的桥梁。掌握 `channel` 的用法是发挥 JSBox SSH 功能强大能力的关键。