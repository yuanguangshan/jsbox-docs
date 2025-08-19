# indexPath

在 `list` 或 `matrix` 里面，表示一个元素所在的位置。

属性 | 类型 | 读写 | 说明
---|---|---|---
section | number | 读写 | 区号
row | number | 读写 | 行号
item | number | 读写 | 等于 `row`，通常在 matrix 里面使用

---

## 文件内容解读与示例

### 用途说明

`indexPath` 对象是 JSBox 中一个非常重要的概念，它用于**精确标识 `list`（列表）和 `matrix`（网格）视图中某个项目（或单元格）的位置**。在这些分层的数据结构中，一个项目的位置通常由它所在的“区”（Section）和该区内的“行”（Row）共同确定。`indexPath` 就是这种分层索引的标准化表示。

### 核心概念：分层索引

在 iOS 的 UI 开发中，列表和网格通常被组织成一个或多个“区”（Section），每个区又包含多行（Row）或多个项目（Item）。`indexPath` 对象就是用来描述“第几区”的“第几行/项目”的。

-   **`$indexPath(section, row)`**: 这是一个构造函数，用于手动创建一个 `indexPath` 对象。例如，`$indexPath(0, 2)` 表示第 0 区的第 2 行（索引从 0 开始）。

### 属性详解

`indexPath` 对象包含以下只读属性：

-   **`indexPath.section`**: 表示项目所在的**区（Section）**的索引。索引从 `0` 开始，即第一个区是 `0`，第二个区是 `1`，以此类推。
-   **`indexPath.row`**: 表示项目在当前区内的**行（Row）**的索引。索引从 `0` 开始，即区内的第一行是 `0`，第二行是 `1`，以此类推。
-   **`indexPath.item`**: 这个属性与 `indexPath.row` 相同。在 `matrix`（网格）视图中，通常用 `item` 来指代项目，但其含义与 `row` 完全一致。

### 示例代码：在列表中获取点击项的位置

下面的示例将创建一个包含多个区的列表。当用户点击列表中的任何一项时，脚本会弹出一个提示，显示被点击项所在的区号和行号。

```javascript
// 准备列表数据，包含多个区
const listData = [
  {
    title: "水果",
    rows: ["苹果", "香蕉", "橙子"]
  },
  {
    title: "蔬菜",
    rows: ["胡萝卜", "土豆", "西红柿"]
  },
  {
    title: "饮品",
    rows: ["咖啡", "茶", "牛奶"]
  }
];

$ui.render({
  props: { title: "IndexPath 示例" },
  views: [
    {
      type: "list",
      props: {
        data: listData
      },
      layout: $layout.fill,
      events: {
        // didSelect 事件的回调函数会接收 indexPath 参数
        didSelect: (sender, indexPath, data) => {
          // indexPath.section: 获取区号
          // indexPath.row: 获取行号
          const sectionTitle = listData[indexPath.section].title;
          const rowContent = listData[indexPath.section].rows[indexPath.row];

          $ui.alert({
            title: "你点击了",
            message: `区: ${sectionTitle} (索引: ${indexPath.section})\n行: ${rowContent} (索引: ${indexPath.row})`
          });
        }
      }
    }
  ]
});
```

**代码解读**：

1.  我们创建了一个 `list` 视图，其 `data` 属性被设置为一个包含“水果”、“蔬菜”、“饮品”三个区的数组。
2.  在 `list` 的 `didSelect` 事件回调函数中，我们接收到了 `indexPath` 参数。
3.  通过 `indexPath.section` 和 `indexPath.row`，我们能够准确地知道用户点击了哪个区中的哪一行，并据此从原始数据 `listData` 中获取到对应的文本内容。

`indexPath` 对象是与 `list` 和 `matrix` 组件进行交互时不可或缺的工具。它提供了一种清晰、标准化的方式来定位和操作这些复杂数据结构中的特定项目。 
