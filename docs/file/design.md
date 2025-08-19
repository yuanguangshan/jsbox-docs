# 设计思路

在 JSBox 里面我们提供了访问文件系统的接口，目的是为了让扩展程序可以把文件存储到磁盘上，例如下载下来的数据。

众所周知 iOS 有沙盒机制，简单说每个 app 可以用的文件系统是被彼此独立开来的，互不干扰，同时 app 能访问的目录是很有限的。

JSBox 里面的文件读写则像是一个 `沙盒中的沙盒`，每个扩展程序的目录都是独立的，也会随着扩展程序的删除或者重命名被清理掉，这样能保证各个扩展程序之间互不干扰，同时也能减少垃圾文件的产生，避免干扰到 JSBox 文件目录下的其他数据。

所以基本原则就是，并不是任何路径都能进行文件读写，当然 JSBox 的接口屏蔽了这些细节，扩展的编写者无须关心具体路径是什么（也无从知晓）。

以上是一点小小的说明。

# 共享目录

为了多个扩展之间共享一些文件，除了每个扩展自己的目录，JSBox 也提供了一个共享的目录，路径以 `shared://` 开头即可。

请注意，共享目录的文件可以被所有的扩展任意读写。

# iCloud Drive

另外，文件系统支持读写 iCloud Drive 内的文件，路径以 `drive://` 开头即可。

请注意，当用户未开启 iCloud 或未允许 JSBox 使用 iCloud Drive 时，iCloud 相关操作将会失败。

# Inbox

文件系统可以读取通过 AirDrop 导入到 JSBox 应用内的文件，路径以 `inbox://` 开头即可。

请注意，Inbox 内的文件可以被所有的扩展任意读写。

# 绝对路径

当需要处理绝对路径的时候，可以通过 `absolute://` 开头来表示当前传入的路径是一个绝对路径。

例如 `$file.copy(...)` 接口，当目标路径为绝对路径时，需要通过上述协议来为该接口提供信息。

---

## 文件内容解读与示例

### 用途说明：JSBox 文件系统的设计哲学

本文档阐述了 JSBox 脚本如何访问文件系统，以及其背后的**设计理念**。理解这些原则对于编写安全、高效且易于维护的 JSBox 脚本至关重要。核心思想是在 iOS 严格的沙盒机制下，为脚本提供既隔离又灵活的文件存储能力。

### 核心概念：“沙盒中的沙盒”

1.  **iOS 沙盒机制**: iOS 系统为每个 App 都分配了一个独立的“沙盒”，App 只能在其沙盒内部读写文件，无法直接访问其他 App 的数据，这保证了系统的安全性和稳定性。

2.  **JSBox 的“沙盒中的沙盒”**: JSBox 在其自身的沙盒内部，为**每个脚本**又创建了一个独立的、隔离的存储空间。这意味着：
    -   **脚本隔离**: 你的脚本默认只能访问自己目录下的文件，不会干扰其他脚本的数据。
    -   **自动清理**: 当你删除或重命名一个脚本时，其对应的文件目录也会被自动清理或重命名，有效避免了垃圾文件的产生。
    -   **简化路径**: 脚本开发者通常无需关心复杂的绝对路径，只需使用相对路径即可操作自己沙盒内的文件。

### 特殊协议：访问特定文件位置

除了每个脚本独立的沙盒目录外，JSBox 还提供了几种特殊的协议（URL Scheme），用于访问共享或系统级别的文件位置：

1.  **`shared://` (共享目录)**:
    -   **用途**: 允许所有 JSBox 脚本读写同一个共享目录下的文件。这是脚本之间共享数据的主要方式。
    -   **注意**: 由于所有脚本都可以访问，请谨慎处理敏感数据，并注意文件读写冲突。

2.  **`drive://` (iCloud Drive)**:
    -   **用途**: 允许脚本直接读写用户 iCloud Drive 中的文件。这对于实现云同步、跨设备数据共享或访问 iCloud Drive 中的文档非常有用。
    -   **注意**: 需要用户在系统设置中开启 iCloud Drive，并允许 JSBox 访问。如果用户未授权或 iCloud 未开启，相关操作会失败。

3.  **`inbox://` (收件箱)**:
    -   **用途**: 访问通过 iOS 的“分享到...”功能（如 AirDrop、其他 App 的“用...打开”）导入到 JSBox 应用内的文件。这些文件会临时存放在 `inbox://` 目录下。
    -   **注意**: `inbox://` 目录下的文件同样可以被所有 JSBox 脚本访问。

4.  **`absolute://` (绝对路径)**:
    -   **用途**: 在极少数情况下，当你需要明确指定一个 JSBox 内部的**绝对路径**时使用。例如，在 `$file.copy()` 或 `$file.move()` 等操作中，如果目标路径是 JSBox 内部的某个固定绝对位置，就需要用 `absolute://` 来明确指示。

### 示例代码：文件读写与协议使用

```javascript
// 1. 在当前脚本的沙盒内读写文件
const myScriptFilePath = "my_data.txt";
$file.write({ path: myScriptFilePath, data: $data({ string: "这是我的脚本私有数据。" }) });
console.log("脚本私有文件内容:", $file.read(myScriptFilePath).string);

// 2. 在共享目录读写文件
const sharedFilePath = "shared://global_config.json";
$file.write({ path: sharedFilePath, data: $data({ string: JSON.stringify({ theme: "dark" }) }) });
console.log("共享文件内容:", $file.read(sharedFilePath).string);

// 3. 尝试访问 iCloud Drive (需要用户授权)
// const iCloudDocPath = "drive://Documents/MyCloudDoc.txt";
// if ($file.exists(iCloudDocPath)) {
//   console.log("iCloud 文件内容:", $file.read(iCloudDocPath).string);
// } else {
//   console.log("iCloud 文件不存在或未授权。");
// }

// 4. 模拟从 Inbox 读取文件 (通常由 Action Extension 触发)
// 假设有一个文件通过 AirDrop 导入到 JSBox，名为 "incoming.txt"
// const inboxFilePath = "inbox://incoming.txt";
// if ($file.exists(inboxFilePath)) {
//   console.log("Inbox 文件内容:", $file.read(inboxFilePath).string);
// } else {
//   console.log("Inbox 中没有文件。");
// }
```

**代码解读**：

这个示例展示了如何使用不同的协议来访问 JSBox 文件系统中的不同区域。理解这些协议是编写能够有效管理数据、与其他脚本或系统服务交互的 JSBox 脚本的关键。 
