> 布局指的是确定视图的大小和位置，以及它与其他视图之间的关系

# 布局系统

布局在用户界面的构建过程中是一个必须的步骤，不同的平台有自己不同的布局方案，比如 iOS 平台的 [Auto Layout](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/AutolayoutPG/index.html) 和 Web 领域的 [Flexbox](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout/Using_CSS_flexible_boxes)。

布局的目的简单说是：确定控件的位置和大小。

*PS: 这一章有点繁琐，可以暂且跳过以看看后面各种控件，不过理解本章是非常重要的。*

在早期的 iOS 里面，由于屏幕尺寸的局限性，iOS 主流的布局方案是 `frame` 布局，也就是基于绝对的位置和大小，在 JSBox 的控件体系里面你也可以这样干（在 `props` 里面指定 `frame`），但基于下列的原因最好不要这样做：

- 计算过程十分繁琐，描述能力较差
- 需要了解父 view 尺寸，无法动态的得到和更新自身尺寸
- 无法适应复杂多变的屏幕分辨率。

事实上 frame 布局也不是一无是处，他的性能会高于 Auto Layout，但在绝大部分场景下 Auto Layout 已经足够了。

通过 frame 布局的时候，你可以考虑使用 events->layoutSubview 来获得父 view 实时的 frame:

```js
$ui.render({
  views: [
    {
      type: "view",
      layout: $layout.fill,
      events: {
        layoutSubviews: function(view) {
          console.log(`frame: ${JSON.stringify(view.frame)}`)
        }
      }
    }
  ]
})
```

# 基本观念

本节试图用最简单的语言来简单介绍 Auto Layout 的基本观念：

**Auto Layout 的核心观念是在一个视图树里建立一系列的约束关系，用以描述控件与父控件、兄弟控件（有同一个父控件的所有控件）的关系。**

上面这句话有点绕口，他是理解基于约束的布局系统的关键之处，只要所有控件之间的约束是完备的，控件的位置和大小就是基于约束而自适应的，例如有一个按钮：

- 他的高度是 40
- 他距离父控件的相对位置是 (10, 10)
- 他距离父控件右边的距离是 10

在上面的描述中我们并没有指定他的宽度是多少，但是他的大小和位置都是相对父控件而言确定的，无论屏幕的尺寸是多少，我们总是可以得到一个与父控件有一点点边距并且高度是 40 的按钮：

```js
layout: function(make) {
  make.height.equalTo(40)
  make.left.top.right.inset(10)
}
```

当你需要使用 view 实例的时候，可以直接加上这个参数：

```js
layout: function(make, view) {
  make.height.equalTo(40)
  make.width.equalTo(view.height)
}
```

比如你可能会需要宽度等于高度，或者要取 `view.super`。

# 什么是约束

对某些 `属性` 进行 `关系` 描述就是约束，属性可以有上下左右，关系可以有大于小于等于，这是最简单的理解。

# 支持的属性

