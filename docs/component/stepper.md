# type: "stepper"

`stepper` 用于创建控制加减的控件：

```js
{
  type: "stepper",
  props: {
    max: 10,
    min: 1,
    value: 5
  },
  layout: function(make, view) {
    make.centerX.equalTo(view.super)
    make.top.equalTo(24)
  },
  events: {
    changed: function(sender) {

    }
  }
}
```

创建一个取值范围 1 ~ 10，初始值是 5 的步进器。

> 步进器仅显示一个加减控件，你可以用 label 组件显示其数值。

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
value | number | 读写 | 当前数值
min | number | 读写 | 最小值
max | number | 读写 | 最大值
step | number | 读写 | 步长
autorepeat | boolean | 读写 | 响应长按
continuous | boolean | 读写 | 连续响应事件

# events: changed

`changed` 事件在数值变化时回调：

```js
changed: function(sender) {
  
}
```

---

## 文件内容解读与示例

### 组件用途

`stepper`（步进器）提供了一个包含“+”和“-”按钮的标准控件，用于让用户以固定的步长增加或减少一个数值。它非常适合用于购物车中的商品数量选择、设置项中的数量调整等场景。

### 核心概念：与 `label` 配合使用

**一个至关重要的概念是：`stepper` 组件本身只显示加减按钮，它不显示当前的数值。**

如文档中所提示，你必须自己额外添加一个 `label` 组件来显示 `stepper` 的当前值。然后，通过在 `stepper` 的 `changed` 事件中更新 `label` 的文本，来实现两者之间的联动。这是一个固定的使用模式。

### 核心属性与事件

- **`min`, `max`, `value`**: 定义了步进器的数值范围和当前值。
  - `min`: 允许的最小值。当达到最小值时，“-”按钮会自动变为禁用状态。
  - `max`: 允许的最大值。当达到最大值时，“+”按钮会自动变为禁用状态。
  - `value`: 当前的数值。

- **`step`**: 每一次点击“+”或“-”时，`value` 改变的量。默认为 `1`。

- **`autorepeat`**: 一个布尔值，默认为 `true`。当设置为 `true` 时，用户按住“+”或“-”按钮不放，数值会持续地改变。

- **`changed` 事件**: 当用户点击按钮导致 `value` 发生变化时，此事件被触发。这是 `stepper` 最核心的事件，所有的响应逻辑都应写在这里。你可以通过 `sender.value` 获取到最新的数值。

### 示例代码：商品数量选择器

下面的示例将演示如何将一个 `stepper` 和一个 `label` 组合起来，创建一个经典的商品数量选择器。

```javascript
$ui.render({
  props: { title: "Stepper 组件示例" },
  views: [
    {
      // 使用一个水平 stack 来对齐各个元素
      type: "stack",
      props: {
        axis: $stackViewAxis.horizontal,
        alignment: $stackViewAlignment.center,
        spacing: 15
      },
      layout: (make, view) => {
        make.center.equalTo(view.super);
        make.height.equalTo(40);
      },
      views: [
        // 商品名称
        { type: "label", props: { text: "商品A:" } },
        // 数量显示标签
        {
          type: "label",
          props: {
            id: "quantity-label",
            text: "1", // 初始数量
            font: $font("bold", 18),
            align: $align.center,
            frame: { width: 40 }
          }
        },
        // 步进器控件
        {
          type: "stepper",
          props: {
            min: 1,      // 最小数量为1
            max: 10,     // 最大数量为10
            value: 1,    // 初始值为1
            step: 1      // 步长为1
          },
          events: {
            changed: (sender) => {
              // 当 stepper 的值改变时，更新 label 的文本
              $("quantity-label").text = sender.value.toString();
            }
          }
        }
      ]
    }
  ]
});
```

**代码解读**：

1.  我们用一个 `stack` 布局来将商品名 `label`、数量 `label` 和 `stepper` 水平排列整齐。
2.  `stepper` 组件负责处理加减的逻辑和数值的增减。我们设置了它的 `min`, `max` 和 `value`。
3.  `id` 为 `quantity-label` 的 `label` 组件专门负责**显示**当前的数值。
4.  **核心联动逻辑**在 `stepper` 的 `changed` 事件中：每当 `stepper` 的值发生变化，我们就通过 `sender.value` 获取新值，并立即更新 `quantity-label` 的 `text` 属性。这就实现了两个独立组件之间的完美配合。

这个“**`stepper` 负责逻辑，`label` 负责显示**”的组合模式，是使用 `stepper` 组件的标准方法。 
