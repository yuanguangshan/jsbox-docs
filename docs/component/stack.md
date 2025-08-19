# type: "stack"

这是一种特殊的视图，它本身并不渲染任何内容，而仅仅是用作对其子 view 自动布局。

请注意，JSBox 的 stack view 完全基于 `UIStackView`，所以为了更好地理解这一节的内容，请先阅读 Apple 的官方文档：https://developer.apple.com/documentation/uikit/uistackview

# 概览

请先看一个 Stack View 的简单样例：

```js
$ui.render({
  views: [
    {
      type: "stack",
      props: {
        spacing: 10,
        distribution: $stackViewDistribution.fillEqually,
        stack: {
          views: [
            {
              type: "label",
              props: {
                text: "Left",
                align: $align.center,
                bgcolor: $color("silver")
              }
            },
            {
              type: "label",
              props: {
                text: "Center",
                align: $align.center,
                bgcolor: $color("aqua")
              }
            },
            {
              type: "label",
              props: {
                text: "Right",
                align: $align.center,
                bgcolor: $color("lime")
              }
            }
          ]
        }
      },
      layout: $layout.fill
    }
  ]
});
```

如果你希望一些视图被 stack 自动布局，应该把它们放到 `stack` 里面，而不是直接放在 `views` 这个节点下面。

另外，你可以通过 `axis`, `distribution`, `alignment`, 和 `spacing` 等属性来控制布局策略。

# props: axis

`axis` 属性可以用来控制 stack 的布局方向，支持竖排或者横排，可选值：

```js
- $stackViewAxis.horizontal
- $stackViewAxis.vertical
```

# props: distribution

`distribution` 属性 stack 在 axis 方向上以何种策略分布，可选值：

```js
- $stackViewDistribution.fill
- $stackViewDistribution.fillEqually
- $stackViewDistribution.fillProportionally
- $stackViewDistribution.equalSpacing
- $stackViewDistribution.equalCentering
```

# props: alignment

`alignment` 属性决定了 stack 上的对齐方式，可选值：

```js
- $stackViewAlignment.fill
- $stackViewAlignment.leading
- $stackViewAlignment.top
- $stackViewAlignment.firstBaseline
- $stackViewAlignment.center
- $stackViewAlignment.trailing
- $stackViewAlignment.bottom
- $stackViewAlignment.lastBaseline
```

# props: spacing

`spacing` 属性决定了 stack 上各个子 view 之间的间距，应该填入一个数字，也可以使用下面这两个常量：

```js
- $stackViewSpacing.useDefault
- $stackViewSpacing.useSystem
```

# props: isBaselineRelative

`isBaselineRelative` 属性决定了垂直方向上的 spacing 是否基于 baseline，应该填入一个布尔值。

# props: isLayoutMarginsRelative

`isLayoutMarginsRelative` 属性决定了 stack 的布局策略是否基于 layoutMargins，应该填入一个布尔值。

# 动态改变一个视图

在视图初始化之后，你可以通过下面的方式动态改变他们：

```js
const stackView = $("stackView");
const views = stackView.stack.views;
views[0].hidden = true;

// This hides the first view in the stack
```

# 添加一个视图

除了在初始化的时候填入 `stack.views`，你还可以动态的添加一个视图到 stack：

```js
const stackView = $("stackView");
stackView.stack.add(newView);

// newView can be created with $ui.create
```

# 移除一个视图

可以通过这样的方式移除一个视图：

```js
const stackView = $("stackView");
stackView.stack.remove(existingView);

// existingView can be retrieved with id
```

# 插入一个视图

向 stack 插入一个视图到 index：

```js
const stackView = $("stackView");
stackView.stack.insert(newView, 2);
```

# 设置视图间距

Stack 视图会自动管理间距，你也可以为某个特定的视图自定义间距：

```js
const stackView = $("stackView");
stackView.stack.setSpacingAfterView(arrangedView, 20);

// arrangedView must be a view that is contained in the stack
```

# 获得视图间距

获得某个特定视图之后的间距：

```js
const stackView = $("stackView");
const spacing = stackView.stack.spacingAfterView(arrangedView);

// arrangedView must be a view that is contained in the stack
```

# 示例代码

相较于其他类型的视图，stack view 并不那么容易理解，所以我们构建了一个示例项目，方便让你理解上述的这些属性是如何工作的：https://github.com/cyanzhong/xTeko/blob/master/extension-demos/stack-view

---

## 文件内容解读与示例

### 组件用途

`stack`（堆栈）是一个不渲染自身、专门用于**自动布局**其内部子视图的强大容器。它极大地简化了将多个视图组织成水平行或垂直列的布局过程。你无需再为每个视图手动编写繁琐的 Auto Layout 约束，只需将它们放入 `stack` 中，并设置几个关键属性，即可实现整齐、自适应的布局。它类似于 Web 开发中的 Flexbox 或 SwiftUI 中的 `HStack`/`VStack`。

