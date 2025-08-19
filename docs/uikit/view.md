> JSBox 提供了一套绘制视图的方案，这里我们介绍这套方案的核心概念

# type: "view"

`view` 是最基层的控件，所有控件的父类：

```js
$ui.render({
  views: [
    {
      type: "view",
      props: {
        bgcolor: $color("#FF0000")
      },
      layout: function(make, view) {
        make.center.equalTo(view.super)
        make.size.equalTo($size(100, 100))
      },
      events: {
        tapped: function(sender) {

        }
      }
    }
  ]
})
```

将会绘制一个红色的视图，`view` 是所有控件的父类，所以 view 支持的属性其他控件都支持。

PS: 关于各种属性的类型转换请参考：[数据转换](data/intro.md)

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
theme | string | 读写 | light, dark, auto
alpha | number | 读写 | 透明度
bgcolor | $color | 读写 | 背景色
cornerRadius | number | 读写 | 圆角半径
smoothCorners | boolean | 读写 | 圆角是否使用平滑曲线
radius | number | 只写 | 圆角半径（过时，请使用 `cornerRadius`）
smoothRadius | number | 只写 | 平滑圆角半径（过时，请使用 `smoothCorners`）
frame | $rect | 读写 | 位置和大小
size | $size | 读写 | 大小
center | $point | 读写 | 中心位置
flex | string | 读写 | 自动缩放规则
userInteractionEnabled | boolean | 读写 | 是否响应触摸事件
multipleTouchEnabled | boolean | 读写 | 是否支持多点触摸
super | view | 只读 | 父视图
prev | view | 只读 | 前一个视图
next | view | 只读 | 后一个视图
window | view | 只读 | 所属的 window
views | array | 只读 | 子视图
clipsToBounds | boolean | 读写 | 是否裁剪子 view
opaque | boolean | 读写 | 是否不透明
hidden | boolean | 读写 | 是否隐藏
contentMode | $contentMode | 读写 | [请参考](https://developer.apple.com/documentation/uikit/uiview/1622619-contentmode)
tintColor | $color | 读写 | 着色
borderWidth | number | 读写 | 边框宽度
borderColor | $color | 读写 | 边框颜色
circular | bool | 读写 | 是否圆形
animator | object | 只读 | 动画对象
snapshot | object | 只读 | 生成截图
info | object | 读写 | 用于绑定一些信息，例如上下文参数
intrinsicSize | $size | 读写 | 固有内容尺寸
isAccessibilityElement | bool | 读写 | 是否支持无障碍
accessibilityLabel | string | 读写 | accessibility label
accessibilityHint | string | 读写 | accessibility hint
accessibilityValue | string | 读写 | accessibility value
accessibilityCustomActions | array | 读写 | [accessibility custom actions](function/index?id=accessibilityactiontitle-handler)

注意：你不能在 layout 函数里面使用 `next`，因为这个时候视图结构还没有被生成。

从 v1.36.0 版本开始，可以通过 $ui.render("main.ux") 来渲染一个通过可视化界面编辑器生成的页面。

# navButtons

我们可以在界面右上角自定义按钮，例如：

```js
$ui.render({
  props: {
    navButtons: [
      {
        title: "Title",
        image, // Optional
        icon: "024", // Or you can use icon name
        symbol: "checkmark.seal", // SF symbols are supported
        handler: sender => {
          $ui.alert("Tapped!")
        },
        menu: {
          title: "Context Menu",
          items: [
            {
              title: "Title",
              handler: sender => {}
            }
          ]
        } // Pull-Down menu
      }
    ]
  }
})
```

了解关于 Pull-Down menu 的更多信息，请参考 [Pull-Down 菜单](uikit/context-menu?id=pull-down-菜单)。

# titleView

除了通过 `title` 来指定标题，还可以通过 `titleView` 来指定 navBar 顶部展示的视图：

```js
$ui.render({
  props: {
    titleView: {
      type: "tab",
      props: {
        bgcolor: $rgb(240, 240, 240),
        items: ["A", "B", "C"]
      },
      events: {
        changed: sender => {
          console.log(sender.index);
        }
      }
    }
  },
  views: [

  ]
});
```

# layout(function)

手动触发 view 的 layout 方法，参数和 view 定义时的 `layout` 函数完全相同：

```js
view.layout((make, view) => {
  make.left.top.right.equalTo(0);
  make.height.equalTo(100);
});
```

# updateLayout(function)

`updateLayout` 方法可以更新一个控件的 layout：

```js
$("label").updateLayout(make => {
  make.size.equalTo($size(200, 200))
})
```

请注意，`updateLayout` 只能对**已存在**的约束进行更新，否则的话将没有效果。

# remakeLayout(function)

和 updateLayout 类似，但是重新设置 layout 会导致更多性能消耗，在可能时应该使用 updateLayout。

# add(object)

动态添加一个子 view，object 的结构定义和 `$ui.render(object)` 中 view 的完全一致，也可以是通过 `$ui.create(...)` 创建出来的 view 实例。

# get(id)

通过 id 获取一个子 view。

# remove()

将此 view 从父视图中移除。

# insertBelow(view, other)

将一个新的视图插入到一个已存在视图的下方：

```js
view.insertBelow(newView, existingView);
```

# insertAbove(view, other)

将一个新的视图插入到一个已存在视图的上方：

```js
view.insertAbove(newView, existingView);
```

# insertAtIndex(view, index)

将一个新的视图插入到一个指定的位置：

```js
view.insertAtIndex(newView, 4);
```

# moveToFront()

将自己移动到父视图的顶部：

```js
existingView.moveToFront();
```

# moveToBack()

将自己移动到父视图的底部：

```js
existingView.moveToBack();
```

# relayout()

更新 layout，此方法可能在动画中调用。

view 添加之后并不会立即 layout，此方法可以让新的约束立即执行，并可以在之后获取诸如 frame 或者 size 之类的信息。

# setNeedsLayout()

将此 view 标记为需要 layout，会在下一个绘制循环中被 layout。

# layoutIfNeeded()

强制触发下一个绘制循环，可以配合 `setNeedsLayout` 使用：

```js
view.setNeedsLayout();
view.layoutIfNeeded();
```

# sizeToFit()

将 view 调整到当前 bounds 下最合适的大小。

# scale(number)

将控件缩放到一个比例，例如：

```js
view.scale(0.5)
```

# snapshotWithScale(scale)

指定比例得到一个截图：

```js
const image = view.snapshotWithScale(1)
```

# rotate(number)

将控件旋转一个角度，例如：

```js
view.rotate(Math.PI)
```

# events: ready

所有的 view 都支持 `ready` 事件，将会在 view 初始化完成之后调用：

```js
ready: function(sender) {
  
}
```

# events: tapped

所有的 view 都支持 `tapped` 事件：

```js
tapped: function(sender) {

}
```

该事件会在被点击的时候调用，`sender` 表示触发此事件的对象，在上述例子中就是 view 本身。

另外，events 也可以简写成：

```js
tapped(sender) {

}
```

和上述代码具有同样效果。

# events: pencilTapped

所有的 view 都支持 `pencilTapped` 来检测来自 Apple Pencil 的点击：

```js
pencilTapped: function(info) {
  var action = info.action; // 0: Ignore, 1: Switch Eraser, 2: Switch Previous, 3: Show Color Palette
  var enabled = info.enabled; // whether the system reports double taps on Apple Pencil to your app
}
```

# events: hoverEntered

在 iPadOS 13.4 及以上使用 Trackpad，指针进入时调用：

```js
hoverEntered: sender => {
  sender.alpha = 0.5;
}
```

# events: hoverExited

在 iPadOS 13.4 及以上使用 Trackpad，指针移出时调用：

```js
hoverExited: sender => {
  sender.alpha = 1.0;
}
```

# events: themeChanged

用于监听 [Dark Mode](uikit/dark-mode.md) 的改变：

```js
themeChanged: (sender, isDarkMode) => {
  
}
```

更多控件如何使用请参考 [控件列表](component/label.md) 一章。

---

## 文件内容解读与示例

### 用途说明

本文档是 **JSBox UI 框架的“基类”文档**，详细介绍了 `type: "view"` 这个最基础的视图组件。`view` 是所有其他 UI 组件（如 `label`, `button`, `image` 等）的父类，因此，**本文档中介绍的所有属性、方法和事件，对于其他所有 UI 组件来说，都是通用的**。掌握 `view` 的能力，是理解和使用整个 UI 框架的基础。

### 核心概念：视图实例 (View Instance)

当你通过 `$ui.render` 或 `$ui.create` 创建一个界面元素后，你就可以通过代码（例如使用 `$(id)`) 获取到这个元素的**实例**。这个实例是一个对象，它暴露了大量的属性和方法，允许你在视图显示后，动态地查询其状态或修改其行为和外观。

