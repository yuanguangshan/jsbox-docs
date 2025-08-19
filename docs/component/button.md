# type: "button"

`button` 可以创建一个按钮，用于处理用户的点击事件：

```js
{
  type: "button",
  props: {
    title: "Click"
  },
  layout: function(make, view) {
    make.center.equalTo(view.super)
  }
}
```

在画布中显示一个标题为 `Click` 的按钮。

类似 image 组件，button 也支持通过 src 属性来设置显示的图片。

另外 button 也支持显示 JSBox 自带的 icon，请参考 [$icon](data/method.md?id=iconcode-color-size)。

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
title | string | 读写 | 标题
titleColor | $color | 读写 | 标题颜色
font | $font | 读写 | 字体
src | string | 读写 | 图片地址
source | object | 读写 | 图片加载信息
symbol | string | 读写 | SF symbols 名称
image | image | 读写 | 图片对象
icon | $icon | 只写 | 内置图标
type | $btnType | 只读 | 类型
menu | object | 只写 | Pull-Down 菜单
contentEdgeInsets | $insets | 读写 | 内容边距
titleEdgeInsets | $insets | 读写 | 标题边距
imageEdgeInsets | $insets | 读写 | 图片边距

# props: source

从 v1.55.0 开始，可以通过 `source` 对图片进行更详细的设定，例如：

```js
source: {
  url: url,
  placeholder: image,
  header: {
    "key1": "value1",
    "key2": "value2",
  }
}
```

# events: tapped

尤其不要忘记的是，button 也支持 `tapped` 事件：

```js
events: {
  tapped: function(sender) {
    
  }
}
```

# Pull-Down 菜单

`button` 视图支持 Pull-Down 菜单，请参考 [Pull-Down 菜单](uikit/context-menu?id=pull-down-菜单) 了解更多。

---

## 文件内容解读与示例

### 组件用途

`button`（按钮）是任何交互式 UI 中最基础、最核心的组件。它的主要作用是响应用户的点击（tap）操作，并执行预定义的代码逻辑。按钮的外观非常灵活，可以显示文字、图标、系统符号或自定义图片。

### 核心功能与属性

#### 1. 按钮内容 (Content)

一个按钮可以有多种内容形式，你需要根据场景选择合适的属性：

- **文字按钮**: 最基本的形式，通过 `props: { title: "按钮文字" }` 来设置。
- **图标按钮**: 
  - **SF Symbol**: **（推荐）** 使用 `props: { symbol: "flame.fill" }` 来设置。SF Symbol 是 Apple 官方的矢量图标库，美观且能自适应字体设置。你可以在 [SF Symbols 应用](https://developer.apple.com/sf-symbols/)中查找所有可用的图标名称。
  - **JSBox 内置图标**: 使用 `props: { icon: $icon("love", $color("red")) }` 设置，这是 JSBox 的内置图标集。
- **图片按钮**: 使用 `props: { src: "https://path/to/image.png" }` 或 `image: $image("assets/logo.png")` 来设置。适用于需要高度自定义视觉效果的场景。

#### 2. 核心事件 `tapped`

这是按钮的灵魂所在。所有点击后需要执行的逻辑都应写在 `events: { tapped: function(sender) { ... } }` 中。

- **`sender` 参数**: `tapped` 函数会接收一个 `sender` 参数，它指向**按钮视图本身**。通过 `sender`，你可以在点击事件内部动态地修改按钮的属性，例如更改标题、禁用按钮等，非常灵活。

#### 3. Pull-Down 菜单

通过设置 `menu` 属性，可以给按钮附加一个下拉菜单。当用户长按或点击按钮时，会弹出一个选项列表。这是一种现代的、用于替代传统“动作表单”（Action Sheet）的交互方式。

### 示例代码：不同类型的按钮及其交互

下面的示例将创建一个界面，包含一个简单的文本按钮、一个可动态改变自身的图标按钮和一个禁用的按钮，以展示其用法。

```javascript
$ui.render({
  props: {
    title: "Button 组件示例"
  },
  views: [
    {
      // 1. 一个基本的文本按钮
      type: "button",
      props: {
        title: "点我弹窗",
        font: $font(18),
        type: $btnType.primary // 使用预设的主要按钮样式
      },
      layout: function(make, view) {
        make.top.equalTo(view.super).offset(20);
        make.centerX.equalTo(view.super);
        make.width.equalTo(150);
      },
      events: {
        tapped: function(sender) {
          $ui.alert("你点击了按钮！");
        }
      }
    },
    {
      // 2. 一个带图标的、可交互的按钮
      type: "button",
      props: {
        id: "interactive-btn",
        title: " 点我变身",
        symbol: "star.fill", // 使用 SF Symbol
        titleColor: $color("orange")
      },
      layout: function(make, view) {
        make.top.equalTo($("button").bottom).offset(20);
        make.centerX.equalTo(view.super);
        make.width.equalTo(150);
      },
      events: {
        tapped: function(sender) {
          // sender 指向这个按钮本身
          sender.title = " 变身成功";
          sender.symbol = "checkmark.circle.fill";
          sender.titleColor = $color("green");
          sender.enabled = false; // 完成任务后禁用自己
        }
      }
    },
    {
      // 3. 一个被禁用的按钮
      type: "button",
      props: {
        title: "我被禁用了",
        enabled: false // 设置为 false，按钮将变灰且不可点击
      },
      layout: function(make, view) {
        make.top.equalTo($("interactive-btn").bottom).offset(20);
        make.centerX.equalTo(view.super);
        make.width.equalTo(150);
      }
    }
  ]
});
```

**代码解读**：

1.  第一个按钮展示了最基础的用法：设置标题，并在 `tapped` 事件中执行 `$ui.alert`。
2.  第二个按钮展示了 `sender` 参数的妙用：当按钮被点击时，我们通过 `sender.title`、`sender.symbol` 等直接修改了按钮自己的外观，并最后用 `sender.enabled = false` 禁用了它，防止重复点击。
3.  第三个按钮通过 `props: { enabled: false }` 在初始时就被设置为禁用状态，用户无法与其交互。

掌握 `button` 组件是编写任何有实际功能的 UI 脚本的第一步。 
