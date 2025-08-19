# type: "chart"

`chart` 用来绘制图表来实现数据可视化：

```js
{
  type: "chart",
  props: {
    options: {
      "legend": {
        "data": ["Chart"]
      },
      "xAxis": {
        "data": [
          "A",
          "B",
          "C",
          "D"
        ]
      },
      "yAxis": {},
      "series": [
        {
          "name": "foo",
          "type": "bar",
          "data": [5, 20, 36, 10]
        }
      ]
    }
  },
  layout: $layout.fill
}
```

这将在页面上画出一个柱状图，options 支持的参数请参考 [ECharts 文档](http://www.echartsjs.com/option.html)。

# 动态绘制

在某些时候数据不是静态的，绘制可以通过一个函数来完成，在 JSBox 里面你可以使用模板字符串来实现：

```js
$ui.render({
  views: [
    {
      type: "chart",
      layout: $layout.fill,
      events: {
        ready: chart => {
          let options = `
          options = {
            tooltip: {},
            backgroundColor: "#fff",
            visualMap: {
              show: false,
              dimension: 2,
              min: -1,
              max: 1,
              inRange: {
                color: [
                  "#313695",
                  "#4575b4",
                  "#74add1",
                  "#abd9e9",
                  "#e0f3f8",
                  "#ffffbf",
                  "#fee090",
                  "#fdae61",
                  "#f46d43",
                  "#d73027",
                  "#a50026"
                ]
              }
            },
            xAxis3D: {
              type: "value"
            },
            yAxis3D: {
              type: "value"
            },
            zAxis3D: {
              type: "value"
            },
            grid3D: {
              viewControl: {
                // projection: 'orthographic'
              }
            },
            series: [
              {
                type: "surface",
                wireframe: {
                  // show: false
                },
                equation: {
                  x: {
                    step: 0.05
                  },
                  y: {
                    step: 0.05
                  },
                  z: function(x, y) {
                    if (Math.abs(x) < 0.1 && Math.abs(y) < 0.1) {
                      return "-";
                    }
                    return Math.sin(x * Math.PI) * Math.sin(y * Math.PI);
                  }
                }
              }
            ]
          };`;
          chart.render(options);
        }
      }
    }
  ]
});
```

用 `options = ` 的方式定义，这种方式和静态绘制的区别是 options 可以含有函数调用。

# chart.dispatchAction(args)

用于触发事件：

```js
chart.dispatchAction({
  type: "dataZoom",
  start: 20,
  end: 30
});
```

# chart.getWidth(handler)

用于获取视图的宽度：

```js
let width = await chart.getWidth();
```

# chart.getHeight(handler)

用于获取视图的高度：

```js
let height = await chart.getHeight();
```

# chart.getOption(handler)

用于获取当前的 options:

```js
let options = await chart.getOption();
```

# chart.resize($size)

用于更新视图的大小：

```js
chart.resize($size(100, 100));
```

# chart.showLoading()

显示加载状态：

```js
chart.showLoading();
```

# chart.hideLoading()

隐藏加载状态：

```js
chart.hideLoading();
```

# chart.clear()

清除当前绘制的图表：

```js
chart.clear();
```

# event: rendered

`rendered` 会在绘制时被调用：

```js
events: {
  rendered: () => {

  }
}
```

# event: finished

`finished` 会在绘制完成时被调用：

```js
events: {
  finished: () => {

  }
}
```

# WebView

当前图表控件使用 WebView 封装 [ECharts](http://www.echartsjs.com/index.html) 来实现，所以支持所有 WebView 支持的特性，例如 JavaScript 注入以及 $notify 机制，详情请参考[网页视图](component/web.md)。

更多样例：https://github.com/cyanzhong/xTeko/tree/master/extension-demos/charts

---

## 文件内容解读与示例

### 组件用途

`chart` 组件是 JSBox 中用于**数据可视化**的专用高级组件。它能让你轻松创建各种常见的交互式图表，如柱状图、折线图、饼图、散点图等，而无需使用 `canvas` 进行底层的手动绘制。

### 核心概念：ECharts 封装

理解 `chart` 组件的关键在于认识到它**本质上是是对著名的开源图表库 [ECharts](https://echarts.apache.org/zh/index.html) 的封装**。JSBox 在一个 WebView（网页视图）中加载了 ECharts，并让你通过 `props` 和方法来与它交互。

这意味着：

- **所有图表的样式、数据和行为都由一个名为 `options` 的巨大配置对象决定。**
- **要掌握 `chart` 组件，你必须查阅 [ECharts 的官方配置项文档](https://echarts.apache.org/zh/option.html)。** JSBox 文档只提供桥梁，ECharts 文档才是真正的使用说明书。

### 两种渲染方式

1.  **静态渲染 (Static)**
    - **方式**：直接在 `props` 中提供 `options` 对象。
    - **适用场景**：图表所需的数据在 UI 渲染时就已经准备好了，是最简单直接的方式。

2.  **动态渲染 (Dynamic)**
    - **方式**：使用 `events: { ready: chart => { ... } }` 事件。当图表组件初始化完毕后，`ready` 事件会被触发，并返回 `chart` 实例。然后你可以在事件内部调用 `chart.render(options)` 来渲染图表。
    - **适用场景**：
        - 数据需要通过网络异步获取。
        - 图表配置项中包含函数（ECharts 的很多高级功能需要函数回调），因为静态的 JSON 对象无法包含函数。
    - **注意**：动态渲染时，`options` 需要被包裹在一个模板字符串中，并以 `options = ` 开头，这是 JSBox 与 WebView 内 ECharts 通信的一种特殊约定。

### 示例代码：创建混合图表（柱状图+折线图）

下面的示例将使用**静态渲染**的方式，创建一个展示“月度销售额”和“同比增长率”的混合图表。

```javascript
$ui.render({
  props: {
    title: "Chart 组件示例"
  },
  views: [
    {
      type: "chart",
      layout: $layout.fill,
      props: {
        // ECharts 的核心配置对象
        options: {
          // 图表标题
          title: {
            text: "月度销售报告"
          },
          // 提示框，鼠标悬浮时显示数据
          tooltip: {
            trigger: "axis"
          },
          // 图例，用于筛选系列
          legend: {
            data: ["销售额", "同比增长率"]
          },
          // X 轴配置
          xAxis: {
            data: ["一月", "二月", "三月", "四月", "五月"]
          },
          // Y 轴配置，这里我们配置了两个 Y 轴
          yAxis: [
            {
              type: "value",
              name: "销售额 (万元)"
            },
            {
              type: "value",
              name: "同比增长率",
              axisLabel: {
                formatter: "{value} %" // 给标签加上百分号
              }
            }
          ],
          // 系列（Series），图表的核心数据
          series: [
            {
              name: "销售额",
              type: "bar", // 类型为柱状图
              data: [50, 90, 110, 80, 150]
            },
            {
              name: "同比增长率",
              type: "line", // 类型为折线图
              yAxisIndex: 1, // 关键：指定此系列使用第二个 Y 轴
              data: [5, 10, 8, 7, 12]
            }
          ]
        }
      }
    }
  ]
});
```

**代码解读**：

- **`options` 结构**：整个配置清晰地分成了 `title`, `tooltip`, `legend`, `xAxis`, `yAxis`, `series` 等几个部分，这完全是 ECharts 的标准结构。
- **双 Y 轴**：我们在 `yAxis` 中定义了两个对象，分别对应“销售额”和“同比增长率”。
- **`series` 数据系列**：我们定义了两个数据系列，一个是 `type: "bar"` 的柱状图，另一个是 `type: "line"` 的折线图。
- **`yAxisIndex: 1`**: 这是实现混合图表的关键。我们告诉 ECharts，“同比增长率”这个折线图系列应该使用索引为 1 的 Y 轴（即我们定义的第二个 Y 轴），从而实现了两种不同量纲的数据在同一个图表中和谐共存。

总而言之，`chart` 组件为你打开了专业数据可视化的大门。它的强大完全取决于你对 ECharts 的熟悉程度。 
