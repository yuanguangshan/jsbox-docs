# 桌面小组件

Apple 在 iOS 14 引入了桌面小组件，JSBox `v2.12.0` 提供了全面支持。与此同时，通知中心小组件的支持也进入过时状态，会在未来的 iOS 版本中被 Apple 移除。

桌面小组件相较通知中心小组件有很大的不同，所以我们有必要花点时间来理解一些基本概念。

# 核心概念

桌面小组件的本质是基于时间线的一系列**快照**，而不是动态构建的界面。所以当用户看到小组件时，并不会直接运行小组件所包含的代码。相反，系统会在一些时机（时间线机制，之后会详细说明）调用小组件的代码，代码可以生成一个快照提供给系统。然后系统会在合适的时间展示这些快照，并通过一些策略重新获取一系列新的快照。

此外，桌面小组件仅能有限地处理用户交互：

- 2 * 2 布局仅支持点击整个小组件
- 2 * 4 和 4 * 4 布局支持点击某个控件
- 点击会打开主应用，并携带一个 URL 给主应用处理

这些系统限制决定了桌面小组件的角色更多是**快速展示信息**，并将复杂的交互和任务代理到主应用去执行。

# 配置小组件

JSBox 支持小组件的全部尺寸，添加步骤：

- 在桌面长按后点击左上角的加号
- 找到 JSBox 然后将某个小组件拖拽到桌面
- 在编辑模式选择“编辑小组件”
- 选择某个支持小组件的脚本

相比于通知中心小组件只能运行一个脚本（尽管 JSBox 提供了三个），桌面小组件可以通过上面的配置方式创建出任意多个实例，并且可以通过系统提供的“堆叠”功能将多个小组件叠放在一起，分别展示不同的内容。

具体使用方法，请参照 Apple 或网上提供的教程了解更多。

# 小组件参数

对于每个小组件，您可以设置要运行的脚本，以及提供更多信息的附加参数。

该参数是一个字符串，可以像这样获取：

```js
const inputValue = $widget.inputValue;
```

对于安装包格式的脚本，小组件参数可以由脚本配置文件提供，例如：

```json
[
  {
    "name": "Option 1",
    "value": "Value 1"
  },
  {
    "name": "Option 2",
    "value": "Value 2"
  }
]
```

其中 `name` 是显示给用户选择的标题，`value` 是上述接口实际会取得的值。将配置内容放置为脚本安装包根目录下的 `widget-options.json` 文件即可。

# 样例代码

为了让您可以更好地上手小组件的开发，我们创建了一些样例项目以供参考：

