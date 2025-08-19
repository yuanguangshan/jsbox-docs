# type: "switch"

`switch` 用于创建一个开关：

```js
{
  type: "switch",
  props: {
    on: true
  },
  layout: function(make, view) {
    make.center.equalTo(view.super)
  }
}
```

将会创建一个初始状态是开启的开关。

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
on | boolean | 读写 | 开关状态
onColor | $color | 读写 | 开启时颜色
thumbColor | $color | 读写 | 滑块颜色

# events: changed

`changed` 事件在状态变化时回调：

```js
changed: function(sender) {
  
}
```

---

## 文件内容解读与示例

### 组件用途

`switch`（开关）组件提供了一个标准的、只有两种状态（开或关）的切换控件。它专门用于表示和控制一个布尔值（`true` 或 `false`）的状态。在设置页面中，如“开启 Wi-Fi”、“进入夜间模式”、“允许通知”等，`switch` 是最理想、最直观的选择。

### 核心属性与事件

- **`on` 属性**: 这是 `switch` 的核心属性，它是一个布尔值，直接对应了开关的两种状态：
  - `true`: 开的状态，通常显示 `onColor` 的背景色。
  - `false`: 关的状态，通常显示为灰色。
  你可以通过在 `props` 中设置 `on` 的初始值，也可以在 `changed` 事件中通过 `sender.on` 读取它的最新状态。

- **`changed` 事件**: 当用户点击开关，使其状态发生改变时，此事件被触发。所有的响应逻辑都应该写在这个事件的回调函数中。

- **颜色属性**: 你可以通过 `onColor`（开启状态下的背景色）和 `thumbColor`（圆形滑块的颜色）来定制开关的外观，以匹配你的应用主题。

### 示例代码：控制夜间模式

下面的示例将创建一个“夜间模式”开关。当用户拨动开关时，整个视图的背景色和状态标签的文本都会随之改变。

```javascript
$ui.render({
  props: {
    id: "main-view",
    title: "Switch 组件示例",
    bgcolor: $color("#F5F5F5") // 初始为亮色背景
  },
  views: [
    {
      // 使用 stack 来水平居中对齐标签和开关
      type: "stack",
      props: {
        axis: $stackViewAxis.horizontal,
        alignment: $stackViewAlignment.center,
        spacing: 10
      },
      layout: (make, view) => {
        make.center.equalTo(view.super);
      },
      views: [
        {
          type: "label",
          props: {
            id: "status-label",
            text: "夜间模式：关"
          }
        },
        {
          type: "switch",
          props: {
            on: false, // 初始状态为关
            onColor: $color("green")
          },
          events: {
            changed: (sender) => {
              const isOn = sender.on;
              if (isOn) {
                // 如果开关打开
                $("main-view").bgcolor = $color("darkGray");
                $("status-label").text = "夜间模式：开";
                $("status-label").textColor = $color("white");
              } else {
                // 如果开关关闭
                $("main-view").bgcolor = $color("#F5F5F5");
                $("status-label").text = "夜间模式：关";
                $("status-label").textColor = $color("black");
              }
            }
          }
        }
      ]
    }
  ]
});
```

**代码解读**：

1.  我们创建了一个 `switch` 组件，并将其初始状态 `on` 设置为 `false`。
2.  核心逻辑在 `changed` 事件中。每次开关被拨动，这个事件就会触发。
3.  我们通过 `const isOn = sender.on;` 获取开关的最新状态。
4.  使用一个 `if/else` 语句，根据 `isOn` 的值来执行不同的操作：
    - 当 `isOn` 为 `true` 时，我们将主视图的背景色变为深灰色，并将标签文本更新为“夜间模式：开”。
    - 当 `isOn` 为 `false` 时，我们将背景色和标签文本恢复为初始的亮色状态。

这个例子清晰地展示了如何使用 `switch` 的状态来驱动应用其他部分的视觉和行为变化，这是它最核心的用法。 
