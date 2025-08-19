# ResultSet

`ResultSet` 是 SQLite 返回的查询结果：

```js
db.query("SELECT * FROM USER", (rs, err) => {
  while (rs.next()) {

  }
  rs.close();
});
```

属性 | 类型 | 读写 | 说明
---|---|---|---
query | string | 只读 | SQL Query
columnCount | number | 只读 | 列数
values | object | 只读 | 所有值

# next()

移动到下一个结果。

# close()

关闭查询结果。

# indexForName(string)

根据列的名字获得列的序号。

# nameForIndex(number)

根据列的序号获得列的名字。

# get(object)

根据列的序号或名字获得值。

# isNull(object)

根据列的序号或名字判断是否为空。

---

## 文件内容解读与示例

### 用途说明

`ResultSet` 对象是 JSBox 中用于表示 **SQLite 数据库查询结果**的标准数据类型。当你使用 `$sqlite` API 执行 `SELECT` 查询时，它会返回一个 `ResultSet` 对象。这个对象允许你逐行访问查询到的数据，并提供了多种方法来获取列的值和元信息。

### 核心概念：游标式遍历

`ResultSet` 采用的是“游标式”遍历机制。这意味着你不能一次性获取所有查询结果，而是需要通过 `next()` 方法逐行移动游标，每次移动到一行数据，然后才能获取当前行的数据。这种方式对于处理大量数据时非常高效，因为它不需要一次性将所有数据加载到内存中。

**重要提示**：在处理完查询结果后，**务必调用 `rs.close()` 方法**来关闭结果集并释放相关资源，避免内存泄漏。

### 属性详解

-   **`resultSet.query`**: 只读，返回执行此查询的原始 SQL 语句字符串。
-   **`resultSet.columnCount`**: 只读，返回查询结果的列数。
-   **`resultSet.values`**: 只读，返回一个数组，包含当前游标所在行所有列的值。这个数组的顺序与 SQL 查询中列的顺序一致。

### 方法详解

-   **`resultSet.next()`**: **核心方法**。将游标移动到结果集的下一行。如果成功移动到下一行（即还有数据可读），则返回 `true`；如果已经到达结果集的末尾，则返回 `false`。
-   **`resultSet.close()`**: **重要方法**。关闭结果集并释放所有相关资源。在完成对 `ResultSet` 的操作后，必须调用此方法。
-   **`resultSet.get(columnIdentifier)`**: 获取当前游标所在行中指定列的值。`columnIdentifier` 可以是列的索引（从 `0` 开始的数字）或列的名称（字符串）。
-   **`resultSet.isNull(columnIdentifier)`**: 判断当前游标所在行中指定列的值是否为 `NULL`。
-   **`resultSet.indexForName(columnName)`**: 根据列的名称（字符串）获取该列的索引。
-   **`resultSet.nameForIndex(columnIndex)`**: 根据列的索引（数字）获取该列的名称。

### 示例代码：查询并显示用户数据

下面的示例将创建一个简单的 SQLite 数据库，插入一些用户数据，然后查询这些数据，并逐行显示查询结果。

```javascript
// 1. 打开或创建数据库
const db = $sqlite.open("test.db");

// 2. 创建表 (如果不存在)
db.update("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT, age INTEGER)");

// 3. 插入一些数据 (如果表为空)
const count = db.query("SELECT COUNT(*) FROM users").result;
if (count === 0) {
  db.update("INSERT INTO users (name, age) VALUES (?, ?)", "张三", 25);
  db.update("INSERT INTO users (name, age) VALUES (?, ?)", "李四", 30);
  db.update("INSERT INTO users (name, age) VALUES (?, ?)", "王五", 22);
  $ui.toast("数据已初始化。");
}

// 4. 执行查询
db.query("SELECT * FROM users", (rs, err) => {
  if (err) {
    $ui.alert(`查询失败: ${err.localizedDescription}`);
    return;
  }

  let resultText = "查询结果:\n";
  resultText += "--------------------\n";

  // 5. 遍历 ResultSet
  while (rs.next()) {
    const id = rs.get("id"); // 通过列名获取值
    const name = rs.get(1); // 通过列索引获取值 (name 是第二列，索引为1)
    const age = rs.get("age");

    resultText += `ID: ${id}, 姓名: ${name}, 年龄: ${age}\n`;
  }

  // 6. 关闭 ResultSet，释放资源
  rs.close();

  resultText += "--------------------\n";
  resultText += `总共 ${rs.columnCount} 列。`;

  $ui.alert({
    title: "数据库查询",
    message: resultText
  });
});

// 7. 关闭数据库连接 (在所有操作完成后)
db.close();
```

**代码解读**：

1.  我们首先打开或创建了一个名为 `test.db` 的数据库，并创建了 `users` 表。
2.  `db.query("SELECT * FROM users", (rs, err) => { ... })` 执行查询，并在回调中接收 `ResultSet` 对象 `rs`。
3.  **`while (rs.next())` 循环是遍历 `ResultSet` 的标准方式**。每次调用 `rs.next()`，游标就会移动到下一行。如果还有数据，循环继续；否则，循环结束。
4.  在循环内部，我们使用 `rs.get("columnName")` 或 `rs.get(columnIndex)` 来获取当前行中指定列的值。
5.  **`rs.close()` 是至关重要的一步**。在 `while` 循环结束后，必须调用 `rs.close()` 来关闭结果集并释放底层资源。

`ResultSet` 对象是 JSBox 中与 SQLite 数据库进行交互的核心。理解其游标式遍历和资源管理（`close()`）对于编写高效、稳定的数据库操作脚本至关重要。 
