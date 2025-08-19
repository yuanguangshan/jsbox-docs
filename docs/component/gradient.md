# type: "gradient"

用于创建一个渐变图层：

```js
$ui.render({
  props: {
    bgcolor: $color("white")
  },
  views: [
    {
      type: "gradient",
      props: {
        colors: [$color("red"), $color("clear"), $color("blue")],
        locations: [0.0, 0.5, 1.0],
        startPoint: $point(0, 0),
        endPoint: $point(1, 1)
      },
      layout: function(make, view) {
        make.left.top.equalTo(0)
        make.size.equalTo($size(100, 100))
      }
    }
  ]
})
```

这个控件的实现与 iOS 的 CAGradientLayer 完全一致，请参考：https://developer.apple.com/documentation/quartzcore/cagradientlayer

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
colors | array | 读写 | 颜色
locations | array | 读写 | 控制点
startPoint | $point | 读写 | 起始点
endPoint | $point | 读写 | 终止点

---

## 文件内容解读与示例

### 组件用途

`gradient` 是一个纯粹的视觉展示组件，它唯一的用途就是创建一个平滑的颜色渐变。你可以用它来作为其他视图的背景，或者作为独立的装饰元素，为你的 UI 增添活力和现代感。它本身不响应任何用户交互。

### 核心属性详解

要创建一个渐变，你需要理解以下四个关键属性：

1.  **`colors`**: 一个由 `$color` 对象组成的数组。它定义了渐变中包含哪些颜色，以及它们的顺序。你至少需要提供两种颜色。

2.  **`locations`**: 一个由 0.0 到 1.0 之间的数字组成的数组，与 `colors` 数组一一对应。它精确地控制每种颜色出现的位置，“0.0”代表渐变的起点，“1.0”代表终点。如果省略此属性，颜色将会被均匀地分布。

3.  **`startPoint` 和 `endPoint`**: 这是控制渐变**方向**的关键。它们的值是 `$point` 对象，在一个“单位坐标系”中工作。在这个坐标系里，视图的左上角是 `(0, 0)`，右下角是 `(1, 1)`。渐变色会沿着从 `startPoint` 到 `endPoint` 的直线方向进行渲染。

    - **垂直渐变 (从上到下)**:
      `startPoint: $point(0.5, 0)`
      `endPoint: $point(0.5, 1)`

    - **水平渐变 (从左到右)**:
      `startPoint: $point(0, 0.5)`
      `endPoint: $point(1, 0.5)`

    - **对角线渐变 (左上到右下)**:
      `startPoint: $point(0, 0)`
      `endPoint: $point(1, 1)`

### 示例代码：创建全屏对角渐变背景

下面的示例将创建一个从左上角到右下角的对角线渐变，并将其作为整个页面的背景。

```javascript
$ui.render({
  views: [
    {
      // 1. 创建渐变视图作为背景
      type: "gradient",
      props: {
        // 定义三种颜色，从亮蓝到紫再到粉色
        colors: [
          $color("#4facfe"),
          $color("#9c52ff"),
          $color("#f06eaa")
        ],
        // locations 数组控制颜色节点的位置
        // 蓝色在起点，紫色在30%处，粉色在终点
        locations: [0.0, 0.3, 1.0],
        // 从左上角 (0, 0) 到右下角 (1, 1) 的对角线方向
        startPoint: $point(0, 0),
        endPoint: $point(1, 1)
      },
      layout: $layout.fill // 让渐变视图填充整个屏幕
    },
    {
      // 2. 在渐变背景上添加其他内容
      type: "label",
      props: {
        text: "Hello, Gradient!",
        font: $font("bold", 32),
        textColor: $color("white"),
        shadow: {
          radius: 3,
          offset: $point(1, 1),
          color: $color("black")
        }
      },
      layout: function(make, view) {
        // 让标签在屏幕中央
        make.center.equalTo(view.super);
      }
    }
  ]
});
```

**代码解读**：

1.  我们将 `gradient` 视图作为第一个子视图，并使用 `$layout.fill` 让它铺满整个屏幕，从而成为背景。
2.  `colors` 数组定义了渐变的色彩构成。
3.  `locations` 数组让颜色过渡不是均匀的，紫色节点被我们设置在了30%的位置，使得蓝色区域较小。
4.  `startPoint` 和 `endPoint` 的设置 `(0,0)` 到 `(1,1)` 是创建对角线渐变的标准方式。
5.  之后添加的 `label` 视图会自然地显示在渐变背景之上。

通过组合这四个属性，你可以创造出几乎任何你想要的线性渐变效果，这是美化 UI 的一个简单而有效的方法。 