- [WidgetDoodles](https://github.com/cyanzhong/jsbox-widgets/tree/master/WidgetDoodles)
- [AppleDevNews2](https://github.com/cyanzhong/jsbox-widgets/tree/master/AppleDevNews2)
- [QRCode](https://github.com/cyanzhong/jsbox-widgets/blob/master/QRCode.js)
- [xkcd](https://github.com/cyanzhong/jsbox-widgets/blob/master/xkcd.js)
- [clock](https://github.com/cyanzhong/jsbox-widgets/blob/master/clock.js)

我们会在之后完善这个仓库，以提供更多例子。

---

## 文件内容解读与示例

### 用途说明：iOS 14+ 桌面小组件

本文档是 JSBox 中关于 **iOS 14 及更高版本桌面小组件（Home Screen Widgets）**的入门介绍。它解释了桌面小组件与旧版通知中心小组件的根本区别，并阐明了其核心工作原理。桌面小组件是 iOS 生态系统中一个重要的信息展示入口，允许你的脚本在用户的主屏幕上提供动态、一目了然的信息。

### 核心概念：快照与时间线

理解桌面小组件的关键在于其“快照”和“时间线”的本质：

1.  **快照 (Snapshots)**: 桌面小组件并非一个实时运行的迷你应用。相反，它们是系统在特定时刻生成的**静态图片**（即“快照”）。用户在主屏幕上看到的小组件，实际上就是这些预先渲染好的快照。

2.  **时间线 (Timelines)**: 你的小组件代码不会持续运行。系统会根据你提供的一个“时间线”（`Timeline`），在预设的时间点调用你的脚本，让脚本生成新的快照。这个时间线可以包含一系列未来的日期和时间，以及每个时间点对应的视图数据。系统会根据这些时间线条目，在合适的时候更新小组件的显示。

3.  **有限的交互**: 
    -   小组件的交互非常有限，主要表现为**点击**。
    -   无论点击小组件的哪个部分，最终都会**打开主应用**（JSBox），并且可以携带一个 URL 参数，让主应用处理后续的复杂逻辑。
    -   小尺寸（2x2）小组件只能点击整个区域，而大尺寸（2x4, 4x4）小组件可以为内部的特定控件设置点击区域。

这些限制决定了小组件的主要角色是**快速展示信息**，而将复杂的交互和任务委托给主应用来完成。

### 小组件配置与输入

-   **添加与配置**: 用户可以通过长按桌面 -> 点击左上角“+”号 -> 找到 JSBox -> 拖拽小组件到桌面 -> 编辑小组件，来选择要运行的脚本，并为其提供额外的配置参数。
-   **`$widget.inputValue`**: 你的小组件脚本可以通过 `$widget.inputValue` 来获取用户在配置小组件时输入的字符串参数。这使得同一个脚本可以根据不同的配置，在多个小组件实例中显示不同的内容。
-   **`widget-options.json`**: 如果你的脚本安装包根目录下存在 `widget-options.json` 文件，你可以在其中定义一组预设的选项（`name` 和 `value`），用户在配置小组件时就可以从这些选项中选择，而不是手动输入字符串，这大大提升了用户体验。

### 概念示例：一个每日格言小组件

让我们构思一个“每日格言”小组件。它每天显示一句不同的格言，并且可以配置显示特定主题的格言。

1.  **`widget-options.json` (可选)**:
    ```json
    [
      { "name": "励志格言", "value": "inspirational" },
      { "name": "编程格言", "value": "coding" }
    ]
    ```

2.  **小组件脚本 (`daily_quote.js`)**:
    ```javascript
    // 获取用户配置的格言主题
    const theme = $widget.inputValue || "general"; // 默认为通用主题

    // 根据主题获取今天的格言
    function getTodayQuote(theme) {
      const quotes = {
        inspirational: [
          "相信自己，你比你想象的更强大。",
          "成功是坚持不懈的努力。"
        ],
        coding: [
          "代码是写给人看的，附带能在机器上运行。",
          "调试是把虫子从代码中移除，而不是把它们放进去。"
        ],
        general: [
          "生活就像一盒巧克力，你永远不知道下一颗是什么滋味。"
        ]
      };
      const today = new Date().getDate();
      const themeQuotes = quotes[theme] || quotes.general;
      return themeQuotes[today % themeQuotes.length];
    }

    const currentQuote = getTodayQuote(theme);

    // 渲染小组件视图
    $ui.render({
      views: [
        {
          type: "label",
          props: {
            text: currentQuote,
            font: $font(16),
            align: $align.center,
            lines: 0
          },
          layout: $layout.fill
        }
      ]
    });

    // 设置时间线，告诉系统何时更新小组件
    // 假设我们希望每天更新一次
    const now = new Date();
    const tomorrow = new Date(now.getFullYear(), now.getMonth(), now.getDate() + 1);

    $widget.setTimeline({
      entries: [
        { date: now, text: currentQuote }, // 当前快照
        { date: tomorrow, text: getTodayQuote(theme) } // 明天的快照
      ]
    });

    // 处理小组件点击事件 (点击会打开主应用)
    $widget.set and $widget.get // 假设点击后打开主应用并传递参数
    $widget.setEvent({
      onClick: () => {
        $app.openURL(`jsbox://run?name=daily_quote&action=show_full_quote&theme=${theme}`);
      }
    });
    ```

**代码解读**：

1.  脚本通过 `$widget.inputValue` 获取用户在配置时选择的主题。
2.  `getTodayQuote` 函数根据主题和日期返回今天的格言。
3.  `$ui.render` 定义了小组件的 UI 布局，这里是一个简单的 `label`。
4.  **`$widget.setTimeline` 是核心**。它告诉系统，当前小组件的快照是什么，以及下一个快照应该在何时生成。系统会根据这个时间线来调度脚本的运行和界面的更新。
5.  `$widget.setEvent` 定义了小组件被点击时的行为，这里是打开 JSBox 主应用并传递参数。

桌面小组件是 iOS 14+ 的重要特性，它让你的脚本能够以一种全新的方式呈现在用户面前。理解其“快照”和“时间线”的机制，是开发高效小组件的关键。 