正如上面代码所示，`height` 属性表示高度，`left` 表示左边，这里介绍最基本最常用的几个，完整文档请参考 [Masonry](https://github.com/SnapKit/Masonry)。

属性 | 说明
---|---
width | 宽度
height | 高度
size | 大小
center | 中心
centerX | x 轴中心
centerY | y 轴中心
left | 左边
top | 上边
right | 右边
bottom | 下边
leading | 前边
trailing | 后边
edges | 四边

你会发现大部分时候 leading/trailing 做的事情和 left/right 看起来没什么不同，这涉及到以阿拉伯语为代表的 RTL (Right-to-left) 语言。简单说，leading 可以屏蔽左右差异，让你的布局在 RTL 语言下面也按他们的顺序显示。

# 支持的关系

关系用来描述属性之间的相对状况，例如 `equalTo`, `offset`，下面是支持的部分关系表：

关系 | 说明
---|---
equalTo(object) | 等于
greaterThanOrEqualTo(object) | 大于或等于
lessThanOrEqualTo(object) | 小于或等于
offset(number) | 偏移量
inset(number) | 边距
insets($insets) | 四边的边距
multipliedBy(number) | 乘以一个倍数
dividedBy(number) | 除以一个倍数
priority(number) | 约束优先级

将上面所说的内容串联起来，我们能得到一个链式语法来描述一个约束：

```js
layout: function(make) {
  make.left.equalTo($("label").right).offset(10)
  make.bottom.inset(0)
  make.size.equalTo($size(40, 40))
}
```

表示此 view 位于 `label` 右侧 10pt 的位置，并且位于父 view 底部 10pt 的位置。

要知道，掌握布局并不是一件容易的事情，各种布局方案都有自己的优点和缺陷，Auto Layout 的描述能力也是有限的（但是是够用的）。

然而，掌握基本的大小和上下左右/居中观念，已经足够应对大部分的场景，剩下的问题可以需要的时候查看文档：

[Masonry 文档](https://github.com/SnapKit/Masonry) [Apple 文档](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/AutolayoutPG/index.html)

这份文档还比较潦草，可以在后续的例子和 demo 扩展里面看到更多的用法。

# Flex

有时候你可能只需要简单的自动缩放，你可以使用视图的 `flex` 属性：

```js
$ui.render({
  views: [
    {
      type: "view",
      props: {
        bgcolor: $color("red"),
        frame: $rect(0, 0, 0, 100),
        flex: "W"
      }
    }
  ]
});
```

这将创建一个宽度等于父 view，高度为 100 的矩形。其中 flex 是一个字符串，可选字符为：

- `L`: [UIViewAutoresizingFlexibleLeftMargin](https://developer.apple.com/documentation/uikit/uiviewautoresizing/uiviewautoresizingflexibleleftmargin?language=objc)
- `W`: [UIViewAutoresizingFlexibleWidth](https://developer.apple.com/documentation/uikit/uiviewautoresizing/uiviewautoresizingflexiblewidth?language=objc)
- `R`: [UIViewAutoresizingFlexibleRightMargin](https://developer.apple.com/documentation/uikit/uiviewautoresizing/uiviewautoresizingflexiblerightmargin?language=objc)
- `T`: [UIViewAutoresizingFlexibleTopMargin](https://developer.apple.com/documentation/uikit/uiviewautoresizing/uiviewautoresizingflexibletopmargin?language=objc)
- `H`: [UIViewAutoresizingFlexibleHeight](https://developer.apple.com/documentation/uikit/uiviewautoresizing/uiviewautoresizingflexibleheight?language=objc)
- `B`: [UIViewAutoresizingFlexibleBottomMargin](https://developer.apple.com/documentation/uikit/uiviewautoresizing/uiviewautoresizingflexiblebottommargin?language=objc)

例如，使用 "LRTB" 表示 `UIViewAutoresizingFlexibleLeftMargin | UIViewAutoresizingFlexibleRightMargin | UIViewAutoresizingFlexibleTopMargin | UIViewAutoresizingFlexibleBottomMargin`。

---

## 文件内容解读与示例

### 用途说明

本文档是 JSBox **布局系统**的核心指南，重要性极高。布局决定了 UI 元素在屏幕上的位置和大小。本文重点介绍了 JSBox 所采用的、基于**约束（Constraint）的 Auto Layout** 系统，并与传统的 `frame` 布局进行了对比。

### 核心观念：约束 vs. Frame

-   **Frame 布局 (不推荐)**: 
    -   **是什么**: 直接指定一个视图的 `x`, `y`, `width`, `height` 绝对数值。
    -   **缺点**: 这是“写死”的布局。当屏幕尺寸变化（如横竖屏切换、不同设备）时，你需要手动重新计算所有视图的 frame，非常繁琐且容易出错。它不具备自适应能力。

-   **Auto Layout (推荐)**: 
    -   **是什么**: 不再关心绝对坐标，而是通过定义**视图之间的关系**来决定布局。这些关系就是“约束”。
    -   **核心思想**: “一个按钮的左边距离父视图左边 10pt”，“它的高度是 40pt”，“它的右边距离父视图右边 10pt”。通过这样一组完整的约束，系统就能在任何屏幕尺寸下自动计算出这个按钮应有的正确位置和宽度。
    -   **优点**: 具有强大的**自适应**能力，一次定义，处处适用。代码更具可读性，因为它描述的是布局的“意图”，而不是一堆魔法数字。

### JSBox 中的 Auto Layout 语法

JSBox 使用了一种基于 [Masonry](https://github.com/SnapKit/Masonry) 的链式 DSL（领域特定语言），让定义约束变得非常简洁和直观。

#### 布局函数

```javascript
layout: (make, view) => {
  // 在这里定义约束
}
```
- `make`: 约束的构建器，所有链式语法的起点。
- `view`: 正在布局的视图实例，可以访问其属性，如 `view.super` (父视图)。

#### 约束的组成：属性 + 关系

一条完整的约束由“视图的某个**属性**”和“另一个**关系**”组成。

**1. 常用属性 (Attribute)**
- **位置**: `left`, `right`, `top`, `bottom`, `leading`, `trailing`
- **尺寸**: `width`, `height`, `size` (同时设置宽高)
- **中心点**: `center`, `centerX`, `centerY`
- **边界**: `edges` (同时设置上、下、左、右四条边)

**2. 常用关系 (Relation)**
- `equalTo(target)`: 等于。`target` 可以是另一个视图的属性（如 `view.super.centerX`）、一个固定数值（如 `44`）或一个 `$size`/`$point` 对象。
- `inset(number)`: 内边距。`make.edges.inset(10)` 相当于同时设置上、下、左、右都距离父视图 10pt。
- `offset(number)`: 偏移量。通常用于在 `equalTo` 之后进行微调，如 `make.left.equalTo(label.right).offset(10)` (在 label 右边 10pt 处)。
- `multipliedBy(number)`: 乘以一个系数。如 `make.width.equalTo(view.super.width).multipliedBy(0.5)` (宽度是父视图的一半)。

### 示例：构建一个自适应卡片布局

这个例子将创建一个包含头像和用户名的卡片，无论屏幕多宽，它都能自适应。

```javascript
$ui.render({
  props: { title: "Auto Layout 示例" },
  views: [
    {
      type: "view",
      props: { id: "card", bgcolor: $color("lightGray"), cornerRadius: 10 },
      layout: (make, view) => {
        // 约束1: 卡片水平居中
        make.centerX.equalTo(view.super);
        // 约束2: 距离顶部 100pt
        make.top.equalTo(100);
        // 约束3: 左右两边距离屏幕边缘 20pt (这决定了卡片的宽度)
        make.left.right.inset(20);
        // 约束4: 高度为 80pt
        make.height.equalTo(80);
      },
      views: [
        {
          type: "image",
          props: { id: "avatar", src: "https://images.unsplash.com/photo-1527980965255-d3b416303d12?w=80&h=80&fit=crop", circular: true },
          layout: (make, view) => {
            // 约束1: 垂直居中于卡片
            make.centerY.equalTo(view.super);
            // 约束2: 距离卡片左边 15pt
            make.left.inset(15);
            // 约束3: 固定尺寸 50x50
            make.size.equalTo($size(50, 50));
          }
        },
        {
          type: "label",
          props: { id: "username", text: "John Doe", font: $font("bold", 18) },
          layout: (make, view) => {
            // 约束1: 垂直居中于卡片
            make.centerY.equalTo(view.super);
            // 约束2: 左边紧靠着头像的右边，并有 15pt 间距
            make.left.equalTo($("avatar").right).offset(15);
            // 约束3: 右边距离卡片右边 15pt (防止文字超长)
            make.right.inset(15);
          }
        }
      ]
    }
  ]
});
```

### 总结

掌握 Auto Layout 是 JSBox UI 开发从入门到进阶的必经之路。虽然初看起来比 `frame` 复杂，但它带来的灵活性、自适应能力和代码可维护性是无与伦比的。核心思想就是**通过描述关系来定义布局**。熟练运用 `left/right/top/bottom`, `center`, `width/height` 等基本属性和 `equalTo`, `inset`, `offset` 等关系，就能应对绝大多数布局需求。