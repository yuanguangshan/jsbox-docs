# type: "progress"

`progress` 用于创建一个不可交互的进度条，例如用来指示下载进度：

```js
{
  type: "progress",
  props: {
    value: 0.5
  },
  layout: function(make, view) {
    make.center.equalTo(view.super)
    make.width.equalTo(100)
  }
}
```

创建一个初始值在 50% 的进度条。

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
value | number | 读写 | 进度(0.0 ~ 1.0)
progressColor | $color | 读写 | 已走进度的颜色
trackColor | $color | 读写 | 进度条背景色

---

## 文件内容解读与示例

### 组件用途

`progress` 组件是一个纯粹的、不可交互的视觉元素，它的唯一用途就是**以图形方式显示一个任务的完成进度**。例如，你可以用它来指示文件下载、视频处理或任何耗时操作的当前状态，为用户提供直观的反馈。

### 核心属性详解

- **`value`**: 这是 `progress` 组件最核心的属性。它是一个 `number` 类型的值，范围从 **`0.0` (表示 0%，进度条是空的) 到 `1.0` (表示 100%，进度条是满的)**。要更新进度条，你只需要改变这个属性的值。

- **`progressColor`**: 用于设置进度条**已完成部分**的颜色。

- **`trackColor`**: 用于设置进度条**未完成部分**（即背景轨道）的颜色。

### 示例代码：模拟一个下载任务

下面的示例将创建一个进度条和一个“开始下载”按钮。点击按钮后，进度条会在 2 秒内从 0% 动态地更新到 100%，并同步显示百分比。

```javascript
$ui.render({
  props: {
    title: "Progress 组件示例"
  },
  views: [
    {
      type: "progress",
      props: {
        id: "download-progress",
        value: 0, // 初始值为 0
        progressColor: $color("green"),
        trackColor: $color("#E0E0E0")
      },
      layout: (make, view) => {
        make.center.equalTo(view.super);
        make.width.equalTo(250);
      }
    },
    {
      type: "label",
      props: {
        id: "percentage-label",
        text: "0%",
        font: $font(14),
        align: $align.center
      },
      layout: (make, view) => {
        make.top.equalTo($("download-progress").bottom).offset(10);
        make.centerX.equalTo(view.super);
      }
    },
    {
      type: "button",
      props: {
        id: "start-button",
        title: "开始下载"
      },
      layout: (make, view) => {
        make.top.equalTo($("percentage-label").bottom).offset(20);
        make.centerX.equalTo(view.super);
        make.width.equalTo(100);
      },
      events: {
        tapped: (sender) => {
          // 禁用按钮，防止重复点击
          sender.enabled = false;
          sender.title = "下载中...";

          let progressValue = 0;
          const timer = $timer.schedule({
            interval: 0.1, // 每 0.1 秒更新一次
            handler: () => {
              progressValue += 0.05; // 每次增加 5%
              
              // 更新进度条和标签
              $("download-progress").value = progressValue;
              $("percentage-label").text = `${Math.round(progressValue * 100)}%`;

              if (progressValue >= 1.0) {
                timer.invalidate(); // 停止计时器
                sender.enabled = true; // 恢复按钮
                sender.title = "重新下载";
                $ui.toast("下载完成！");
              }
            }
          });
        }
      }
    }
  ]
});
```

**代码解读**：

1.  我们创建了一个 `id` 为 `download-progress` 的进度条，并初始化 `value` 为 0。
2.  核心逻辑在按钮的 `tapped` 事件中。我们使用 `$timer.schedule` 创建了一个定时器，它每 0.1 秒执行一次。
3.  在定时器的 `handler` 中，我们不断增加 `progressValue` 的值，然后将这个新值赋给 `$("download-progress").value`，从而实现进度条的动态更新。
4.  同时，我们也更新了下方的 `label` 来显示具体的百分比数字。
5.  当进度达到 1.0 (100%) 时，我们调用 `timer.invalidate()` 来停止定时器，并恢复按钮的状态。

这个例子展示了 `progress` 组件最典型的用法：通过外部逻辑（如此处的定时器，在实际应用中可能是网络请求的回调）来周期性地更新其 `value` 属性，以向用户反馈任务的实时进度。 