### 核心语法：独特的 `stack` 属性

使用 `stack` 组件时，有一个必须记住的、与其他容器都不同的语法规则：所有受其管理的子视图，都必须放在 `props` 内部一个名为 `stack` 的对象里的 `views` 数组中。

```javascript
{
  type: "stack",
  props: {
    // ... stack 的布局属性
    stack: { // 注意这个层级
      views: [
        // ... 这里是所有被 stack 管理的子视图
      ]
    }
  }
}
```

### 核心布局属性

- **`axis` (轴向)**: `stack` 的最基本属性，决定了子视图的排列方向。
  - `$stackViewAxis.vertical`: **垂直**排列，形成一列。
  - `$stackViewAxis.horizontal`: **水平**排列，形成一行。

- **`distribution` (分布)**: 决定了子视图在 `axis` 方向上如何被调整大小和放置。
  - `.fillEqually`: **等尺寸**填充，所有子视图在 `axis` 方向上获得相同的尺寸。
  - `.equalSpacing`: **等间距**分布，子视图保持自身大小，它们之间的间距被拉伸至相等。
  - `.fill`: 填充，`stack` 会尝试拉伸其中一个子视图来填满所有可用空间。

- **`alignment` (对齐)**: 决定了子视图在**垂直于 `axis` 的方向**上如何对齐。
  - 对于 `vertical` 轴向：控制**水平**对齐 (`.leading` 左, `.center` 中, `.trailing` 右)。
  - 对于 `horizontal` 轴向：控制**垂直**对齐 (`.top` 上, `.center` 中, `.bottom` 下)。
  - `.fill`: 子视图被拉伸以填满交叉轴方向的空间。

- **`spacing` (间距)**: 一个数字，定义了相邻子视图之间的固定间距。

### 示例代码：使用嵌套 `stack` 构建复杂布局

下面的示例将通过嵌套 `stack` 来创建一个包含头像、用户名、简介和统计信息的用户卡片，这在手动布局下会非常繁琐。

```javascript
$ui.render({
  props: { title: "Stack 组件示例" },
  views: [
    {
      // 最外层的垂直 stack，用于整体布局
      type: "stack",
      props: {
        axis: $stackViewAxis.vertical,
        alignment: $stackViewAlignment.fill,
        spacing: 15,
        bgcolor: $color("#F5F5F5"),
        radius: 10,
        stack: {
          views: [
            // --- 顶部用户信息区域 (水平 stack) ---
            {
              type: "stack",
              props: {
                axis: $stackViewAxis.horizontal,
                alignment: $stackViewAlignment.center,
                spacing: 10,
                stack: {
                  views: [
                    { type: "image", props: { symbol: "person.crop.circle.fill", tintColor: $color("gray"), frame: { width: 60, height: 60 } } },
                    {
                      // 用户名和简介的垂直 stack
                      type: "stack",
                      props: {
                        axis: $stackViewAxis.vertical,
                        alignment: $stackViewAlignment.leading,
                        stack: {
                          views: [
                            { type: "label", props: { text: "JSBox User", font: $font("bold", 18) } },
                            { type: "label", props: { text: "热爱编程，探索世界。", textColor: $color("gray") } }
                          ]
                        }
                      }
                    }
                  ]
                }
              }
            },
            // --- 底部统计数据区域 (水平 stack) ---
            {
              type: "stack",
              props: {
                axis: $stackViewAxis.horizontal,
                distribution: $stackViewDistribution.fillEqually,
                stack: {
                  views: [
                    { type: "label", props: { text: "108\n帖子", lines: 0, align: $align.center } },
                    { type: "label", props: { text: "4.2K\n关注者", lines: 0, align: $align.center } },
                    { type: "label", props: { text: "360\n关注中", lines: 0, align: $align.center } }
                  ]
                }
              }
            }
          ]
        }
      },
      layout: (make, view) => {
        make.center.equalTo(view.super);
        make.left.right.inset(20);
      }
    }
  ]
});
```

**代码解读**：

1.  我们使用了一个**垂直**的 `stack` 作为最外层的容器。
2.  容器的第一个子元素是另一个**水平** `stack`，用于并排摆放“头像”和“文字信息区”。
3.  在“文字信息区”内部，我们又用了一个**垂直** `stack` 来上下摆放“用户名”和“简介”。
4.  容器的第二个子元素是一个**水平** `stack`，我们设置了 `distribution: .fillEqually`，使得“帖子”、“关注者”、“关注中”这三个 `label` **平分**了所有水平空间，达到了整齐排列的效果。

这个例子充分展示了通过**嵌套**不同轴向和属性的 `stack`，可以轻松构建出清晰、自适应的复杂布局。 
