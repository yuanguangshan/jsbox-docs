# type: "scroll"

`scroll` 用于布局一系列 views 在一个滚动的容器里：

```js
{
  type: "scroll",
  layout: function(make, view) {
    make.center.equalTo(view.super)
    make.size.equalTo($size(100, 500))
  },
  views: [
    {

    },
    {

    }
  ]
}
```

其实和创建别的 views 没有什么区别，唯一的不同是这个容器是可以滚动的。

# 视图大小

`scroll` 组件的可滚动区域大小是一个复杂的话题，有两种方法来调整大小。

方法一，scroll 组件的每个子 view 都显式的设置了大小，这样 scroll 组件会自动调整自己的大小：

```js
$ui.render({
  views: [{
    type: "scroll",
    layout: $layout.fill,
    views: [{
      type: "label",
      props: {
        text,
        lines: 0
      },
      layout: function(make, view) {
        make.width.equalTo(view.super)
        make.top.left.inset(0)
        make.height.equalTo(1000)
      }
    }]
  }]
})
```

方法二，手动给 scroll 组件设置 contentSize:

```js
props: {
  contentSize: $size(0, 1000)
}
```

简而言之，`scroll` 组件的高度依赖于设置，他不能完全靠子 view 计算得到。

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
contentOffset | $point | 读写 | 滚动偏移
contentSize | $size | 读写 | 滚动区域大小
alwaysBounceVertical | boolean | 读写 | 永远可以上下滚动
alwaysBounceHorizontal | boolean | 读写 | 永远可以左右滚动
pagingEnabled | boolean | 读写 | 是否分页
scrollEnabled | boolean | 读写 | 是否可以滚动
showsHorizontalIndicator | boolean | 读写 | 显示横向滚动条
showsVerticalIndicator | boolean | 读写 | 显示纵向滚动条
contentInset | $insets | 读写 | 内容边距
indicatorInsets | $insets | 读写 | 滚动条边距
tracking | boolean | 只读 | 用户是否在触摸
dragging | boolean | 只读 | 用户是否在拖动
decelerating | boolean | 只读 | 是否正在减速
keyboardDismissMode | number | 读写 | 键盘收起模式
zoomEnabled | bool | 读写 | 内置图片是否可以缩放
maxZoomScale | number | 读写 | 图片缩放最大比例
doubleTapToZoom | number | 读写 | 是否双击缩放

# beginRefreshing()

开始显示下拉刷新的加载动画。

# endRefreshing()

结束显示下拉刷新加载动画。

# resize()

根据内容自动调整可滚动区域的大小。

# updateZoomScale()

重新计算可缩放视图的最佳比例，您可能需要在屏幕旋转后调用这个方法。

# scrollToOffset($point)

滚动到一个偏移量：

```js
$("scroll").scrollToOffset($point(0, 100));
```

# events: pulled

`pulled` 方法将触发下拉刷新：

```js
pulled: function(sender) {
  
}
```

# events: didScroll

`didScroll` 在视图滚动时回调：

```js
didScroll: function(sender) {

}
```

# events: willBeginDragging

`willBeginDragging` 在用户开始拖动时回调：

```js
willBeginDragging: function(sender) {

}
```

# events: willEndDragging

`willEndDragging` 在将要结束拖拽时回调：

```js
willEndDragging: function(sender, velocity, target) {

}
```

其中 `target` 为滚动停下来时候的位置，可以通过返回一个 `$point` 进行覆盖：

```js
willEndDragging: function(sender, velocity, target) {
  return $point(0, 0);
}
```

# events: didEndDragging

`didEndDragging` 在已经结束拖拽时回调：

```js
didEndDragging: function(sender, decelerate) {

}
```

# events: willBeginDecelerating

`willBeginDecelerating` 在将要开始减速时回调：

```js
willBeginDecelerating: function(sender) {

}
```

# events: didEndDecelerating

`didEndDecelerating` 在结束减速时回调：

```js
didEndDecelerating: function(sender) {

}
```

# events: didEndScrollingAnimation

`didEndScrollingAnimation` 也是在结束减速时回调（不同的是他会在非人为触发的滚动时回调）：

```js
didEndScrollingAnimation: function(sender) {

}
```

# events: didScrollToTop

`didScrollToTop` 在滚动到顶部时回调：

```js
didScrollToTop: function(sender) {

}
```

# Auto Layout

要让 Auto Layout 对 scrollView 正常工作有些困难，我们推荐通过 `layoutSubviews` 来解决某些问题：

```js
const contentHeight = 1000;
$ui.render({
  views: [
    {
      type: "scroll",
      layout: $layout.fill,
      events: {
        layoutSubviews: sender => {
          $("container").frame = $rect(0, 0, sender.frame.width, contentHeight);
        }
      },
      views: [
        {
          type: "view",
          props: {
            id: "container"
          },
          views: [
            {
              type: "view",
              props: {
                bgcolor: $color("red")
              },
              layout: (make, view) => {
                make.left.top.right.equalTo(0);
                make.height.equalTo(300);
              }
            }
          ]
        }
      ]
    }
  ]
});
```

