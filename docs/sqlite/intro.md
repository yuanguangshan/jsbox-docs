# SQLite

除了使用文件和缓存系统以外，在 JSBox v1.28 之后你还可以使用 SQLite 作为数据持久化方案，SQLite 是一个轻量级的数据库系统，适合在移动平台使用。请参考 SQLite 官方文档以获得更多信息：https://www.sqlite.org/。

# FMDB

[FMDB](https://github.com/ccgus/fmdb) 是 SQLite 的一个 Objective-C 封装，JSBox 使用了这个库，并提供了更上一层的封装让你可以通过 JavaScript 调用。也即，在必要的情况下你可以通过 Runtime 直接使用 FMDB。当然在大部分的时候，JSBox 提供的接口已经够用。

# SQLite 浏览器

为了更好地支持 SQLite，与接口同时推出的还有一个轻量、简便的 SQLite 浏览器。你可以可视化的查看 SQLite 文件的内容，目前还是只读版本，敬请期待后续的更新。

---

## 文件内容解读与示例

### 用途说明

本文档介绍了 JSBox 对 **SQLite 数据库**的支持。对于需要结构化、持久化存储大量数据的脚本来说，SQLite 是一个比简单的文件读写或使用 `$cache` 更强大、更高效的解决方案。它适用于需要管理复杂数据关系、进行快速查询和事务操作的场景。

### 核心概念

1.  **SQLite**: 
    -   **是什么**: SQLite 是一个自包含的、无服务器的、零配置的、事务性的 SQL 数据库引擎。它非常轻量，是移动端和嵌入式设备上最流行的数据库之一。
    -   **优点**: 与纯文本文件相比，它能更好地组织数据（通过表、行和列），支持强大的查询语言（SQL），并能高效地处理大量数据。

2.  **FMDB**: 
    -   **是什么**: FMDB 是一个流行的 Objective-C 库，它封装了原生 C 语言的 SQLite API，使其在 iOS/macOS 开发中更易于使用。
    -   **与 JSBox 的关系**: JSBox 的 SQLite 功能**基于 FMDB 构建**。JSBox 进一步将 FMDB 的接口封装成简单易用的 JavaScript API。这意味着开发者可以直接享受 FMDB 带来的稳定性和便利性，而无需处理复杂的 Objective-C Runtime 调用。不过，对于高级用户，如果 JSBox 的 API 不足以满足需求，仍然可以通过 Runtime 直接调用底层的 FMDB 方法。

3.  **SQLite 浏览器**: 
    -   **功能**: JSBox 内置了一个 SQLite 数据库的可视化查看工具。开发者可以通过这个工具直接打开并浏览 `.db` 文件的内容，包括查看表结构和数据。
    -   **用途**: 这在开发和调试阶段非常有用，可以让你直观地检查数据是否已正确写入、表结构是否符合预期等。文档提到初期版本是只读的，但它依然是验证数据库操作结果的利器。

### 示例：创建一个简单的笔记数据库

假设你想创建一个脚本来管理本地笔记，使用 SQLite 会是一个很好的选择。

**1. 初始化数据库并创建表 (`database.js`)**

```javascript
// 引入 SQLite 模块
const sqlite = $sqlite;

// 打开（或创建）一个名为 'notes.db' 的数据库文件
const db = sqlite.open("notes.db");

// 创建一个 notes 表，如果它还不存在的话
// 表包含 id, content 和 created_at 三个字段
db.update(`
  CREATE TABLE IF NOT EXISTS notes (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  )
`);

// 关闭数据库连接
db.close();

$ui.toast("数据库初始化成功！");
```

**2. 使用 SQLite 浏览器查看**

-   运行上述脚本后，在 JSBox 的文件系统中会出现一个 `notes.db` 文件。
-   在 JSBox 中找到这个文件并点击它，选择“用 SQLite 浏览器打开”。
-   你应该能看到一个名为 `notes` 的表，以及它的三个列：`id`, `content`, `created_at`。

### 总结

通过集成 SQLite，JSBox 为开发者提供了一个专业级的数据持久化方案。它将移动端最流行、最稳定的数据库技术与简洁的 JavaScript API 相结合，并辅以方便的调试工具（SQLite 浏览器），使得在 JSBox 中开发数据驱动的复杂应用成为可能。