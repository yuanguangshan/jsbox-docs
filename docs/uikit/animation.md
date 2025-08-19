> 动画是 iOS 的强项之一，JSBox 也对其进行了简单的支持，并且提供了两种支持的方式。

# UIView animation

最简单的方式是使用 `$ui.animate`，例如：

```js
$ui.animate({
  duration: 0.4,
  animation: function() {
    $("label").alpha = 0
  },
  completion: function() {
    $("label").remove()
  }
})
```

这段代码会把 label 的透明度在 0.4 秒的时间里变成 0，然后移除这个 view（completion 是可选的）。

当然这个方法也支持更多的属性让你来实现一个 `spring animation`:

属性 | 类型 | 说明
---|---|---
delay | number | 延迟执行秒数
damping | number | 阻尼大小
velocity | number | 初始速度
options | number | 选项[请参考](https://developer.apple.com/documentation/uikit/uiviewanimationoptions)

# 链式调用

JSBox 引入了一种链式调用的机制，基于 [JHChainableAnimations](https://github.com/jhurray/JHChainableAnimations) 这个库：

```js
$("label").animator.makeBackground($color("red")).easeIn.animate(0.5)
```

比如这行代码可以让 label 的背景色在 0.5 秒的时间里变成红色，同时指定了动画曲线为 `easeIn`。

是的，对 view 调用 `animator` 对象，会返回一个 animator 实例，可以用一个链式语句完成整个动画过程。

具体细节请参考 `JHChainableAnimations` 的文档：https://github.com/jhurray/JHChainableAnimations

> 鉴于该项目缺乏持续的维护，目前不推荐使用此方式实现动画

---

## 文件内容解读与示例

### 用途说明

本文档介绍了在 JSBox 中为 UI 控件添加**动画效果**的两种主要方法。动画能极大地提升用户体验，让界面过渡更自然、交互反馈更生动。JSBox 封装了 iOS 原生的 `UIView` 动画接口，并引入了一个链式动画库，让创建动画变得简单直观。

### 方法一：`$ui.animate` (推荐)

这是最常用、最直接的动画方式，它映射了 iOS 的 `UIView.animate` 方法。

-   **核心概念**: 你定义了一个“目标状态”，系统会自动为你创建从“当前状态”到“目标状态”的平滑过渡动画。
-   **工作方式**: 
    1.  在 `$ui.animate` 的 `animation` 函数内部，修改你想要产生动画效果的视图属性（如 `alpha`, `frame`, `bgcolor`, `transform` 等）。
    2.  JSBox 会捕捉到这些属性的变更，并在指定的 `duration` (持续时间) 内完成动画。
    3.  动画完成后，可选的 `completion` 函数会被调用，你可以在这里执行一些清理工作，比如移除视图。

-   **弹簧动画 (Spring Animation)**: 通过设置 `damping` (阻尼) 和 `velocity` (初始速度) 参数，可以轻松创建出富有物理效果的弹簧动画，让界面元素看起来更具活力。
    -   `damping`: 阻尼比，范围从 0 到 1。值越小，弹性（震动）效果越强。值为 1 时无弹性效果。
    -   `velocity`: 初始速度。一个正值会让动画开始时有一个“推”一下的加速效果。

#### 示例：弹簧动画

```javascript
// 创建一个蓝色方块
$ui.render({
  views: [
    {
      type: "view",
      props: {
        id: "box",
        bgcolor: $color("blue"),
        cornerRadius: 10
      },
      layout: (make, view) => {
        make.center.equalTo(view.super);
        make.size.equalTo($size(100, 100));
      },
      events: {
        tapped: (sender) => {
          // 点击时触发弹簧动画
          $ui.animate({
            duration: 0.8, // 动画时长
            damping: 0.4,  // 较小的阻尼，产生明显弹性
            velocity: 0.6, // 给予一个初始速度
            animation: () => {
              // 目标状态：放大1.5倍并旋转180度
              sender.transform = $transform({ 
                scale: 1.5, 
                rotation: Math.PI 
              });
            },
            completion: () => {
              // 动画结束后，恢复原状
              $ui.animate({
                duration: 0.4,
                animation: () => { sender.transform = $transform() }
              });
            }
          });
        }
      }
    }
  ]
});
```

### 方法二：链式动画 (`.animator`)

这种方式提供了一种声明式的、链式调用的语法来构建复杂的连续动画。

-   **核心概念**: 通过在一个视图的 `animator` 属性上连续调用方法，来构建一个动画序列，最后通过 `.animate()` 触发执行。
-   **优点**: 对于需要按顺序执行的多个动画步骤，代码可以写得非常紧凑和易读。
-   **缺点**: 文档明确指出，其依赖的底层库 `JHChainableAnimations` 已缺乏维护，因此**不推荐在新项目中使用此方法**，以免遇到潜在的 bug 或兼容性问题。

#### 示例：链式动画

```javascript
// 获取视图
const label = $("myLabel");

// 定义一个移动、变色再旋转的动画序列
label.animator
  .moveX(50)      // 向右移动50点
  .then(0.5)      // 等待0.5秒
  .makeBackground($color("red")) // 背景变红
  .rotate(360)    // 旋转360度
  .easeIn         // 使用 easeIn 动画曲线
  .animate(1.0);  // 以1秒的持续时间执行整个序列
```

### 总结

JSBox 提供了强大的动画能力。对于绝大多数场景，`$ui.animate` 都是最佳选择，它稳定、功能强大，并且支持创建富有表现力的弹簧动画。虽然链式动画在语法上很优雅，但考虑到其维护状态，应谨慎使用。优先掌握 `$ui.animate` 将能满足你几乎所有的动画需求。