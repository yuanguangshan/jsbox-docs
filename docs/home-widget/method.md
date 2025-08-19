# 相关方法

我们在 `$widget` 模块上新增了一些适用于桌面小组件的方法和常量，方便使用。

# $widget.setTimeline(object)

为桌面小组件提供时间线：

```js
$widget.setTimeline({
  entries: [
    {
      date: new Date(),
      info: {}
    }
  ],
  policy: {
    atEnd: true
  },
  render: ctx => {
    return {
      type: "text",
      props: {
        text: "Hello, World!"
      }
    }
  }
});
```

详情请参考[时间线](home-widget/timeline.md)。

# $widget.reloadTimeline()

手动触发时间线的刷新，是否刷新由系统决定：

```js
$widget.reloadTimeline();
```

# $widget.inputValue

返回当前小组件设置的参数：

```js
const inputValue = $widget.inputValue;
```

> 在主应用运行时 `inputValue` 为空，请使用假数据作为测试目的

# $widget.family

返回当前小组件的尺寸，0 ~ 3 分别表示小、中、大、超大（iPadOS 15）：

```js
const family = $widget.family;
// 0, 1, 2, 3
```

绝大部分情况下，您应该依赖上述 `render` 函数中返回的 `ctx` 来获取 `family`。仅当您需要在调用 `setTimeline` 之前就获取才使用这个接口。

在主应用运行时，这个值默认返回 0，可以被测试代码覆盖：

```js
$widget.family = $widgetFamily.medium;
```

# $widget.displaySize

返回当前小组件的显示大小：

```js
const size = $widget.displaySize;
// size.width, size.height
```

绝大部分情况下，您应该依赖上述 `render` 函数中返回的 `ctx` 来获取 `displaySize`。仅当您需要在调用 `setTimeline` 之前就获取才使用这个接口。

在主应用运行时，这个值默认返回小尺寸的大小，测试代码可以通过覆盖 `family` 来模拟这个值：

```js
$widget.family = $widgetFamily.medium;
```

# $widget.isDarkMode

当前小组件是否运行在深色模式下：

```js
const isDarkMode = $widget.isDarkMode;
```

绝大部分情况下，您应该依赖上述 `render` 函数中返回的 `ctx` 来获取 `isDarkMode`。仅当您需要在调用 `setTimeline` 之前就获取才使用这个接口。

# $widget.alignment

返回在视图里面会用到的 `alignment` 常量：

```js
const alignment = $widget.alignment;
// center, leading, trailing, top, bottom
// topLeading, topTrailing, bottomLeading, bottomTrailing
```

也可以直接使用字符串常量，例如 "center", "leading"...

# $widget.horizontalAlignment

返回在视图里面会用到的 `horizontalAlignment` 常量：

```js
const horizontalAlignment = $widget.horizontalAlignment;
// leading, center, trailing
```

也可以直接使用字符串常量，例如 "leading", "center"...

# $widget.verticalAlignment

返回在视图里面会用到的 `verticalAlignment` 常量：

```js
const verticalAlignment = $widget.verticalAlignment;
// top, center, bottom
// firstTextBaseline, lastTextBaseline
```

也可以直接使用字符串常量，例如 "top", "center"...

# $widget.dateStyle

返回在使用时间设置 `text` 组件时会用到的 `dateStyle` 常量：

```js
const dateStyle = $widget.dateStyle;
// time, date, relative, offset, timer
```

也可以直接使用字符串常量，例如 "time", "date"...

# $env.widget

检查是否运行在桌面小组件环境：

```js
if ($app.env == $env.widget) {
  
}
```

---

## 文件内容解读与示例

### 用途说明

`$widget` API 是 JSBox 中用于**开发 iOS 14+ 桌面小组件**的核心模块。它提供了定义小组件内容、控制更新时机、获取配置参数以及响应用户交互的关键方法和属性。理解 `$widget` 是构建高效、美观且符合系统规范的小组件的基础。

### 核心方法：`$widget.setTimeline(options)`

这是小组件开发中**最重要、最核心**的方法。它告诉 iOS 系统小组件应该显示什么内容，以及在未来的哪些时间点应该更新。

-   **`entries`**: 一个 `TimelineEntry` 对象的数组。每个 `entry` 定义了一个 `date`（该快照应该显示的时间）和一个 `info` 对象（与该快照相关的数据）。
-   **`policy`**: 定义了系统在时间线结束后如何处理小组件的更新策略（例如 `atEnd: true` 表示在时间线结束后请求新的时间线）。
-   **`render(ctx)`**: 一个回调函数，它接收一个 `context` 对象 (`ctx`)，并返回小组件的**UI 视图层级**。`ctx` 对象包含了当前渲染上下文的关键信息，如 `ctx.family`（小组件尺寸）、`ctx.displaySize`（显示尺寸）和 `ctx.isDarkMode`（是否深色模式）。你在这里构建小组件的实际视觉内容。

