# 锁屏小组件 (Beta)

在 iOS 16 Beta 中，锁屏界面也可以有小组件，JSBox 也支持构建它们（目前在 TestFlight 版本）。

## 构建锁屏小组件

锁屏小组件本质上就是您熟悉的桌面小组件，有两个主要区别。

### 明亮的颜色

在锁屏状态下，小组件是用明亮的颜色而不是全色来呈现的。

您应该使用没有透明度的颜色，如纯白或纯黑来构建锁屏小组件。半透明的颜色将与墙纸混合，结果将无法阅读。

### 更小的尺寸

与桌面小组件相比，锁屏小组件有三种新的尺寸：

```js
const $widgetFamily = {
  // small, medium, large, xLarge
  accessoryCircular: 5,
  accessoryRectangular: 6,
  accessoryInline: 7,
}
```

- `accessoryCircular`: 1 * 1 圆形
- `accessoryRectangular`: 1 * 2 长方形
- `accessoryInline`: 在日期后面的一行信息

要检测当前的运行环境，请参考[这里](home-widget/timeline.md?id=render)。

## 应用内预览

为小组件提供的应用内预览目前不支持锁屏小组件的预览，请继续关注即将到来的更新。

## 样例代码

请参考这个 [GitHub 仓库](https://github.com/cyanzhong/jsbox-widgets) 以了解更多。

---

## 文件内容解读与示例

### 用途说明：iOS 16+ 锁屏小组件

本文档介绍了 iOS 16 及更高版本中引入的**锁屏小组件（Lock Screen Widgets）**。它们是桌面小组件的延伸，允许你的脚本在用户不解锁设备的情况下，直接在锁屏界面上显示信息。这为信息展示提供了更直接、更便捷的入口。

### 核心概念：桌面小组件的变体

锁屏小组件在本质上与桌面小组件是相同的（都基于“快照”和“时间线”机制），但它们在视觉呈现和尺寸上存在关键差异，这些差异是开发时需要特别注意的。

#### 1. 颜色呈现：明亮与不透明

-   **问题**: 锁屏小组件会直接叠加在用户的壁纸之上。如果使用半透明或复杂的颜色，可能会导致小组件的文本和图标与壁纸混淆，难以阅读。
-   **解决方案**: 强烈建议使用**明亮、高对比度且不透明**的颜色（如纯白、纯黑、纯色）来构建锁屏小组件。避免使用任何半透明的颜色，以确保在各种壁纸背景下的可读性。

#### 2. 尺寸：更小、更紧凑

锁屏小组件引入了三种新的、更紧凑的尺寸，这些尺寸通过 `$widgetFamily` 常量来表示：

-   **`$widgetFamily.accessoryCircular`**: 1x1 的圆形小组件，通常用于显示简单的图标或数字。
-   **`$widgetFamily.accessoryRectangular`**: 1x2 的长方形小组件，提供更多空间显示文本或少量数据。
-   **`$widgetFamily.accessoryInline`**: 一种特殊的内联小组件，它显示在锁屏日期和时间之后的一行文本中，非常适合显示简短的状态信息。

你可以通过 `$widget.family` 属性来检测当前小组件的尺寸类型，从而为不同尺寸提供不同的布局和内容。

### 示例代码：一个适应锁屏的电池电量小组件

下面的示例将创建一个电池电量小组件。它会根据小组件的尺寸（圆形锁屏小组件或普通桌面小组件）来调整其显示方式。

```javascript
// 获取电池电量信息
const batteryLevel = Math.round($device.info.battery.level * 100);
const batteryState = $device.info.battery.state; // 0:未知, 1:正常, 2:充电中, 3:充满

// 根据电池状态选择图标
let batterySymbol = "battery.0";
if (batteryLevel > 75) batterySymbol = "battery.100";
else if (batteryLevel > 50) batterySymbol = "battery.75";
else if (batteryLevel > 25) batterySymbol = "battery.50";
else if (batteryLevel > 0) batterySymbol = "battery.25";

if (batteryState === 2 || batteryState === 3) {
  batterySymbol = "battery.charging"; // 充电中图标
}

$widget.setTimeline(ctx => {
  let widgetView;

  // 根据小组件尺寸 family 来调整布局
  if (ctx.family === $widgetFamily.accessoryCircular) {
    // 锁屏圆形小组件布局
    widgetView = {
      type: "zstack",
      views: [
        {
          type: "view",
          props: {
            bgcolor: $color("white"), // 锁屏小组件推荐使用不透明亮色
            cornerRadius: 999 // 圆形
          },
          layout: $layout.fill
        },
        {
          type: "image",
          props: {
            symbol: batterySymbol,
            tintColor: $color("black") // 锁屏小组件推荐使用高对比度颜色
          },
          layout: make => {
            make.center.equalTo(view.super);
            make.size.equalTo($size(30, 30));
          }
        },
        {
          type: "label",
          props: {
            text: `${batteryLevel}%`,
            font: $font("bold", 10),
            textColor: $color("black")
          },
          layout: make => {
            make.centerX.equalTo(view.super);
            make.bottom.inset(5);
          }
        }
      ]
    };
  } else if (ctx.family === $widgetFamily.accessoryRectangular) {
    // 锁屏长方形小组件布局
    widgetView = {
      type: "hstack",
      props: {
        alignment: $widget.verticalAlignment.center,
        spacing: 5,
        bgcolor: $color("white"),
        cornerRadius: 10
      },
      views: [
        {
          type: "image",
          props: {
            symbol: batterySymbol,
            tintColor: $color("black")
          },
          layout: make => {
            make.left.inset(10);
            make.size.equalTo($size(30, 30));
          }
        },
        {
          type: "label",
          props: {
            text: `电量: ${batteryLevel}%`,
            font: $font("bold", 16),
            textColor: $color("black")
          },
          layout: $layout.fill
        }
      ]
    };
  } else if (ctx.family === $widgetFamily.accessoryInline) {
    // 锁屏内联小组件布局
    widgetView = {
      type: "text",
      props: {
        text: `🔋 ${batteryLevel}%`,
        font: $font(14),
        textColor: $color("white") // 内联小组件通常是白色文本
      }
    };
  } else {
    // 桌面小组件布局 (small, medium, large, xLarge)
    widgetView = {
      type: "vstack",
      props: {
        alignment: $widget.horizontalAlignment.center,
        bgcolor: $color("systemBackground")
      },
      views: [
        {
          type: "image",
          props: {
            symbol: batterySymbol,
            tintColor: $color("tintColor")
          },
          layout: make => {
            make.size.equalTo($size(50, 50));
            make.top.inset(10);
          }
        },
        {
          type: "label",
          props: {
            text: `电池电量: ${batteryLevel}%`,
            font: $font("bold", 20)
          },
          layout: make => {
            make.top.equalTo(view.prev.bottom).offset(5);
          }
        }
      ]
    };
  }

  // 设置时间线，这里简化为每 5 分钟更新一次
  const now = new Date();
  const nextUpdate = new Date(now.getTime() + 5 * 60 * 1000);

  $widget.setTimeline({
    entries: [
      { date: now, view: widgetView },
      { date: nextUpdate, view: widgetView } // 假设视图内容不变，只更新时间
    ]
  });
});
```

**代码解读**：

1.  脚本首先获取设备的电池电量和状态，并根据电量选择合适的 SF Symbol 图标。
2.  **核心逻辑**在 `ctx => { ... }` 函数内部，通过 `ctx.family` 判断当前小组件的尺寸类型。
3.  根据不同的 `family`，我们返回了完全不同的视图结构：
    -   `accessoryCircular` 采用了 `zstack` 来实现圆形背景和电量百分比的叠加。
    -   `accessoryRectangular` 采用了 `hstack` 来并排显示图标和文本。
    -   `accessoryInline` 直接返回一个 `text` 视图。
    -   其他桌面小组件尺寸则使用传统的 `vstack` 布局。
4.  **颜色选择**：对于锁屏小组件，我们特意使用了纯白或纯黑的背景和高对比度的文本颜色，以确保在锁屏壁纸上的可读性。
5.  最后，通过 `$widget.setTimeline` 设置了小组件的时间线，告知系统何时更新。

开发锁屏小组件需要特别关注其独特的尺寸和颜色要求，并利用 `$widget.family` 进行布局适配，以提供最佳的用户体验。 
