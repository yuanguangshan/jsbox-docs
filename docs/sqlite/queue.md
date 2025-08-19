# 多线程下的 SQLite 实例

不要用多个线程同时访问一个 SQLite 实例，如果有这个必要，你可以通过 Queue 来实现：

```js
const queue = $sqlite.dbQueue("test.db");

// Operations
queue.operations(db => {
  db.update();
  db.query();
  //...
});

// Transaction
queue.transaction(db => {
  db.update();
  db.query();
  //...
  const rollback = errorOccured;
  return rollback;
});

queue.close();
```

---

## 文件内容解读与示例

### 用途说明

本文档介绍了在 JSBox 中如何**安全地在多线程环境下使用 SQLite**。直接从不同的线程并发访问同一个 SQLite 数据库连接实例会导致数据损坏、应用崩溃等严重问题。`$sqlite.dbQueue` 提供了一个关键的解决方案，确保所有数据库操作都以线程安全的方式执行。

### 核心概念：串行调度队列（Serial Dispatch Queue）

-   **问题所在**: SQLite 本身在某些模式下是线程安全的，但 FMDB（JSBox 使用的库）的单个 `db` 实例并非设计为在多线程中共享。如果在主线程查询数据的同时，另一个子线程（例如网络请求的回调）尝试写入数据，就会产生竞争条件。
-   **解决方案**: `$sqlite.dbQueue` 的核心思想是创建一个**串行队列（Queue）**。所有提交给这个队列的数据库操作（无论是读还是写）都会被**按顺序、一个接一个地执行**。这样就避免了多个操作同时访问数据库的冲突，保证了操作的原子性和数据的完整性。
-   **工作方式**: 当你调用 `queue.operations(...)` 或 `queue.transaction(...)` 时，你提供一个函数。这个函数会被放入队列中，并在轮到它的时候在一个安全的、独立的线程上执行。函数接收一个 `db` 对象作为参数，这个 `db` 对象是该队列管理的、安全的数据库连接实例。

### API 详解

1.  **`const queue = $sqlite.dbQueue(path)`**
    -   创建一个与指定数据库文件（`path`）关联的数据库队列。如果数据库文件不存在，它会被创建。

2.  **`queue.operations(db => { ... })`**
    -   用于执行一组**非事务性**的数据库操作。
    -   你应该将所有的 `db.update()` 和 `db.query()` 调用放在这个闭包函数内部。
    -   这适用于独立的、不需要作为一个整体成功或失败的读写操作。

3.  **`queue.transaction(db => { ...; return rollback; })`**
    -   用于执行一个**事务（Transaction）**。事务是一组必须**全部成功**或**全部失败**的操作。
    -   **原子性**: 在这个闭包内的所有数据库操作被包裹在一个 `BEGIN TRANSACTION` 和 `COMMIT` 或 `ROLLBACK` 之间。
    -   **回滚机制**: 闭包函数的返回值决定了事务的最终状态。如果返回 `true`，事务将被**回滚（Rollback）**，所有更改被撤销。如果返回 `false`（或无返回值），事务将被**提交（Commit）**。

4.  **`queue.close()`**
    -   关闭数据库队列。在脚本结束或不再需要访问数据库时调用，以释放资源。

### 示例：在网络请求后更新数据库

想象一个场景：你的脚本从一个 API 获取用户数据，然后需要将其存入本地数据库。网络请求是在子线程中完成的，因此必须使用 `dbQueue` 来确保线程安全。

```javascript
// 创建或打开一个数据库队列
const queue = $sqlite.dbQueue("user_data.db");

// 确保用户表存在
queue.operations(db => {
  db.update("CREATE TABLE IF NOT EXISTS users (id TEXT, name TEXT)");
});

// 发起网络请求获取数据
$http.get("https://api.example.com/user/1").then(resp => {
  const user = resp.data;
  if (user) {
    // 在事务中插入或更新用户数据
    // 网络请求的回调在子线程，所以必须用 queue
    queue.transaction(db => {
      try {
        db.update({
          sql: "INSERT OR REPLACE INTO users (id, name) VALUES (?, ?)",
          args: [user.id, user.name]
        });
        console.log("用户数据已更新");
        return false; // 返回 false 提交事务
      } catch (error) {
        console.error("数据库操作失败: ", error);
        return true; // 返回 true 回滚事务
      }
    });
  }
});

// 注意：在实际应用中，需要确保在所有异步操作完成后才 close
// queue.close();
```

### 总结

`$sqlite.dbQueue` 是在 JSBox 中进行健壮的数据库编程的**必备工具**。任何涉及多线程（如网络请求、定时器等）的数据库操作都**必须**通过它来完成，以避免数据混乱和程序崩溃。它是保证数据一致性和应用稳定性的关键。