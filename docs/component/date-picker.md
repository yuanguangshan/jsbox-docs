# type: "date-picker"

`date-picker` 用于创建一个日期选择器：

```js
{
  type: "date-picker",
  layout: function(make) {
    make.left.top.right.equalTo(0)
  }
}
```

创建一个默认的日期选择器。

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
date | object | 读写 | 当前值
min | object | 读写 | 最小值
max | object | 读写 | 最大值
mode | number | 读写 | [请参考](https://developer.apple.com/documentation/uikit/uidatepickermode)
interval | number | 读写 | 步长（分钟）

# events: changed

`changed` 事件在当前值变化时回调：

```js
changed: function(sender) {
  
}
```

---

## 文件内容解读与示例

### 组件用途

`date-picker` 组件提供了一个标准、易用的可视化界面，专门用于让用户选择日期、时间或一个时间段。它封装了 iOS 原生的 `UIDatePicker`，因此用户体验与系统设置（如“闹钟”、“日历”）中的选择器完全一致，是获取日期时间输入的首选方式。

### 核心属性详解

#### `mode`: 选择器模式

这是 `date-picker` 最重要的属性，它决定了选择器的外观和功能。`mode` 接受一个数字，对应不同的模式：

- **`0` (Time - 时间模式)**: 只显示小时和分钟（可能还有 AM/PM），用于选择一天中的某个时间。
- **`1` (Date - 日期模式)**: 显示年、月、日，用于选择一个具体的日期。
- **`2` (Date and Time - 日期和时间模式)**: 同时显示日期和时间，是功能最全的模式。
- **`3` (Countdown - 倒计时模式)**: 只显示小时和分钟，但用于设置一个持续的时长（例如“2小时15分钟”），而不是一个具体的时间点。

#### `date`, `min`, `max`: 日期控制

这三个属性的值都应该是标准的 JavaScript `Date` 对象。

- **`date`**: 用于设置选择器**初始显示**的日期和时间。同时，你也可以在任何时候读取这个属性来获取当前选中的值。
- **`min` 和 `max`**: 用于限制用户可选择的日期范围。例如，设置 `min: new Date()` 可以防止用户选择过去的时间。这对于预定会议、设置提醒等场景非常有用。

#### `interval`: 分钟步长

此属性仅在与时间相关的模式下（`mode` 为 0, 2, 3）生效。它定义了分钟滚轮的间隔，单位是分钟。例如，`interval: 15` 会让分钟滚轮只显示 0, 15, 30, 45 这些选项。

### 核心事件 `changed`

这是 `date-picker` 的核心交互事件。每当用户滚动滚轮，导致选中的值发生变化时，`changed` 事件就会被触发。

- **`sender`**: 事件回调函数会收到 `sender` 参数，它指向 `date-picker` 视图本身。
- **获取值**: 你可以通过 `sender.date` 来获取用户最新选择的 `Date` 对象。

### 示例代码：会议时间选择器

下面的示例将创建一个日期时间选择器，并把用户选择的结果实时显示在一个标签上。

```javascript
$ui.render({
  props: {
    title: "会议时间选择器"
  },
  views: [
    {
      // 创建一个标签，用于显示结果
      type: "label",
      props: {
        id: "result-label",
        text: "请选择会议开始时间",
        font: $font("bold", 18),
        align: $align.center
      },
      layout: function(make, view) {
        make.top.equalTo(view.super).offset(20);
        make.left.right.inset(10);
      }
    },
    {
      // 创建日期选择器
      type: "date-picker",
      props: {
        // mode 2: 同时选择日期和时间
        mode: 2,
        // 最小可选时间为当前时间，防止选择过去
        min: new Date(),
        // 设置分钟的间隔为5分钟
        interval: 5
      },
      layout: function(make, view) {
        make.top.equalTo($("result-label").bottom).offset(10);
        make.left.right.bottom.equalTo(0);
      },
      events: {
        changed: function(sender) {
          // 获取当前选中的 Date 对象
          const selectedDate = sender.date;
          // 将 Date 对象格式化为本地化的可读字符串
          const dateString = selectedDate.toLocaleString();
          // 将结果更新到标签上
          $("result-label").text = `选定时间：${dateString}`;
        }
      }
    }
  ]
});
```

**代码解读**：

1.  我们设置了 `mode: 2` 来让用户同时选择日期和时间。
2.  通过 `min: new Date()`，我们确保了用户无法选择一个已经过去的时间点。
3.  核心逻辑在 `changed` 事件中：每当用户滚动选择器，我们就从 `sender.date` 获取新的值，将其格式化后，更新到上方的 `result-label` 标签中。这就构成了一个完整的“输入 -> 处理 -> 输出”的交互闭环。