### 手动刷新：`$widget.reloadTimeline()`

-   **用途**: 允许你的脚本**手动请求系统刷新小组件的时间线**。系统会根据其内部策略决定是否立即刷新。这在你的脚本数据发生重大变化（例如，用户完成了某个任务，或者从服务器获取了新数据）时非常有用，可以强制小组件更新。

### 获取配置与环境信息

-   **`$widget.inputValue`**: 获取用户在添加小组件时通过配置界面设置的字符串参数。这是实现小组件可配置性的关键，允许同一个脚本为多个小组件实例提供不同的内容。
-   **`$widget.family`**: 获取当前小组件的尺寸类型（例如 `$widgetFamily.small`, `$widgetFamily.medium`, `$widgetFamily.large`, `$widgetFamily.xLarge`，以及锁屏小组件的尺寸）。**推荐在 `setTimeline` 的 `render` 函数的 `ctx` 参数中获取 `ctx.family`，因为它更准确地反映了当前渲染的上下文。**
-   **`$widget.displaySize`**: 获取当前小组件的实际显示尺寸（以点为单位）。同样，`ctx.displaySize` 更推荐。
-   **`$widget.isDarkMode`**: 判断小组件是否运行在深色模式下。`ctx.isDarkMode` 更推荐。

### 布局与日期样式常量

-   **`$widget.alignment` / `$widget.horizontalAlignment` / `$widget.verticalAlignment`**: 这些常量用于在小组件的布局（如 `hstack`, `vstack`, `zstack`）中控制子视图的对齐方式。
-   **`$widget.dateStyle`**: 用于在文本视图中格式化日期和时间，例如显示为“时间”、“日期”、“相对时间”等。

### 示例代码：一个可配置的“Hello Widget”

下面的示例将创建一个简单的小组件，它会显示一个问候语，并根据用户配置的参数和当前小组件的尺寸进行调整。同时，在主应用中提供一个按钮来手动刷新小组件。

```javascript
// 小组件的渲染函数
function renderWidget(ctx) {
  const family = ctx.family; // 获取当前小组件尺寸
  const inputValue = $widget.inputValue || "世界"; // 获取配置参数，默认为“世界”
  const isDarkMode = ctx.isDarkMode; // 获取是否深色模式

  let greetingText = "";
  if (family === $widgetFamily.small) {
    greetingText = `你好, ${inputValue}!`;
  } else if (family === $widgetFamily.medium) {
    greetingText = `你好, ${inputValue}!\n来自 JSBox 中型小组件`;
  } else if (family === $widgetFamily.large) {
    greetingText = `你好, ${inputValue}!\n来自 JSBox 大型小组件\n当前时间: ${new Date().toLocaleTimeString()}`; 
  } else {
    greetingText = `你好, ${inputValue}!`;
  }

  return {
    type: "vstack",
    props: {
      alignment: $widget.horizontalAlignment.center,
      spacing: 5,
      bgcolor: isDarkMode ? $color("black") : $color("white"),
      cornerRadius: 15
    },
    views: [
      {
        type: "label",
        props: {
          text: greetingText,
          font: $font("bold", family === $widgetFamily.small ? 18 : 24),
          textColor: isDarkMode ? $color("white") : $color("black"),
          lines: 0,
          align: $align.center
        }
      },
      {
        type: "label",
        props: {
          text: `尺寸: ${family}`, // 显示当前尺寸类型
          font: $font(12),
          textColor: isDarkMode ? $color("lightGray") : $color("darkGray")
        }
      }
    ]
  };
}

// 小组件脚本的入口
if ($app.env === $env.widget) {
  // 如果在小组件环境中运行
  const now = new Date();
  const nextUpdate = new Date(now.getTime() + 60 * 1000); // 每分钟更新一次

  $widget.setTimeline({
    entries: [
      { date: now, info: {} }, // 当前时间线入口
      { date: nextUpdate, info: {} } // 下一个时间线入口
    ],
    policy: {
      atEnd: true // 在时间线结束后请求新的时间线
    },
    render: renderWidget // 使用上面定义的渲染函数
  });
} else {
  // 如果在主应用中运行，提供一个刷新按钮
  $ui.render({
    props: { title: "小组件管理" },
    views: [
      {
        type: "label",
        props: {
          text: "点击下方按钮手动刷新桌面小组件。\n(实际刷新由系统决定)",
          lines: 0,
          align: $align.center
        },
        layout: make => make.center.equalTo(view.super).left.right.inset(20)
      },
      {
        type: "button",
        props: { title: "刷新小组件" },
        layout: make => make.top.equalTo(view.prev.bottom).offset(20).centerX.equalTo(view.super),
        events: {
          tapped: () => {
            $widget.reloadTimeline();
            $ui.toast("已请求刷新小组件！");
          }
        }
      }
    ]
  });
}
```