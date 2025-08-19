# $widget

$widget 接口提供对 Widget 的相关操作。

# $widget.height

获取或更改 widget 的高度：

```js
// Get
const height = $widget.height;
// Set
$widget.height = 400;
```

# $widget.mode

获取 widget 当前状态（展开/折叠）：

```js
const mode = $widget.mode; // 0: 折叠 1: 展开
```

# $widget.modeChanged

观察 widget 折叠展开状态的变化：

```js
$widget.modeChanged = mode => {
  
}
```

---

## 文件内容解读与示例

### 用途说明

本文档介绍了专用于**今日视图小组件（Today Widget）**的全局对象 `$widget`。这个对象提供了一系列独特的 API，用于获取和控制 Widget 的**特殊属性**，主要是其**尺寸和显示模式**。这些是在标准 JSBox 脚本中不存在的，因为它们是 Widget 这种特定展现形式所独有的。

### 核心概念：可变高度与显示模式

与固定大小的应用界面不同，iOS 的 Widget 具有两种显示状态：

1.  **折叠模式 (Compact Mode)**: Widget 初始显示时的紧凑状态，高度有系统限制，只用于展示最关键的信息。
2.  **展开模式 (Expanded Mode)**: 用户点击“显示更多”后，Widget 可以展开以显示更多内容。在此模式下，你可以通过代码请求一个更大的高度。

`$widget` 对象的核心就是围绕这两种模式进行管理。

### API 详解

-   **`$widget.height`**
    -   **作用**: 获取或**设置** Widget 的高度。
    -   **读取**: `const currentHeight = $widget.height;` 可以获取到 Widget 当前的实际高度。
    -   **写入**: `$widget.height = 400;` **这是最重要的一个功能**。你可以在代码中动态地改变 Widget 的高度，以适应不同内容的显示需求。例如，当列表加载了更多数据后，你可以增加 Widget 的高度以完整显示它们。
    -   **注意**: 设置的高度受限于 iOS 对 Widget 的最大高度限制。设置一个超大的值并不会无限增高。

-   **`$widget.mode`**
    -   **作用**: **只读**属性，用于获取 Widget 当前的显示模式。
    -   **返回值**: 
        -   `0`: 折叠模式 (Compact)
        -   `1`: 展开模式 (Expanded)
    -   **用途**: 你可以根据这个值来决定显示哪些 UI 元素。例如，在折叠模式下只显示一个标题，在展开模式下才显示完整的列表。

-   **`$widget.modeChanged`**
    -   **作用**: 这是一个**回调函数**，当用户点击“显示更多”或“显示更少”导致 Widget 的显示模式发生变化时，这个函数会被调用。
    -   **参数**: 回调函数接收一个新的 `mode` 值（`0` 或 `1`）作为参数。
    -   **用途**: 这是实现动态布局的关键。你应该在这个回调函数中，根据新的 `mode` 值，重新组织你的 UI，并可能通过设置 `$widget.height` 来调整 Widget 的高度。

### 示例：一个可展开的新闻列表 Widget

这个例子将创建一个 Widget，默认折叠时只显示 2 条新闻标题，展开后显示 5 条，并相应地调整自身高度。

```javascript
const newsHeadlines = [
  "Apple 发布新款 MacBook Pro",
  "JSBox 更新，支持全新 Widget API",
  "科学家发现新的系外行星",
  "本地足球队赢得冠军",
  "周末天气预报：晴朗"
];

// 渲染主界面
$ui.render({
  props: {
    title: "今日新闻"
  },
  views: [
    {
      type: "list",
      props: {
        id: "newsList",
        data: [] // 初始为空
      },
      layout: $layout.fill
    }
  ]
});

// 根据模式更新 UI 的函数
function updateUIByMode(mode) {
  const list = $("newsList");
  if (mode === 0) { // 折叠模式
    console.log("切换到折叠模式");
    list.data = newsHeadlines.slice(0, 2); // 只显示前2条
    $widget.height = 110; // 设置一个适合2行的高度
  } else { // 展开模式
    console.log("切换到展开模式");
    list.data = newsHeadlines; // 显示全部5条
    $widget.height = 220; // 设置一个适合5行的高度
  }
}

// 监听模式变化
$widget.modeChanged = (mode) => {
  updateUIByMode(mode);
};

// 初始化：使用当前模式渲染初始状态
updateUIByMode($widget.mode);
```

**如何使用**: 
1. 将此脚本添加到 Widget。
2. 在今日视图中，初始会看到一个较矮的 Widget，显示 2 条新闻。
3. 点击右上角的“显示更多”箭头。
4. `modeChanged` 事件触发，Widget 会变高，并显示全部 5 条新闻。

### 总结

`$widget` 对象是开发动态、交互式 Widget 的核心。通过读取 `$widget.mode`、响应 `$widget.modeChanged` 事件以及设置 `$widget.height`，你可以构建出能够根据用户操作和内容变化而智能调整布局和尺寸的、体验优秀的 Widget。