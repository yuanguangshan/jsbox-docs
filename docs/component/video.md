# type: "video"

`video` 用于创建一个播放视频的控件：

```js
{
  type: "video",
  props: {
    src: "https://images.apple.com/media/cn/ipad-pro/2017/43c41767_0723_4506_889f_0180acc13482/films/feature/ipad-pro-feature-cn-20170605_1280x720h.mp4",
    poster: "https://images.apple.com/v/iphone/home/v/images/home/limited_edition/iphone_7_product_red_large_2x.jpg"
  },
  layout: function(make, view) {
    make.left.right.equalTo(0)
    make.centerY.equalTo(view.super)
    make.height.equalTo(256)
  }
}
```

由于 video 组件目前使用 WebView 实现，你也可以用 `local://` 协议来加载本地文件：

```js
{
  type: "video",
  props: {
    src: "local://assets/video.mp4",
    poster: "local://assets/poster.jpg"
  },
  layout: function(make, view) {
    make.left.right.equalTo(0)
    make.centerY.equalTo(view.super)
    make.height.equalTo(256)
  }
}
```

另外，你还可以像这个 demo 一样使用 `AVPlayerViewController` 来播放视频：https://gist.github.com/cyanzhong/c3992af39043c8e0f25424536c379595

# video.pause()

停止播放视频：

```js
$("video").pause()
```

# video.play()

继续播放视频：

```js
$("video").play()
```

# video.toggle()

切换视频播放状态：

```js
$("video").toggle()
```

`src`: 视频地址 `poster`: 封面图片地址。

内部通过一个 WKWebView 实现，所以理论上也完全可以通过 web 组件自行实现。

---

## 文件内容解读与示例

### 组件用途

`video` 组件提供了一个在 UI 中直接嵌入**视频播放器**的便捷方式。你可以用它来播放来自网络 URL 或本地文件系统中的视频。

### 核心概念

1.  **工作原理**: 如文档所述，`video` 组件是基于 `WKWebView` 实现的，本质上是利用了 HTML5 的 `<video>` 标签能力。这意味着它的行为和支持的视频格式与 iOS 上的 Safari 浏览器基本一致。

2.  **`src` 和 `poster` 属性**: 这是设置播放器的两个主要属性。
    - `src`: **视频源地址**。可以是 `https://` 开头的网址，也可以是 `local://` 开头的本地文件路径（需将视频文件与脚本一同打包）。
    - `poster`: **封面图片地址**。在视频开始播放前，播放器区域会显示这张图片，这能极大地改善初始加载时的用户体验。

3.  **播放控制**: 你可以通过 `play()`、`pause()` 和 `toggle()` 这三个简单的方法，使用外部按钮或其他事件来控制视频的播放状态。

4.  **高级替代方案**: 文档中提到，对于需要更精细播放控制（如自定义UI、轨道选择、精确跳转等）的高级用户，可以通过 `$objc` Runtime 的方式来创建和使用原生的 `AVPlayerViewController`，以获得更强大的功能和性能。

### 示例代码：带自定义控制按钮的播放器

下面的示例将创建一个视频播放器，并在其下方添加“播放”和“暂停”两个按钮来手动控制播放。

```javascript
const videoURL = "https://www.w3schools.com/html/mov_bbb.mp4";
const posterURL = "https://www.w3schools.com/html/pic_trulli.jpg";

$ui.render({
  props: { title: "Video 组件示例" },
  views: [
    {
      type: "video",
      props: {
        id: "my-player",
        src: videoURL,
        poster: posterURL
      },
      layout: (make, view) => {
        make.top.left.right.inset(10);
        make.height.equalTo(200);
      }
    },
    {
      // 控制按钮容器
      type: "stack",
      props: {
        axis: $stackViewAxis.horizontal,
        distribution: $stackViewDistribution.fillEqually,
        spacing: 10
      },
      layout: (make, view) => {
        make.top.equalTo($("my-player").bottom).offset(10);
        make.centerX.equalTo(view.super);
        make.width.equalTo(200);
        make.height.equalTo(40);
      },
      views: [
        {
          type: "button",
          props: { title: "播放", symbol: "play.fill" },
          events: {
            tapped: () => $("my-player").play()
          }
        },
        {
          type: "button",
          props: { title: "暂停", symbol: "pause.fill" },
          events: {
            tapped: () => $("my-player").pause()
          }
        }
      ]
    }
  ]
});
```

**代码解读**：

1.  我们创建了一个 `id` 为 `my-player` 的 `video` 视图，并为其 `src` 和 `poster` 属性分别指定了视频源和封面图的 URL。
2.  在视频下方，我们用一个 `stack` 放置了“播放”和“暂停”两个按钮。
3.  “播放”按钮的 `tapped` 事件直接调用 `$("my-player").play()` 方法，命令播放器开始或继续播放。
4.  “暂停”按钮的 `tapped` 事件则调用 `$("my-player").pause()` 方法来暂停视频。

这个例子展示了 `video` 组件的基础用法：设置源、提供封面，并通过其提供的方法进行外部控制。对于大多数简单的视频展示需求，`video` 组件是一个足够便捷的选择。 
