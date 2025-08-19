# type: "spinner"

`spinner` 用于创建一个 loading view：

```js
{
  type: "spinner",
  props: {
    loading: true
  },
  layout: function(make, view) {
    make.center.equalTo(view.super)
  }
}
```

将会创建一个初始状态是开启的开关。

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
loading | boolean | 读写 | 是否加载中
color | $color | 读写 | 颜色
style | number | 读写 | 0 ~ 2 表示样式

# start()

开始加载，等同于 `spinner.loading = true`。

# stop()

结束加载，等同于 `spinner.loading = false`。

# toggle()

切换状态，等同于 `spinner.loading = !spinner.loading`。

---

## 文件内容解读与示例

### 组件用途

`spinner` 组件用于显示一个**不确定性进度指示器**（Indeterminate Progress Indicator）。通俗地说，就是“加载中的菊花”。当你的应用正在执行一个耗时操作（如网络请求、数据处理），但无法预知其确切完成时间时，就应该使用 `spinner` 来告诉用户“程序正在处理中，请稍候”。

它与 `progress`（进度条）的区别在于：`progress` 用于显示一个已知进度的任务（例如，完成度 35%），而 `spinner` 用于显示一个未知时长的任务。

### 核心属性与方法

- **`loading` 属性**: 这是控制 `spinner` 状态的核心。它是一个布尔值：
  - `true`: `spinner` 可见并且开始旋转动画。
  - `false`: `spinner` 停止动画并变得不可见。

- **`start()` 和 `stop()` 方法**: 这两个方法是 `loading` 属性的便捷别名，能让代码更具可读性。
  - `$("spinnerId").start()` 等同于 `$("spinnerId").loading = true`。
  - `$("spinnerId").stop()` 等同于 `$("spinnerId").loading = false`。

- **`style` 属性**: 用于改变指示器的样式，通常对应不同的大小。例如 `0` 可能是中等大小，`1` 是大尺寸（具体对应关系可能因系统版本而异）。

### 示例代码：模拟网络请求

下面的示例将创建一个“获取数据”的按钮。点击后，一个 `spinner` 会出现并旋转；2秒后，模拟的数据加载完成，`spinner` 消失，数据显示在标签上。

```javascript
$ui.render({
  props: {
    title: "Spinner 组件示例"
  },
  views: [
    {
      type: "button",
      props: {
        id: "fetch-button",
        title: "获取最新消息"
      },
      layout: (make, view) => {
        make.centerX.equalTo(view.super);
        make.top.inset(30);
      },
      events: {
        tapped: (sender) => {
          const spinner = $("loading-spinner");
          const resultLabel = $("result-label");

          // 1. 任务开始：开始动画，并清空旧数据
          spinner.start();
          resultLabel.text = "";
          sender.enabled = false; // 禁用按钮

          // 2. 模拟一个耗时2秒的网络请求
          $delay(2, () => {
            // 3. 任务结束：停止动画，并显示新数据
            spinner.stop();
            resultLabel.text = "消息获取成功：JSBox 新版本已发布！";
            sender.enabled = true; // 恢复按钮
          });
        }
      }
    },
    {
      type: "spinner",
      props: {
        id: "loading-spinner",
        loading: false, // 初始状态为不加载
        style: 1 // 使用大尺寸样式
      },
      layout: (make, view) => {
        make.centerX.equalTo(view.super);
        make.top.equalTo($("fetch-button").bottom).offset(20);
      }
    },
    {
      type: "label",
      props: {
        id: "result-label",
        align: $align.center,
        lines: 0
      },
      layout: (make, view) => {
        make.top.equalTo($("loading-spinner").bottom).offset(20);
        make.left.right.inset(20);
      }
    }
  ]
});
```

**代码解读**：

这个例子完整地展示了 `spinner` 的标准使用流程：

1.  **初始状态**：`spinner` 的 `loading` 属性被设置为 `false`，所以它在界面加载时是不可见的。
2.  **任务开始**：当用户点击按钮，我们立即调用 `spinner.start()`，加载菊花开始旋转，同时禁用按钮以防重复操作。
3.  **任务结束**：在 `$delay` 模拟的异步回调中，我们首先调用 `spinner.stop()` 来隐藏加载菊花，然后才处理返回的数据（更新 `label` 的文本）并恢复按钮状态。

这个“**开始 -> 显示 spinner -> 结束 -> 隐藏 spinner**”的模式是所有加载指示器的通用逻辑。 
