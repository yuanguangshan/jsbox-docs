# 时间线

正如我们前文提到，桌面小组件是基于时间的一系列**快照**。时间线是整个小组件工作方式的基石，我们会花一点时间解释这个概念。不过在此之前，请先阅读 Apple 提供的一篇文章：https://developer.apple.com/documentation/widgetkit/keeping-a-widget-up-to-date

这篇文章对理解时间线的工作原理极为重要，尤其是下面这个图：

<img src='https://docs-assets.developer.apple.com/published/2971813b6a098a34d134a04e38a50b83/2550/WidgetKit-Timeline-At-End@2x.png' width=360px/>

我们的代码扮演的是图中 Timeline Provider 的角色，系统在合适的时候向我们获取一个时间线和更新策略。然后系统会在合适的时间调用我们提供的方法显示小组件，并在策略满足的时候进行下一次获取。

所以，准确的时间线和合理的更新策略是小组件体验的保障。例如，天气应用可以知道接下来几个小时的天气状况，所以可以一次提供多个快照给系统，并在所有快照显示完成之后进行下一轮更新。

# $widget.setTimeline(object)

JSBox 使用 `$widget.setTimeline(...)` 函数来提供上述的时间线，使用样例：

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

在 `setTimeline` 之前，我们可以进行一些数据获取的操作，例如请求网络或地理位置。但请注意，小组件必须在很短的时间内完成对时间线的提供，以免因为性能问题而无法完成。

## entries

指定快照相应的时间和额外的上下文信息，在时间线可以预测的情况下，可以一次性提供多个给系统。

在一个 `entry` 里面，用 `date` 指定该快照显示的时间，用 `info` 携带上下文信息，可以在 `render` 函数中通过 `ctx` 取出。

> 如未提供 entries，JSBox 将以当前的时间生成一个默认的 entry。

## policy

指定系统下次获取时间线的策略，有以下几种方式：

```js
$widget.setTimeline({
  policy: {
    atEnd: true
  }
});
```

该方式在最后一个 entry 被使用之后，进行一次新的时间线获取。

```js
$widget.setTimeline({
  policy: {
    afterDate: aDate
  }
});
```

该方式在时间到达 `afterDate` 之后，进行一次新的时间线获取。

```js
$widget.setTimeline({
  policy: {
    never: true
  }
});
```

该方式表示时间线为静态内容，不会周期性地更新。

上述策略仅仅是给系统一些“建议”，系统并不保证刷新一定会完成。为了避免系统过滤掉太过频繁的刷新，请设计有节制的策略以确保体验。

> 在不提供 `entries` 和 `policy` 的状况下，JSBox 为每个脚本提供了每小时刷新一次的默认实现。

## render

在到达每个 entry 指定的时间时，系统将调用上述的 `render` 函数，我们的代码在这里返回一个 JSON 数据作为视图的描述：

```js
$widget.setTimeline({
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

上述代码将在小组件上显示 "Hello, World!"，关于描述视图的语法，我们将在后面详细介绍。

`ctx` 包含了上下文，可以获取到包括 entry 在内的一些环境信息：

```js
$widget.setTimeline({
  render: ctx => {
    const entry = ctx.entry;
    const family = ctx.family;
    const displaySize = ctx.displaySize;
    const isDarkMode = ctx.isDarkMode;
  }
});
```

其中 `family` 表示小组件的尺寸，取值从 0 ~ 2 分别表示小、中、大三种布局。`displaySize` 反映了当前小组件的显示大小。

通过这些环境信息，我们可以动态地决定要返回什么样的视图给系统。

## 默认实现

当您的脚本并不需要刷新，或是默认的刷新策略满足需求，您可以仅提供一个 render 函数：

```js
$widget.setTimeline(ctx => {

});
```

这样，JSBox 会自动创建 entry 并设置每小时刷新一次的策略。

# 应用内预览

开发阶段，在主应用内调用 `$widget.setTimeline` 时，将打开预览小组件的模拟环境。同时支持三种尺寸，固定展示在时间线中的第一个 entry。

如需将脚本应用到实际桌面，请参考前述的设置方法。

# 网络请求最佳实践

您可以在 `setTimeline` 之前请求网络，通过异步的方式获取数据。由于时间线工作机制的限制，我们可能无法实现先展示缓存，然后获取新数据，再刷新页面这样的经典缓存逻辑。

但网络请求总是可能会失败的，在这种情况下展示缓存结果要好于显示出错信息，所以我们建议使用如下缓存策略：

```js
async function fetch() {
  const cache = readCache();
  const resp1 = await $http.get();
  if (failed(resp1)) {
    return cache;
  }

  const resp2 = await $http.download();
  if (failed(resp2)) {
    return cache;
  }

  const data = resp2.data;
  if (data) {
    writeCache(data);
  }

  return data;
}

