# type: "blur"

`blur` 用来创建一个模糊效果：

```js
{
  type: "blur",
  props: {
    style: 1 // 0 ~ 20
  },
  layout: $layout.fill
}
```

`style` 0 ~ 20 表示了不同的模糊效果，[参考](data/constant.md?id=blurstyle)。

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
style | $blurStyle | w | 效果类型

---

## 文件内容解读与示例

### 组件用途

`blur` 组件用于在界面上创建一个“毛玻璃”或“模糊”效果的视图。这是一种在现代 UI 设计中非常流行的视觉元素，常见于 iOS 的控制中心、通知中心等处。它并非一个不透明的色块，而是会实时地模糊其下方的所有内容，从而产生一种层次感和通透感，常用于需要突出前景内容但又不想完全遮挡背景的场景。

### 核心属性 `style`

`blur` 组件最重要的属性是 `style`。这个属性的值不是一个简单的模糊半径，而是一个预设的样式枚举 `$blurStyle`。这些样式与 iOS 系统的原生模糊效果完全对应，能够自适应浅色模式和深色模式。

- **`$blurStyle`**: 你可以把它理解为一个样式列表，例如 `$blurStyle.light`（明亮样式）、`$blurStyle.dark`（暗黑样式）、`$blurStyle.prominent`（突出样式）等。通过选择不同的样式，你可以获得不同观感的模糊效果。具体可用的样式常量可以查阅文档中给出的链接。

### 示例代码：创建“磨砂玻璃”信息卡

下面的示例将演示一个常见用法：在一张背景图片上放置一个模糊视图，并在模糊视图上添加文字，形成一个“磨砂玻璃”卡片的效果。

```javascript
$ui.render({
  props: {
    title: "Blur 组件示例"
  },
  views: [
    {
      // 首先，放置一个背景图片，让模糊效果有内容可以作用
      type: "image",
      props: {
        src: "https://images.unsplash.com/photo-1579546929518-9e396f3cc809?ixlib=rb-1.2.1&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=1080&fit=max",
        contentMode: $contentMode.scaleAspectFill
      },
      layout: $layout.fill // 填充整个界面
    },
    {
      // 接着，在图片上层添加 blur 视图
      type: "blur",
      props: {
        // 选用一个明亮的模糊效果
        // 你可以尝试 $blurStyle.dark, $blurStyle.extraLight 等不同值
        style: $blurStyle.light,
        // 为了让效果更美观，给它加上圆角
        cornerRadius: 15
      },
      layout: function(make, view) {
        // 让它在父视图中居中，尺寸为 200x100
        make.center.equalTo(view.super);
        make.size.equalTo($size(200, 100));
      },
      views: [
        {
          // 在 blur 视图内部添加一个标签
          type: "label",
          props: {
            text: "Frosted Glass",
            font: $font("bold", 22),
            textColor: $color("primaryText")
          },
          layout: function(make, view) {
            // 让标签在 blur 视图中居中
            make.center.equalTo(view.super);
          }
        }
      ]
    }
  ]
});
```

**代码解读**：

1.  我们先放置了一个 `image` 视图作为背景，这是让 `blur` 效果能够生效的前提。
2.  然后，我们在图片之上添加了 `blur` 视图。通过 `layout` 函数，我们将其定位在屏幕中央，并设置了固定大小和圆角。
3.  最关键的是，我们在 `blur` 视图的 `views` 数组内部又添加了一个 `label`。这说明 `blur` 视图自身也可以作为一个容器，承载其他子视图。这些子视图会清晰地显示在模糊背景之上。

通过这个例子，你可以直观地感受到 `blur` 组件如何为你的 UI 增添精致的层次感。 