### 属性 (Props) 详解

`props` 列表非常长，这里按功能分类解读一些最重要的属性：

-   **外观**: `alpha` (透明度), `bgcolor` (背景色), `cornerRadius` (圆角), `borderWidth`/`borderColor` (边框), `tintColor` (着色，影响子视图如图标), `hidden` (是否隐藏)。
-   **位置与大小**: `frame`, `size`, `center`。这些属性既可读也可写，但**强烈建议**通过 Auto Layout 进行布局，只在必要时读取它们的值。
-   **层级关系**: `super` (父视图), `views` (子视图数组), `prev`/`next` (同级的前后视图)。这些都是只读的，用于在视图树中导航。
-   **交互**: `userInteractionEnabled` (是否响应点击), `multipleTouchEnabled` (是否支持多点触控)。
-   **数据绑定**: `info` 是一个非常有用的“杂物箱”，你可以在这个对象里存取任何自定义的数据，将其与视图实例关联起来。
-   **无障碍 (Accessibility)**: `isAccessibilityElement`, `accessibilityLabel` 等属性用于适配 iOS 的“旁白”功能，让视障用户也能使用你的脚本。

### 方法 (Methods) 详解

这些方法提供了动态操作视图的能力。

-   **布局相关**:
    -   `layout(make => ...)`: 为通过 `$ui.create` 创建的、尚未布局的视图设置约束。
    -   `updateLayout(make => ...)`: **更新**已存在的约束。例如，改变一个视图的高度。只能修改，不能添加新约束。
    -   `remakeLayout(make => ...)`: **重置**所有约束。性能开销比 `updateLayout` 大，但更灵活，可以完全改变布局逻辑。
    -   `setNeedsLayout()` / `layoutIfNeeded()`: 手动控制布局刷新时机，用于需要立即获取更新后 `frame` 的高级场景。