也即，向 scrollView 添加一个用于布局的子 view，该子 view 通过 layoutSubviews 设置 frame，然后在上面添加的 view 就可以使用 Auto Layout 了。了解更多：https://developer.apple.com/library/archive/technotes/tn2154/_index.html

---

## 文件内容解读与示例

### 组件用途

`scroll` 是一个基础的**容器类组件**。它的核心功能是允许其内部的子视图（内容）的尺寸可以超出 `scroll` 视图本身在屏幕上可见的**框架（frame）**，用户可以通过滑动、捏合等手势来查看所有内容。它是构建可滚动文章、图片查看器等界面的基础，也是 `list` 和 `matrix` 组件的底层依赖。

### 核心概念：`frame` 与 `contentSize`

要掌握 `scroll`，必须理解两个尺寸的区别，我们可以用一个“卷轴画”的比喻来理解：

- **`frame`**: `scroll` 视图本身的大小，即你在屏幕上能看到的那个“窗口”的大小。
- **`contentSize`**: `scroll` 视图内部所有内容展开后，实际占用的总尺寸。这相当于那幅“卷轴画”本身的完整大小。

**滚动的本质，就是移动“卷轴画”，透过“窗口”去看它的不同部分。**

要让 `scroll` 能够滚动，`contentSize` 必须至少在一个维度上大于 `frame` 的尺寸。你有两种方式来设定 `contentSize`：

1.  **隐式设置**：给 `scroll` 内部的子视图设置一个明确的、比 `scroll` 自身 `frame` 更大的尺寸。`scroll` 会自动计算出能包裹所有子视图的最小 `contentSize`。
2.  **显式设置**：直接设置 `scroll` 的 `props: { contentSize: $size(width, height) }`。

### 关键特性与用法

- **下拉刷新 (`pulled` 事件)**: 这是移动端非常常见的交互。当用户在内容顶部继续向下滑动时，会触发 `pulled` 事件。你可以在此事件中执行刷新数据的网络请求，请求完成后，务必调用 `sender.endRefreshing()` 来结束顶部的加载动画。

- **分页滚动 (`pagingEnabled`)**: 将此属性设置为 `true`，`scroll` 的滚动方式会从连续滚动变为“翻页”模式。它会按 `scroll` 视图 `frame` 的整数倍进行“吸附”式滚动，适合用于制作简单的引导页。

- **图片缩放 (`zoomEnabled`)**: 要实现图片的双指捏合缩放，必须使用 `scroll` 作为容器。将 `image` 放入 `scroll` 中，并设置 `scroll` 的 `props: { zoomEnabled: true }` 即可。

- **滚动监控 (`didScroll` 事件)**: 当视图滚动时，此事件会被持续触发。你可以从 `sender.contentOffset` 中获取当前的滚动偏移量，用于实现导航栏渐变、视差滚动等高级视觉效果。

### 示例代码：可滚动的长文章与下拉刷新

下面的示例将创建一个包含很长文本的滚动视图，并实现下拉刷新功能。

```javascript
const longText = "JSBox 是一个强大的工具...\n".repeat(50); // 重复50次以创建长文本

$ui.render({
  props: { title: "Scroll 组件示例" },
  views: [
    {
      type: "scroll",
      props: {
        id: "main-scroll"
      },
      layout: $layout.fill,
      views: [
        {
          type: "label",
          props: {
            text: longText,
            lines: 0, // 允许多行
            font: $font(18)
          },
          layout: (make, view) => {
            // 关键：让 label 的宽度与 scroll 视图一致，但高度由内容决定
            make.width.equalTo(view.super);
            make.top.left.inset(10);
          }
        }
      ],
      events: {
        pulled: (sender) => {
          // 响应下拉刷新事件
          $ui.toast("正在刷新...");
          // 模拟网络请求
          $delay(2, () => {
            $ui.toast("刷新完成");
            sender.endRefreshing(); // 结束刷新动画
          });
        }
      }
    }
  ]
});
```

**代码解读**：

1.  我们将一个包含大量文本的 `label` 放入 `scroll` 视图中。通过 Auto Layout，我们约束了 `label` 的宽度与 `scroll` 视图相同，而高度则由其内容（`lines: 0`）自由撑开。这样，`scroll` 的 `contentSize` 就会被隐式地设定为 `label` 的实际大小，从而实现了垂直滚动。
2.  我们在 `events` 中定义了 `pulled` 方法。当用户执行下拉刷新手势时，此方法被调用。
3.  在 `pulled` 方法内部，我们用 `$delay` 模拟了一个耗时 2 秒的异步操作。操作完成后，我们必须调用 `sender.endRefreshing()` 来通知 `scroll` 视图收起顶部的加载指示器。

`scroll` 组件是构建动态、可扩展内容区域的基础。理解 `frame` 和 `contentSize` 的关系，并善用其丰富的事件，是掌握它的关键。 
