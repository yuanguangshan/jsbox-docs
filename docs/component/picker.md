# type: "picker"

`picker` 用于创建一个通用的数据选择器：

```js
{
  type: "picker",
  props: {
    items: Array(3).fill(Array.from(Array(256).keys()))
  },
  layout: function(make) {
    make.left.top.right.equalTo(0)
  }
}
```

上述代码会创建一个用于选择 RGB 颜色的三个滚轮，他们之间彼此的滚动是独立的。

除此之外，你也可以创建具有`级联（cascade）`关系的选择器，类似于级联表单，在创建的时候指定 `items` 的方式有所不同：

```js
items: [
  {
    title: "Language",
    items: [
      {
        title: "Web",
        items: [
          {
            title: "JavaScript"
          },
          {
            title: "PHP"
          }
        ]
      },
      {
        title: "Client",
        items: [
          {
            title: "Swift"
          },
          {
            title: "Objective-C"
          }
        ]
      }
    ]
  },
  {
    title: "Framework",
    // ...
  }
]
```

显而易见这是一个递归的结构，当用户选择了某个 `title` 之后，将会更新下一级滚轮的 `items`，要理解这个控件首先要理解级联的含义。

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
data | object | 只读 | 选中的所有项
selectedRows | object | 只读 | 选中的所有序号

# events: changed

`changed` 事件在当前值变化时回调：

```js
changed: function(sender) {
  
}
```

# $picker.date(object)

创建一个日期选择器的快捷方式，参数和上面的方式相同，picker 将会直接从屏幕底部弹出。

# $picker.data(object)

创建一个通用选择器的快捷方式，参数和上面的方式相同，picker 将会直接从屏幕底部弹出。

以上两个方法的参数和之前提到的格式完全相同。

# $picker.color(object)

使用内建的颜色选择器来选择颜色：

```js
const color = await $picker.color({
  // color: aColor
});
```

---

## 文件内容解读与示例

### 组件用途

`picker` 组件提供了一个或多个滚轮（Wheel）式的选择器，用于从预设的数据列表中进行选择。它是 `date-picker` 的通用版本，适用于任何需要让用户从结构化数据中进行选择的场景，例如省市选择、商品规格选择等。

### 两种核心数据模式

`picker` 的行为完全由 `props` 中的 `items` 数组的结构决定。理解这两种结构是使用此组件的关键。

#### 1. 独立滚轮模式

- **数据结构**: `items` 是一个“数组的数组”，例如 `items: [["A", "B"], [1, 2, 3]]`。
- **行为**: 每一个内部数组都对应一个独立的、互不干扰的滚轮。上例会创建两个滚轮，第一个显示 "A", "B"，第二个显示 1, 2, 3。它们各自滚动，没有关联。
- **应用场景**: 选择 RGB 颜色值。三个滚轮都独立地从 0 滚动到 255。

#### 2. 级联滚轮模式 (Cascading)

- **数据结构**: `items` 是一个具有特定 `title` 和 `items` 键的、可递归嵌套的对象数组。
- **行为**: 滚轮之间存在父子关系。当用户在父级滚轮中选择某一项时，子级滚轮的内容会根据父级的选择动态更新。
- **应用场景**: 省市二级联动选择。当第一个滚轮选择了“广东省”，第二个滚轮的内容会自动更新为“广州市”、“深圳市”等；若第一个滚轮切换为“湖南省”，第二个滚轮则会更新为“长沙市”、“岳阳市”等。

### 获取选中结果

无论在哪种模式下，你都通过 `changed` 事件来获取用户的选择。事件的回调函数会收到 `sender` 参数，你可以通过它来获取结果：

- `sender.data`: 一个数组，包含了**当前每个滚轮上被选中的项的值**。例如 `["广东省", "深圳市"]`。
- `sender.selectedRows`: 一个数组，包含了**当前每个滚轮上被选中的项的索引**。例如 `[0, 1]`。

### `picker` 组件 vs. `$picker` 全局方法

- **`{ type: "picker" }`**: 是一个 **UI 组件**，你可以将它自由地放置在你设计的界面中的任何位置。
- **`$picker.data()` / `$picker.date()` / `$picker.color()`**: 是一组**全局辅助函数**。它们会从屏幕底部弹出一个模态选择器，用于快速获取用户的单次选择，而无需你构建完整的 UI。这是一种更轻量级的快捷方式。

### 示例代码：级联选择器

下面的示例将创建一个级联选择器，用于选择编程语言的分类和具体语言。

```javascript
// 1. 定义级联数据结构
const languageData = [
  {
    title: "Web 前端",
    items: [
      { title: "JavaScript" },
      { title: "TypeScript" },
      { title: "CSS" }
    ]
  },
  {
    title: "移动端",
    items: [
      { title: "Swift" },
      { title: "Kotlin" },
      { title: "Objective-C" }
    ]
  },
  {
    title: "后端",
    items: [
      { title: "Python" },
      { title: "Java" },
      { title: "Go" }
    ]
  }
];

$ui.render({
  props: { title: "Picker 组件示例" },
  views: [
    {
      type: "label",
      props: {
        id: "result-label",
        text: "当前选择: Web 前端, JavaScript",
        align: $align.center,
        font: $font("bold", 18)
      },
      layout: make => make.top.left.right.inset(20)
    },
    {
      type: "picker",
      props: {
        items: languageData
      },
      layout: make => {
        make.top.equalTo($("result-label").bottom).offset(10);
        make.left.right.bottom.equalTo(0);
      },
      events: {
        changed: sender => {
          // sender.data 是一个包含选中项 title 的数组
          const selection = sender.data.join(", ");
          $("result-label").text = `当前选择: ${selection}`;
        }
      }
    }
  ]
});
```

**代码解读**：

1.  我们定义了 `languageData`，它是一个典型的级联结构。第一级是“Web 前端”、“移动端”等分类，第二级是具体的语言。
2.  我们将这个数据直接传给 `picker` 的 `items` 属性，`picker` 会自动识别并创建出级联效果的两个滚轮。
3.  在 `changed` 事件中，我们通过 `sender.data` 获取到一个包含两项选择的数组（例如 `["移动端", "Swift"]`），然后用 `join(", ")` 将它们合并成一个字符串，并更新到上方的 `label` 中。 
