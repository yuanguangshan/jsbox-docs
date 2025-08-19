# session.sftp

session 中的 sftp 对象提供对文件相关的操作，结构如下：

参数 | 类型 | 说明
---|---|---
session | session | session 实例
bufferSize | number | buffer size
connected | bool | 是否连接

# sftp.connect()

创建 SFTP 连接：

```js
await sftp.connect();
```

# sftp.moveItem(object)

移动文件：

```js
sftp.moveItem({
  src: "/home/user/notes.md",
  dest: "/home/user/notes-new.md",
  handler: function(success) {

  }
})
```

# sftp.directoryExists(object)

判断文件夹是否存在：

```js
sftp.directoryExists({
  path: "/home/user/notes.md",
  handler: function(exists) {

  }
})
```

# sftp.createDirectory(object)

创建文件夹：

```js
sftp.createDirectory({
  path: "/home/user/folder",
  handler: function(success) {

  }
})
```

# sftp.removeDirectory(object)

删除文件夹：

```js
sftp.removeDirectory({
  path: "/home/user/folder",
  handler: function(success) {

  }
})
```

# sftp.contentsOfDirectory(object)

列出文件夹中所有的文件：

```js
sftp.contentsOfDirectory({
  path: "/home/user/folder",
  handler: function(contents) {

  }
})
```

# sftp.infoForFile(object)

获取一个文件的属性：

```js
sftp.infoForFile({
  path: "/home/user/notes.md",
  handler: function(file) {

  }
})
```

file 的数据结构：

参数 | 类型 | 说明
---|---|---
filename | string | 文件名
isDirectory | bool | 是否文件夹
modificationDate | date | 修改时间
lastAccess | date | 最后访问时间
fileSize | number | 文件大小
ownerUserID | number | owner user id
ownerGroupID | number | owner group id
permissions | string | 权限
flags | number | flags

# sftp.fileExists(object)

判断文件是否存在：

```js
sftp.fileExists({
  path: "/home/user/notes.md",
  handler: function(exists) {

  }
})
```

# sftp.createSymbolicLink(object)

创建符号链接：

```js
sftp.createSymbolicLink({
  path: "/home/user/notes.md",
  dest: "/home/user/notes-symbolic.md",
  handler: function(success) {

  }
})
```

# sftp.removeFile(object)

删除文件：

```js
sftp.removeFile({
  path: "/home/user/notes.md",
  handler: function(success) {

  }
})
```

# sftp.contents(object)

获取文件二进制数据：

```js
sftp.contents({
  path: "/home/user/notes.md",
  handler: function(file) {

  }
})
```

# sftp.write(object)

将二进制文件写入到服务器：

```js
sftp.write({
  file,
  path: "/home/user/notes.md",
  progress: function(sent) {
    // Optional: determine whether is finished here
    return sent > 1024 * 1024
  },
  handler: function(success) {
    
  }
})
```

# sftp.append(object)

将文件追加到远程文件：

```js
sftp.append({
  file,
  path: "/home/user/notes.md",
  handler: function(success) {
    
  }
})
```

# sftp.copy(object)

复制文件：

```js
sftp.copy({
  path: "/home/user/notes.md",
  dest: "/home/user/notes-copy.md",
  progress: function(copied, totalBytes) {
    // Optional: determine whether is finished here
    return sent > 1024 * 1024
  },
  handler: function(success) {
    
  }
})
```

注：以上所有的 handler 也可以通过 async/await 实现。

---

## 文件内容解读与示例

### 用途说明

本文档详细介绍了 `session.sftp` 对象。`sftp` 是 **SSH File Transfer Protocol** 的缩写，它在 SSH 的安全连接基础上，提供了一套丰富的接口用于**操作远程服务器上的文件和目录**。相较于 `channel` 提供的基础 `upload`/`download`，`sftp` 提供了更全面、更强大的文件管理能力，类似于在本地用 `$file` API 操作文件。

### 核心概念：远程文件系统操作

`sftp` 的核心是让你能够像操作本地文件一样去管理远程服务器上的文件。所有操作都是通过已建立的 SSH `session` 进行，因此继承了 SSH 的安全性。