-   **视图层级操作**:
    -   `add(view)`: 添加一个子视图。
    -   `remove()`: 将视图从其父视图中移除。
    -   `insert...`, `moveTo...`: 提供对子视图顺序的精细控制。

-   **变换与效果**:
    -   `scale(number)`, `rotate(number)`: 快速应用缩放和旋转变换。
    -   `snapshotWithScale(scale)`: 对视图进行“截图”，返回一个 `image` 对象。

### 事件 (Events) 详解

-   **`ready`**: 视图**初始化完成**后触发。这是最早可以安全访问视图所有属性和方法的地方，非常适合在这里执行一些初始设置。
-   **`tapped`**: 最常用的事件，响应单击。
-   **`pencilTapped`**: 专门用于响应 Apple Pencil 的双击操作。
-   **`hoverEntered` / `hoverExited`**: 用于适配 iPadOS 的触控板/鼠标，当指针进入或离开视图区域时触发。
-   **`themeChanged`**: 监听亮色/暗色模式的切换。

### 页面级组件 (`navButtons`, `titleView`)

-   **`navButtons`**: 在页面的 `props` 中定义，用于在导航栏右侧创建自定义按钮。每个按钮都可以有标题、图标、事件处理器，甚至可以关联一个下拉菜单 (`menu`)。
-   **`titleView`**: 同样在页面 `props` 中定义，允许你用一个完全自定义的视图来替换导航栏中央的标题文本。例如，可以用一个 `tab` 视图作为 `titleView` 来实现分段选择器。

### 示例：动态修改视图

```javascript
$ui.render({
  props: { title: "View 实例操作" },
  views: [
    {
      type: "view",
      props: {
        id: "myBox",
        bgcolor: $color("blue"),
        cornerRadius: 10
      },
      layout: (make, view) => {
        make.center.equalTo(view.super);
        make.size.equalTo($size(100, 100));
      },
      events: {
        ready: (sender) => {
          // 视图准备好后，绑定一个自定义信息
          sender.info = { creationTime: new Date() };
          console.log("Box created at: " + sender.info.creationTime);
        }
      }
    },
    {
      type: "button",
      props: { title: "变色并放大" },
      layout: (make, view) => {
        make.top.equalTo($("myBox").bottom).offset(20);
        make.centerX.equalTo(view.super);
      },
      events: {
        tapped: () => {
          // 获取 myBox 实例并操作它
          const box = $("myBox");
          box.bgcolor = $color("red");
          box.updateLayout(make => {
            make.size.equalTo($size(150, 150));
          });
          // 动画地应用布局变化
          $ui.animate({ 
            duration: 0.4,
            animation: () => box.relayout()
          });
        }
      }
    }
  ]
});
```

### 总结

`view` 是 JSBox UI 宇宙中的“原子”。本文档中详述的属性、方法和事件构成了整个 UI 框架的通用能力集。无论你使用哪种更具体的组件，它们都继承了 `view` 的这些基本特性。因此，深入理解本文档是进行任何高级 UI 编程、实现动态和交互式界面的前提。