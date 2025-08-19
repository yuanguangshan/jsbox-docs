# type: "slider"

`slider` 用于创建一个可拖动的进度选择器：

```js
{
  type: "slider",
  props: {
    value: 0.5,
    max: 1.0,
    min: 0.0
  },
  layout: function(make, view) {
    make.center.equalTo(view.super)
    make.width.equalTo(100)
  }
}
```

将会创建一个取值范围为 0.0 ~ 1.0 的选择器，初始值是 0.5。

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
value | number | 读写 | 当前值
min | number | 读写 | 最小值
max | number | 读写 | 最大值
continuous | boolean | 读写 | 是否连续
minColor | $color | 读写 | 左边颜色
maxColor | $color | 读写 | 右边颜色
thumbColor | $color | 读写 | 滑块颜色

# events: changed

`changed` 事件在数值变化时回调：

```js
changed: function(sender) {

}
```

---

## 文件内容解读与示例

### 组件用途

`slider`（滑块）组件提供了一个水平条，用户可以在上面拖动一个滑块（thumb）来在一个连续的数值范围内进行选择。它常用于调节某个设置项，如音量、亮度、画笔大小、播放进度等。

### 核心属性详解

- **`min`, `max`, `value`**: 这三个属性共同定义了滑块的行为。
  - `min`: 允许的最小值。
  - `max`: 允许的最大值。
  - `value`: 滑块当前所处的值。你可以设置它的初始值，并在 `changed` 事件中读取它的新值。

- **`continuous` (连续性)**: 这是一个重要的布尔属性，决定了 `changed` 事件的触发时机。
  - `true` (默认): 当用户**拖动滑块的过程中**，`changed` 事件会**持续、实时地**触发。
  - `false`: `changed` 事件只会在用户**松开手指**的那一刻触发一次。
  - **如何选择**: 如果滑块控制的更新操作非常快（如改变颜色），使用 `true` 可以提供实时反馈。如果更新操作比较耗时，或者你只关心最终结果，使用 `false` 可以避免不必要的性能开销。

- **颜色属性**: `minColor`（滑块左侧轨道的颜色）、`maxColor`（滑块右侧轨道的颜色）和 `thumbColor`（滑块本身的颜色）可以让你轻松地自定义滑块的外观。

### 核心事件 `changed`

这是 `slider` 最主要的事件。当滑块的值发生改变时（根据 `continuous` 的设置），此事件会被触发。回调函数会收到 `sender` 参数，即滑块视图本身，你可以通过 `sender.value` 来获取当前最新的数值。

### 示例代码：一个简单的字体大小调节器

下面的示例将创建一个滑块和一个标签。拖动滑块时，标签的字体大小会实时地发生变化。

```javascript
$ui.render({
  props: {
    title: "Slider 组件示例"
  },
  views: [
    {
      type: "label",
      props: {
        id: "preview-label",
        text: "拖动下方滑块改变我的大小",
        font: $font(12), // 初始字体大小
        align: $align.center
      },
      layout: (make, view) => {
        make.centerX.equalTo(view.super);
        make.top.inset(50);
      }
    },
    {
      type: "slider",
      props: {
        id: "font-slider",
        min: 12, // 最小字号
        max: 48, // 最大字号
        value: 12, // 初始字号
        continuous: true // 持续触发 changed 事件
      },
      layout: (make, view) => {
        make.top.equalTo($("preview-label").bottom).offset(30);
        make.left.right.inset(40);
      },
      events: {
        changed: (sender) => {
          // 1. 从 sender.value 获取当前滑块的值
          const newSize = sender.value;
          // 2. 更新 label 的 font 属性
          $("preview-label").font = $font(newSize);
        }
      }
    }
  ]
});
```

**代码解读**：

1.  我们创建了一个 `slider`，并设置了它的取值范围 `min: 12` 到 `max: 48`，这对应我们希望的字体大小范围。
2.  `continuous: true` 的设置确保了在拖动过程中，`changed` 事件会实时触发，从而带来流畅的预览体验。
3.  在 `changed` 事件的回调中，我们从 `sender.value` 获取了滑块的当前值，并立刻用这个值创建一个新的 `$font` 对象，然后将其赋给上方 `label` 的 `font` 属性，实现了两个组件之间的实时联动。

`slider` 是进行数值范围选择的理想工具，通过响应 `changed` 事件，你可以轻松地将用户的拖动操作与应用中的任何数值属性绑定起来。 