-   **连接管理**: 在执行任何 SFTP 操作之前，必须先显式调用 `sftp.connect()` 来建立 SFTP 连接。这是一个独立于 SSH 会话连接的步骤。
-   **异步操作**: 所有 SFTP 操作都是异步的。你可以使用传统的回调函数 `handler`，或者更现代、更易读的 `async/await` 语法来处理操作结果。

### API 详解 (按功能分类)

**连接管理**
-   `sftp.connect()`: 初始化 SFTP 会话。在使用任何其他 SFTP 方法前必须调用。推荐使用 `await sftp.connect()`。

**目录操作**
-   `sftp.directoryExists({path, handler})`: 检查指定路径的目录是否存在。
-   `sftp.createDirectory({path, handler})`: 在远程服务器上创建新目录。
-   `sftp.removeDirectory({path, handler})`: 删除一个**空**目录。
-   `sftp.contentsOfDirectory({path, handler})`: 列出指定目录下的所有文件和子目录，返回一个包含文件信息对象的数组。

**文件操作**
-   `sftp.fileExists({path, handler})`: 检查指定路径的文件是否存在。
-   `sftp.removeFile({path, handler})`: 删除一个文件。
-   `sftp.moveItem({src, dest, handler})`: 移动或重命名一个文件或目录。
-   `sftp.copy({path, dest, progress, handler})`: 复制远程服务器上的一个文件到另一个位置。
-   `sftp.createSymbolicLink({path, dest, handler})`: 创建一个符号链接（软链接）。

**文件读写**
-   `sftp.contents({path, handler})`: 读取远程文件的**完整内容**并返回一个二进制数据对象（`$data` 类型）。适合读取小文件。
-   `sftp.write({file, path, progress, handler})`: 将本地的二进制数据（`$data` 对象）写入到远程文件。如果文件已存在，会**覆盖**其内容。
-   `sftp.append({file, path, handler})`: 将本地的二进制数据**追加**到远程文件的末尾。

**文件信息**
-   `sftp.infoForFile({path, handler})`: 获取单个文件或目录的详细属性，如大小、修改日期、权限等。返回一个结构化的 `file` 对象。

### 示例：同步本地日志文件到服务器

这个示例将演示如何使用 `async/await` 语法，将本地的一个日志文件上传到服务器的备份目录中，并以日期命名。

```javascript
// (需先通过 $ssh.connect 获得 session 对象)

async function syncLogFile(session) {
  const sftp = session.sftp;

  try {
    // 1. 建立 SFTP 连接
    $ui.loading("正在连接 SFTP...");
    await sftp.connect();
    console.log("SFTP 连接成功");

    // 2. 检查并创建远程备份目录
    const remoteBaseDir = "/var/log/backups";
    const exists = await sftp.directoryExists({ path: remoteBaseDir });
    if (!exists) {
      console.log(`目录 ${remoteBaseDir} 不存在，正在创建...`);
      await sftp.createDirectory({ path: remoteBaseDir });
    }

    // 3. 准备要上传的文件和远程路径
    const logContent = "This is a log entry at " + new Date();
    const logData = $data({ string: logContent });
    const remoteFileName = `log-${new Date().toISOString().split('T')[0]}.txt`;
    const remotePath = `${remoteBaseDir}/${remoteFileName}`;

    // 4. 将日志内容追加到远程文件
    $ui.loading(`正在上传日志到 ${remotePath}`)
    await sftp.append({ file: logData, path: remotePath });

    // 5. 获取上传后的文件信息进行验证
    const fileInfo = await sftp.infoForFile({ path: remotePath });
    $ui.success(`日志追加成功！\n文件大小: ${fileInfo.fileSize} 字节`);

  } catch (error) {
    $ui.error("SFTP 操作失败");
    console.error(error);
  } finally {
    // 6. 关闭连接
    sftp.disconnect();
    $ssh.disconnect();
  }
}

// 假设 session 已经获取
// syncLogFile(session);
```

### 总结

`session.sftp` 是 JSBox 中进行远程文件管理的瑞士军刀。它提供了一套完整、强大且易于使用的 API，无论是简单的文件同步，还是复杂的远程文件系统操作，`sftp` 都能胜任。结合 `async/await` 语法，可以写出逻辑清晰、易于维护的远程文件管理脚本。
