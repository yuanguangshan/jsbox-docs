# type: "label"

`label` 用于显示一段不可编辑的文字，是最常用的控件之一：

```js
{
  type: "label",
  props: {
    text: "Hello, World!",
    align: $align.center
  },
  layout: function(make, view) {
    make.center.equalTo(view.super)
  }
}
```

在画布中显示一个 `Hello, World!`。

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
text | string | 读写 | 文字内容
styledText | object | 读写 | 带格式的文本，[参考](component/text.md?id=styledtext)
font | $font | 读写 | 字体
textColor | $color | 读写 | 文字颜色
shadowColor | $color | 读写 | 阴影颜色
align | $align | 读写 | 对齐方式
lines | number | 读写 | 显示行数（0 表示任意多行）
autoFontSize | boolean | 读写 | 是否动态调整字体大小

---

## 文件内容解读与示例

### 组件用途

`label`（标签）是 UI 中最基础的构件，它的唯一作用就是**显示一段不可编辑的静态文本**。从大标题、段落内容到简单的说明文字，几乎所有界面上的文本信息都是通过 `label` 来呈现的。

### 核心属性详解

掌握 `label` 的关键在于熟悉以下几个核心属性，它们共同决定了文本的样式和布局。

- **`text`**: 最核心的属性，用于设置要显示的字符串内容。

- **`font`**: 设置文本的字体和大小。它需要一个 `$font` 对象，例如 `$font(20)` 表示一个大小为 20 的常规字体，`$font("bold", 24)` 表示一个大小为 24 的粗体。

- **`textColor`**: 设置文本的颜色，需要一个 `$color` 对象，例如 `$color("red")` 或 `$color("#FF5733")`。

- **`align`**: 控制文本在其矩形框内的**水平对齐方式**。它接受 `$align` 枚举中的值：
  - `$align.left`: 左对齐（默认）
  - `$align.center`: 居中对齐
  - `$align.right`: 右对齐
  - `$align.justified`: 两端对齐
  - `$align.natural`: 自然对齐（根据语言习惯，例如英语是左对齐）

- **`lines`**: 控制文本可以显示的最大行数。这是一个非常重要的属性：
  - `lines: 1` (默认): 文本只显示一行，如果内容超出，末尾会显示省略号（...）。
  - `lines: 0`: **表示不限制行数**。`label` 的高度会自动扩展，以显示全部文本内容。这是创建多行段落的关键。

- **`styledText`**: 一个高级属性，用于显示“富文本”或“属性化文本”，即在同一段文本中混合使用不同的样式（如不同颜色、字体）。

### 示例代码：构建一个简单的图文卡片

下面的示例将演示如何使用多个 `label` 来构建一个包含标题、副标题和正文内容的文本布局。

```javascript
const articleContent = "JSBox 是一个强大的工具，它提供了一个完整的 JavaScript IDE，让你可以学习编程，并利用它来解决你日常生活中的问题。你可以用它来编写自己的 iOS 小组件、查询公交、聚合新闻等等。";

$ui.render({
  props: {
    title: "Label 组件示例"
  },
  views: [
    {
      type: "view",
      props: {
        bgcolor: $color("#F5F5F5"),
        radius: 10
      },
      layout: function(make, view) {
        make.center.equalTo(view.super);
        make.size.equalTo($size(300, 250));
      },
      views: [
        // 1. 主标题
        {
          type: "label",
          props: {
            text: "关于 JSBox",
            font: $font("bold", 22),
            align: $align.center
          },
          layout: function(make, view) {
            make.top.inset(15);
            make.left.right.inset(10);
          }
        },
        // 2. 副标题
        {
          type: "label",
          props: {
            text: "一个创造工具",
            font: $font(14),
            textColor: $color("gray"),
            align: $align.center
          },
          layout: function(make, view) {
            make.top.equalTo(view.prev.bottom).offset(5);
            make.left.right.inset(10);
          }
        },
        // 3. 正文段落
        {
          type: "label",
          props: {
            text: articleContent,
            lines: 0, // 关键：设置为0，允许显示多行
            font: $font(16),
            align: $align.justified // 两端对齐
          },
          layout: function(make, view) {
            make.top.equalTo(view.prev.bottom).offset(15);
            make.left.right.inset(15);
          }
        }
      ]
    }
  ]
});
```

**代码解读**：

1.  **主标题**：我们使用了较大的粗体字（`$font("bold", 22)`）和居中对齐（`$align.center`）来使其突出。
2.  **副标题**：使用了较小的字号和灰色字体，作为补充说明。
3.  **正文段落**：最关键的设置是 `lines: 0`，它告诉 `label` 根据内容自动换行并调整自身高度。我们还使用了 `$align.justified` 来让段落的左右边缘看起来更整齐。

通过组合使用这些基础属性，你可以用 `label` 构建出任何你需要的文本布局，它是所有复杂 UI 的信息基础。 