const data = await fetch();
```

然后使用 `data` 构建时间线，这样小组件上会尽可能地显示内容，尽管内容可能不是最新。

完整示例请参考：https://github.com/cyanzhong/jsbox-widgets/blob/master/xkcd.js

---

## 文件内容解读与示例

### 用途说明

本文档深入探讨了 JSBox 桌面小组件的**时间线（Timeline）**这一核心概念。时间线是 iOS 14+ 桌面小组件更新和显示内容的基石。理解其工作原理对于开发高效、响应迅速且符合系统规范的小组件至关重要。

### 核心概念：时间线的工作原理

桌面小组件并非实时运行的应用程序，它们是系统在特定时间点显示的**一系列静态快照**。你的 JSBox 脚本扮演着“时间线提供者”（Timeline Provider）的角色，负责告诉系统：

1.  **何时**显示哪个快照（`entries`）。
2.  **何时**再次请求新的时间线（`policy`）。

系统会根据你提供的时间线和更新策略，在后台调度你的脚本运行，生成新的快照，并在合适的时候更新小组件的显示。

### `$widget.setTimeline(options)` 详解

这是小组件脚本中**最重要、最核心**的函数，用于向系统提供小组件的时间线。

#### 1. `entries` (时间线入口)

-   **用途**: 定义一系列预先计算好的快照，以及它们应该显示的时间。如果你的小组件内容可以提前预测（例如，未来几小时的天气预报），你可以一次性提供多个 `entry`。
-   **结构**: 每个 `entry` 是一个对象，包含：
    -   `date`: 必填，一个 `Date` 对象，表示该快照应该显示的时间点。
    -   `info`: 可选，一个任意的 JavaScript 对象，用于携带该快照所需的上下文数据。这些数据可以在 `render` 函数中通过 `ctx.entry.info` 访问。

#### 2. `policy` (更新策略)

-   **用途**: 告诉系统在时间线中的最后一个 `entry` 被显示后，何时再次请求新的时间线。这只是一个“建议”，系统会根据设备电量、网络状况等因素进行优化，不保证立即刷新。
-   **类型**: 
    -   `atEnd: true`: 在时间线中的最后一个 `entry` 被显示后，系统会请求新的时间线。适用于数据有规律更新的场景。
    -   `afterDate: aDate`: 在指定 `aDate` 之后，系统会请求新的时间线。适用于在某个特定时间点后需要更新的场景。
    -   `never: true`: 小组件内容是静态的，不会周期性更新。适用于显示固定信息的场景。

#### 3. `render(ctx)` (渲染函数)

-   **用途**: 这是实际构建小组件 UI 的地方。系统在需要显示某个 `entry` 时会调用此函数。
-   **`ctx` 对象**: 包含了当前渲染上下文的关键信息，你可以利用这些信息来动态调整小组件的布局和内容：
    -   `ctx.entry`: 当前正在渲染的 `TimelineEntry` 对象，你可以从中获取 `date` 和 `info`。
    -   `ctx.family`: 小组件的尺寸类型（例如 `$widgetFamily.small`, `medium`, `large` 等）。
    -   `ctx.displaySize`: 小组件的实际显示尺寸。
    -   `ctx.isDarkMode`: 小组件是否处于深色模式。
-   **返回值**: 必须返回一个符合小组件布局语法（如 `hstack`, `vstack`, `zstack`）的视图定义对象。

### 默认实现

如果你的脚本没有提供 `entries` 和 `policy`，JSBox 会自动为你创建一个默认的时间线，通常是每小时刷新一次。

### 应用内预览

在开发阶段，当你在主应用中调用 `$widget.setTimeline` 时，JSBox 会自动打开一个模拟环境来预览小组件的效果。这对于快速迭代和调试非常方便。

### 网络请求最佳实践

由于小组件的性能限制，`setTimeline` 必须在很短的时间内完成。因此，**推荐在调用 `setTimeline` 之前进行所有耗时的数据获取操作**（如网络请求）。同时，为了确保小组件在网络不佳时也能显示内容，建议采用“缓存优先”的策略：

1.  尝试从缓存中读取数据。
2.  发起网络请求获取最新数据。
3.  如果网络请求失败，则显示缓存数据。
4.  如果网络请求成功，则更新缓存并显示最新数据。

这样可以确保小组件上始终有内容显示，即使内容可能不是最新。

### 示例代码：一个实时更新的时钟小组件

下面的示例将创建一个简单的时钟小组件，它会每分钟更新一次，显示当前的精确时间。

```javascript
// 1. 定义小组件的渲染函数
function renderClockWidget(ctx) {
  const now = ctx.entry.date; // 获取当前快照的时间
  const family = ctx.family; // 获取小组件尺寸

  let timeString;
  if (family === $widgetFamily.small) {
    // 小尺寸只显示小时和分钟
    timeString = now.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
  } else {
    // 中型或大型显示完整时间
    timeString = now.toLocaleTimeString();
  }

  return {
    type: "vstack",
    props: {
      alignment: $widget.horizontalAlignment.center,
      spacing: 5,
      bgcolor: $color("black"),
      cornerRadius: 15
    },
    views: [
      {
        type: "label",
        props: {
          text: timeString,
          font: $font("bold", family === $widgetFamily.small ? 30 : 40),
          textColor: $color("white")
        }
      },
      {
        type: "label",
        props: {
          text: now.toLocaleDateString(),
          font: $font(16),
          textColor: $color("lightGray")
        }
      }
    ]
  };
}

