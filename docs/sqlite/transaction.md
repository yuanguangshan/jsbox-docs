# db.beginTransaction()

对于 SQLite 实例，可以使用 `beginTransaction()` 来开始一个事务。

# db.commit()

对于 SQLite 实例，可以使用 `commit()` 来提交一个事务。

# db.rollback()

对于 SQLite 实例，可以使用 `rollback()` 来回滚一个事务。

---

## 文件内容解读与示例

### 用途说明

本文档介绍了在 JSBox 中如何手动管理 **SQLite 事务（Transactions）**。事务是一系列数据库操作的集合，这些操作被视为一个单一的逻辑工作单元。事务确保了数据的**原子性**，即这组操作要么全部成功执行，要么在出现错误时全部不执行（回滚），从而保证数据库的完整性和一致性。

### 核心概念：ACID 属性

虽然文档没有明说，但事务的核心是保证数据库操作的 ACID 属性，其中**原子性（Atomicity）**是最直接相关的：

-   **原子性（Atomicity）**: 事务内的所有操作是一个不可分割的单元。例如，在一个银行转账操作中，“从A账户扣款”和“向B账户存款”这两个更新必须同时成功。如果扣款后、存款前程序崩溃，事务可以确保A账户的扣款操作被撤销，就像什么都没发生过一样。

### API 详解

JSBox 提供了三个基本方法来手动控制事务的边界：

1.  **`db.beginTransaction()`**
    -   **作用**: 标记一个事务的开始。在此调用之后执行的所有数据库更新（`update`、`insert`、`delete`）都将成为该事务的一部分。
    -   **性能**: 将多个写操作包裹在一个事务中，通常比单独执行它们要快得多，因为数据库只需在最后 `commit` 时将所有更改一次性写入磁盘。

2.  **`db.commit()`**
    -   **作用**: 提交事务。如果从 `beginTransaction()` 到 `commit()` 之间的所有操作都成功，此方法会将所有更改永久性地保存到数据库中。

3.  **`db.rollback()`**
    -   **作用**: 回滚事务。如果在事务处理过程中发生任何错误，调用此方法会撤销从 `beginTransaction()` 开始的所有更改，使数据库恢复到事务开始之前的状态。

**重要提示**: 这些手动事务控制方法通常用于单线程操作。在多线程环境下，强烈推荐使用上一节文档中提到的 [`queue.transaction()`](sqlite/queue.md?id=transaction)，因为它能以更简洁和安全的方式自动管理事务和线程问题。

### 示例：安全的资金转账

假设我们有一个账户表 `accounts`，需要实现一个从一个账户向另一个账户转账的功能。这个操作必须是事务性的。

```javascript
const db = $sqlite.open("bank.db");

// 准备工作：创建表并插入初始数据
db.update("CREATE TABLE IF NOT EXISTS accounts (id TEXT, balance REAL)");
db.update({ sql: "INSERT OR IGNORE INTO accounts VALUES (?, ?)", args: ["A", 100] });
db.update({ sql: "INSERT OR IGNORE INTO accounts VALUES (?, ?)", args: ["B", 100] });

function transfer(fromId, toId, amount) {
  // 1. 开始事务
  db.beginTransaction();

  try {
    // 2. 从 A 账户扣款
    const fromUpdate = db.update({
      sql: "UPDATE accounts SET balance = balance - ? WHERE id = ? AND balance >= ?",
      args: [amount, fromId, amount]
    });

    // 检查扣款是否成功（例如，余额不足）
    if (fromUpdate.result === false || fromUpdate.changes === 0) {
      throw new Error("余额不足或付款方账户不存在");
    }

    // 3. 向 B 账户存款
    const toUpdate = db.update({
      sql: "UPDATE accounts SET balance = balance + ? WHERE id = ?",
      args: [amount, toId]
    });

    if (toUpdate.result === false || toUpdate.changes === 0) {
      throw new Error("收款方账户不存在");
    }

    // 4. 如果所有操作都成功，提交事务
    db.commit();
    console.log(`转账 ${amount} 成功！`);

  } catch (error) {
    // 5. 如果任何步骤出错，回滚事务
    db.rollback();
    console.error(`转账失败: ${error.message}`);
  }
}

// 执行转账
transfer("A", "B", 20);

// 查询结果
db.query("SELECT * FROM accounts", (rs, err) => {
  while(rs.next()) {
    console.log(rs.values);
  }
  rs.close();
});

db.close();
```

### 总结

手动事务控制是实现复杂、可靠数据库操作的基础。通过 `beginTransaction`, `commit`, 和 `rollback`，开发者可以确保一系列操作的原子性，维护数据的完整性。虽然在多线程场景下应优先使用 `dbQueue`，但在单线程或需要精细控制事务流程的场景下，这三个API非常有用。