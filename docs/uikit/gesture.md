# 手势识别

在触摸设备里面，手势是一个很重要的交互形式，这一伟大的创新得益于 2007 年 [iPhone 问世](https://www.youtube.com/watch?v=x7qPAY9JqE4)。

iOS 支持丰富的手势识别，在 JSBox 里面目前仅支持重要的几个。

# events: tapped

实现了 `tapped` 事件的 view 会响应点击手势：

```js
tapped: function(sender) {

}
```

# events: longPressed

实现了 `longPressed` 事件的 view 会响应长按手势：

```js
longPressed: function(info) {
  let sender = info.sender;
  let location = info.location;
}
```

# events: doubleTapped

实现了 `doubleTapped` 事件的 view 会响应双击事件：

```js
doubleTapped: function(sender) {

}
```

# events: touchesBegan

当点击事件触发时调用：

```js
touchesBegan: function(sender, location, locations) {

}
```

# events: touchesMoved

当点击发生移动时调用：

```js
touchesMoved: function(sender, location, locations) {

}
```

# events: touchesEnded

当点击事件结束时调用：

```js
touchesEnded: function(sender, location, locations) {

}
```

# events: touchesCancelled

当点击事件取消时调用：

```js
touchesCancelled: function(sender, location, locations) {

}
```

---

## 文件内容解读与示例

### 用途说明

本文档介绍了在 JSBox 中处理**手势识别**的**传统方法**。手势是触摸屏交互的核心，JSBox 将 iOS 中最常用的一些手势封装为简单的事件，可以直接在视图的 `events` 块中进行定义。这套 API 主要用于处理基本的点击、长按以及更底层的触摸事件序列。

**注意**: 这是在视图定义时**静态绑定**事件的方式。对于需要动态添加或移除手势的场景，推荐使用[新版的事件处理机制](uikit/event.md)。

### API 详解 (高级手势)

JSBox 将常见手势封装为易于使用的事件：

-   **`events: { tapped: (sender) => { ... } }`**: 
    -   **触发**: 单次点击（手指按下后快速抬起）。
    -   **用途**: 最常见的交互，用于触发按钮点击、项目选择等操作。

-   **`events: { doubleTapped: (sender) => { ... } }`**: 
    -   **触发**: 快速连续点击两次。
    -   **用途**: 常用于放大图片、收藏内容等快捷操作。

-   **`events: { longPressed: (info) => { ... } }`**: 
    -   **触发**: 手指在屏幕上按住并保持一段时间。
    -   **用途**: 通常用于触发上下文菜单（Context Menu）、进入编辑模式或显示额外信息。
    -   **`info` 对象**: 回调函数接收一个 `info` 对象，包含两个属性：
        -   `info.sender`: 触发手势的视图本身。
        -   `info.location`: 手指长按在视图坐标系中的位置（一个 `$point` 对象）。

### API 详解 (底层触摸事件)

这组事件提供了对触摸过程更精细的控制，让你能够追踪一个或多个手指从接触屏幕到离开屏幕的完整轨迹。它们通常用于实现自定义的、复杂的手势，如拖动、绘制等。

-   **`events: { touchesBegan: (sender, location, locations) => { ... } }`**: 
    -   **触发**: 当一个或多个手指**初次接触**到视图时调用。
    -   **用途**: 标记一个触摸序列的开始，可以用来记录初始位置或改变视图外观以响应触摸。

-   **`events: { touchesMoved: (sender, location, locations) => { ... } }`**: 
    -   **触发**: 当手指在屏幕上**移动**时被**连续调用**。
    -   **用途**: 实现拖动、绘图等核心逻辑的地方。你会在这里根据手指位置的变化来更新视图的 `frame` 或绘制路径。

-   **`events: { touchesEnded: (sender, location, locations) => { ... } }`**: 
    -   **触发**: 当一个或多个手指**离开**屏幕时调用，标志着一次成功的触摸序列结束。
    -   **用途**: 标记触摸序列的结束，可以用来执行最终操作或将视图动画地恢复到某个状态。

-   **`events: { touchesCancelled: (sender, location, locations) => { ... } }`**: 
    -   **触发**: 当触摸序列被系统**意外中断**时调用（例如，来电话）。
    -   **用途**: 处理中断情况，通常需要在这里将视图恢复到初始状态，并撤销 `touchesBegan` 中所做的更改。

-   **参数说明**: 
    -   `sender`: 触发事件的视图。
    -   `location`: 第一个触摸点在视图坐标系中的位置 (`$point`)。
    -   `locations`: 所有触摸点位置的数组 (Array of `$point`)，用于多点触控。

### 示例：实现一个可拖动的方块

这个例子演示了如何使用底层的触摸事件来实现一个可以被用户在屏幕上任意拖动的视图。

```javascript
let initialCenter = null;

$ui.render({
  views: [
    {
      type: "view",
      props: {
        id: "draggableBox",
        bgcolor: $color("tint"),
        cornerRadius: 10
      },
      layout: (make, view) => {
        make.center.equalTo(view.super);
        make.size.equalTo($size(100, 100));
      },
      events: {
        // 手指按下时：记录视图的初始中心点
        touchesBegan: (sender) => {
          initialCenter = sender.center;
          $device.taptic(1); // 轻微震动提示
        },
        // 手指移动时：计算偏移量并更新视图中心点
        touchesMoved: (sender, location, locations) => {
          // location 是相对于视图本身的，(0,0)是左上角
          // 我们需要转换到父视图坐标系来计算位移
          const touchInSuperview = sender.convertPointToView(location, sender.super);
          const initialTouchInSuperview = sender.convertPointToView($point(50, 50), sender.super);
          
          const delta = $point(touchInSuperview.x - initialTouchInSuperview.x, touchInSuperview.y - initialTouchInSuperview.y);

          sender.center = $point(initialCenter.x + delta.x, initialCenter.y + delta.y);
        },
        // 手指抬起时：重置初始位置记录
        touchesEnded: (sender) => {
          initialCenter = null;
          $device.taptic(2);
        },
        // 意外中断时：同样重置
        touchesCancelled: (sender) => {
          initialCenter = null;
        }
      }
    }
  ]
});
```

### 总结

通过 `events` 块定义手势是 JSBox 中实现交互的基础。高级手势（`tapped`, `longPressed`）满足了大部分常见需求，而底层触摸事件（`touchesBegan` 等）则为实现自定义拖动、缩放、旋转等复杂手势提供了必要的工具。