// 2. 计算时间线入口
const entries = [];
const now = new Date();

// 为接下来的 60 分钟，每分钟创建一个 entry
for (let i = 0; i < 60; i++) {
  const entryDate = new Date(now.getFullYear(), now.getMonth(), now.getDate(), now.getHours(), now.getMinutes() + i, 0, 0);
  entries.push({
    date: entryDate,
    info: {},
  });
}

// 3. 设置小组件时间线
$widget.setTimeline({
  entries: entries,
  policy: {
    atEnd: true // 在最后一个 entry 显示后，请求新的时间线
  },
  render: renderClockWidget // 使用上面定义的渲染函数
});
```

**代码解读**：

1.  **`renderClockWidget(ctx)`**: 这个函数根据 `ctx.family` 动态调整了时间的显示格式和字体大小，确保在不同尺寸的小组件上都能良好显示。
2.  **`entries` 的生成**: 我们预先计算了未来 60 分钟的每个整分钟的时间点，并为每个时间点创建了一个 `entry`。这意味着系统可以在接下来的一个小时内，无需再次运行脚本就能更新小组件。
3.  **`policy: { atEnd: true }`**: 告诉系统，当最后一个 `entry`（即当前时间一个小时后的时间）被显示后，再来请求新的时间线。这确保了小组件能够持续更新。

`Timeline` 是桌面小组件的灵魂。熟练掌握 `entries`、`policy` 和 `render` 函数的用法，是开发动态、高效且用户体验良好小组件的关键。 
