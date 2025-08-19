# $sqlite.open(path)

使用文件路径来创建一个 SQLite 实例：

```js
const db = $sqlite.open("test.db");
```

# $sqlite.close(db)

关闭一个 SQLite 实例：

```js
const db = $sqlite.open("test.db");

//...
$sqlite.close(db); // Or db.close();
```

# 更新操作

SQLite 实例支持通过 update 进行更新操作：

```js
db.update("CREATE TABLE User(name text, age integer)");
// Return: { result: true, error: error }
```

也支持占位符和参数：

```js
db.update({
  sql: "INSERT INTO User values(?, ?)",
  args: ["Cyan", 28]
});
```

注：请勿通过参数拼接来进行数据库的更新操作，在任何时候都要使用占位符和参数列表。

# 查询操作

SQLite 实例支持通过 query 进行查询操作：

```js
db.query("SELECT * FROM User", (rs, err) => {
  while (rs.next()) {
    const values = rs.values;
    const name = rs.get("name"); // Or rs.get(0);
  }
  rs.close();
});
```

Result set 还支持如下操作：

```js
const columnCount = rs.columnCount; // Column count
const columnName = rs.nameForIndex(0); // Column name
const columnIndex = rs.indexForName("age"); // Column index
const query = rs.query; // SQL Query
```

请参考 [Result Set 文档](object/result-set.md)了解更多内容。

同样的，当查询有占位符和参数时，使用方法和 update 类似：

```js
db.query({
  sql: "SELECT * FROM User where age = ?",
  args: [28]
}, (rs, err) => {

});
```

---

## 文件内容解读与示例

### 用途说明

本文档是 JSBox 中 **SQLite API 的核心使用指南**，涵盖了数据库的创建、连接、关闭、数据更新（增、删、改）和查询（查）等基本操作。这是使用 SQLite 功能时最常用、最基础的一部分。

### API 详解

1.  **`$sqlite.open(path)`**
    -   **作用**: 打开或创建一个数据库文件。`path` 是一个字符串，表示数据库文件的名称或相对路径。如果文件不存在，SQLite 会自动创建它。
    -   **返回值**: 返回一个数据库连接实例（`db` 对象），后续的所有操作都将通过这个对象进行。

2.  **`$sqlite.close(db)` 或 `db.close()`**
    -   **作用**: 关闭一个已打开的数据库连接，释放相关资源。这是一个好习惯，应该在完成所有数据库操作后调用。

3.  **`db.update(...)`**
    -   **作用**: 执行任何**不返回数据**的 SQL 语句，主要包括 `CREATE TABLE`, `INSERT`, `UPDATE`, `DELETE` 等。
    -   **参数**: 
        -   可以直接传入一个 SQL 字符串，如 `db.update("CREATE TABLE ...")`。
        -   **（推荐）**可以传入一个包含 `sql` 和 `args` 的对象，用于**参数化查询**。`sql` 是带 `?` 占位符的 SQL 语句，`args` 是一个数组，其元素会按顺序替换 `?`。
    -   **安全警告**: 文档中明确指出，**“请勿通过参数拼接来进行数据库的更新操作”**。这是为了防止 **SQL 注入**攻击。始终使用 `?` 占位符和 `args` 数组来传递用户输入或变量，这是数据库编程的最佳安全实践。

4.  **`db.query(...)`**
    -   **作用**: 执行一个**返回数据**的 SQL 语句，主要是 `SELECT`。
    -   **参数**: 
        -   和 `update` 一样，支持直接传入 SQL 字符串或带参数的对象。
        -   第二个参数是一个回调函数 `(rs, err) => { ... }`，查询结果将通过这个函数返回。
    -   **结果处理 (ResultSet - `rs`)**: 
        -   回调函数中的 `rs` 对象是查询结果集。你需要使用一个循环 `while (rs.next())` 来遍历每一行数据。
        -   `rs.next()`: 将光标移动到下一行，如果有下一行则返回 `true`。
        -   `rs.values`: 以数组形式返回当前行的所有值。
        -   `rs.get(columnName)` 或 `rs.get(columnIndex)`: 获取当前行指定列的值，可以通过列名（字符串）或列索引（数字）获取。
        -   `rs.close()`: **非常重要**。在处理完结果集后，必须调用 `rs.close()` 来释放资源。

### 示例：一个完整的用户管理流程

```javascript
// 1. 打开数据库
const db = $sqlite.open("users.db");

// 2. 创建表（如果不存在）
db.update("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT, email TEXT)");

// 3. 插入新用户 (使用参数化查询)
const newUser = { id: 1, name: "Alice", email: "alice@example.com" };
db.update({
  sql: "INSERT INTO users (id, name, email) VALUES (?, ?, ?)",
  args: [newUser.id, newUser.name, newUser.email]
});

// 插入另一个用户
db.update({
  sql: "INSERT INTO users (id, name, email) VALUES (?, ?, ?)",
  args: [2, "Bob", "bob@example.com"]
});

// 4. 更新用户信息
db.update({
  sql: "UPDATE users SET email = ? WHERE name = ?",
  args: ["new_email_for_bob@example.com", "Bob"]
});

// 5. 查询所有用户并打印
console.log("所有用户:");
db.query("SELECT * FROM users", (rs, err) => {
  if (err) {
    console.error("查询失败: ", err);
    return;
  }
  while (rs.next()) {
    // 使用 get(columnName) 获取数据
    const user = {
      id: rs.get("id"),
      name: rs.get("name"),
      email: rs.get("email")
    };
    console.log(user);
  }
  rs.close(); // 必须关闭结果集
});

// 6. 按条件查询
console.log("\n查询ID为1的用户:");
db.query({ sql: "SELECT * FROM users WHERE id = ?", args: [1] }, (rs, err) => {
  while (rs.next()) {
    console.log(rs.values); // 使用 .values 获取数据数组
  }
  rs.close();
});

// 7. 关闭数据库
db.close();
```

### 总结

该文档提供了在 JSBox 中使用 SQLite 所需的最基本、最核心的操作方法。掌握 `open`, `close`, `update`, 和 `query` 的用法，特别是**如何使用参数化查询来保证安全**，以及**如何正确处理查询结果集**，是进行任何数据库开发的基础。
