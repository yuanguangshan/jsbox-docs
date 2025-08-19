# 事件处理

从 v1.49.0 开始，我们引入了更为完善的事件处理机制，能够在视图生成之后对事件处理进行动态绑定，并且完整支持 iOS 内置的所有事件类型。

# view.whenTapped(handler)

当一个视图被按下时触发：

```js
button.whenTapped(() => {
  
});
```

# view.whenDoubleTapped(handler)

当一个视图被按下两次时触发：

```js
button.whenDoubleTapped(() => {
  
});
```

# view.whenTouched(args)

指定视图按下需要的触点个数和按下次数：

```js
button.whenTouched({
  touches: 2,
  taps: 2,
  handler: () => {

  }
});
```

上述代码在视图两指双击的状况下触发。

如果 button 是一个 Runtime 环境下的对象 (ocValue)，可以通过以下两种方式绑定事件：

```js
button.jsValue().whenTapped(() => {
  
});
```

或者使用 $block:

```js
button.$whenTapped($block("void, void", () => {

}));
```

# view.addEventHandler(args)

为视图添加自定义的事件响应：

```js
textField.addEventHandler({
  events: $UIEvent.editingChanged,
  handler: sender => {

  }
});
```

注意：此方法只能用于 `button`, `text`, `input` 等本身就支持事件响应的 UI controls，而对于 `image` 一类的视图则不支持。完整的事件类型请查看：[$UIEvent](data/constant.md?id=uievent)

# view.removeEventHandlers(events)

移除视图上已添加的事件：

```js
textField.removeEventHandlers($UIEvent.editingChanged);
```

同样的，上述代码也可以在 Runtime 环境使用：

```js
textField.$addEventHandler({
  events: $UIEvent.editingChanged,
  handler: $block("void, id", sender => {
    
  })
});
```

---

## 文件内容解读与示例

### 用途说明

本文档介绍了一套**现代化、动态的事件处理机制**。与在视图定义时使用 `events: { ... }` 的传统方式相比，这套新 API 允许你在**视图创建之后**，随时**动态地添加或移除**事件处理器。这为编写更灵活、更模块化的交互逻辑提供了极大的便利。

### 核心概念：动态事件绑定

-   **传统方式的局限**: 在 `events` 块中定义事件处理器，意味着事件逻辑必须在视图初始化时就确定下来。如果想在后期根据条件改变或移除某个事件，会非常困难。
-   **新机制的优势**: 这套 API 将事件处理从静态定义中解放出来。你可以获取到一个视图对象（例如通过 `$(id)`)，然后调用相应的方法（如 `.whenTapped()`) 来**在任意时刻**为其附加行为。这使得代码组织更清晰，特别是在处理复杂的、有状态的 UI 时。

### API 详解

#### 手势事件封装

JSBox 为最常见的几种点击手势提供了便捷的封装：

-   **`view.whenTapped(handler)`**: 响应单击手势。这是 `events: { tapped: ... }` 的动态版本。
-   **`view.whenDoubleTapped(handler)`**: 响应双击手势。
-   **`view.whenTouched({ touches, taps, handler })`**: 更通用的点击手势接口，可以自定义需要的手指数量（`touches`）和点击次数（`taps`）。例如，`{ touches: 2, taps: 1, ... }` 可以用来监听双指单击。

#### 通用事件处理器

-   **`view.addEventHandler({ events, handler })`**: 这是一个更底层的接口，用于监听 iOS `UIControl` 的各种原生事件。它功能强大，但不适用于所有视图。
    -   **`events`**: 指定要监听的事件类型，其值来自于常量集合 `[$UIEvent](data/constant.md?id=uievent)`，例如 `$UIEvent.editingChanged` (文本框内容改变时) 或 `$UIEvent.valueChanged` (开关、滑块等值改变时)。
    -   **适用范围**: 此方法主要用于本身就是“控件”（Control）的视图，如 `button`, `text`, `input`, `switch`, `slider` 等。对于 `view`, `label`, `image` 等非控件视图，应使用上面的手势事件。

-   **`view.removeEventHandlers(events)`**: 移除之前通过 `addEventHandler` 添加的特定事件的处理器。

#### Runtime 环境下的使用

-   文档还提到了如何在 Objective-C Runtime 环境下为原生 `UIView` 对象绑定事件，这属于高级用法。通过 `.jsValue()` 将 OC 对象包装成 JSBox 视图对象后，即可使用这些动态绑定方法。

### 示例：动态控制按钮行为

假设我们有一个开关（switch）和一个按钮（button）。我们希望只有当开关闭合时，按钮才能被点击并执行操作。

```javascript
$ui.render({
  views: [
    {
      type: "switch",
      props: {
        id: "controlSwitch",
        on: false
      },
      layout: (make, view) => {
        make.top.equalTo(20);
        make.centerX.equalTo(view.super);
      }
    },
    {
      type: "button",
      props: {
        id: "actionButton",
        title: "请先打开开关"
      },
      layout: (make, view) => {
        make.top.equalTo($("controlSwitch").bottom).offset(20);
        make.centerX.equalTo(view.super);
        make.width.equalTo(200);
      }
    }
  ]
});

// 获取视图对象
const controlSwitch = $("controlSwitch");
const actionButton = $("actionButton");

// 定义按钮的点击事件处理器
const buttonTappedHandler = () => {
  $ui.alert("操作已执行！");
};

// 为开关添加值改变事件监听
controlSwitch.addEventHandler({
  events: $UIEvent.valueChanged,
  handler: (sender) => {
    if (sender.on) {
      // 开关打开时，为按钮“添加”点击事件
      actionButton.title = "执行操作";
      actionButton.whenTapped(buttonTappedHandler);
    } else {
      // 开关关闭时，为按钮“移除”所有事件
      // 注意：这里没有直接的 removeTapped, 
      // 通常通过改变状态来禁用或重新绑定
      actionButton.title = "请先打开开关";
      // 简单的做法是重新绑定一个空操作
      actionButton.whenTapped(() => {
        $ui.toast("开关未打开");
      });
    }
  }
});

// 初始状态下，按钮点击无效
actionButton.whenTapped(() => {
  $ui.toast("开关未打开");
});
```

### 总结

这套动态事件处理 API 是对 JSBox `UIKit` 框架的重要补充。它使得事件管理更加灵活和程序化，让开发者能够根据应用的实时状态来控制UI的交互行为，是构建复杂、动态用户界面的利器。