# type: "lottie"

`lottie` 用于创建一个 [Lottie](http://airbnb.io/lottie/) 控件：

```js
{
  type: "lottie",
  props: {
    src: "assets/lottie-btn.json",
  },
  layout: (make, view) => {
    make.size.equalTo($size(100, 100));
    make.center.equalTo(view.super);
  }
}
```

`src` 可以是 bundle 内的路径，也可以是一个 http 地址，同时，你还可以通过 `json` 或 `data` 来加载 Lottie：

```js
// JSON
$("lottie").json = {};
// Data
$("lottie").data = $file.read("assets/lottie-btn.json");
```

# lottie.play(args?)

播放动画：

```js
$("lottie").play();
```

注册回调消息：

```js
$("lottie").play({
  handler: finished => {

  }
});
```

控制起始结束帧数：

```js
$("lottie").play({
  fromFrame: 0, // Optional
  toFrame: 0,
  handler: finished => {
    // Optional
  }
});
```

控制起始结束进度：

```js
$("lottie").play({
  fromProgress: 0, // Optional
  toProgress: 0.5,
  handler: finished => {
    // Optional
  }
});
```

使用 Promise：

```js
let finished = await $("lottie").play({ toProgress: 0.5, async: true });
```

# lottie.pause()

暂停动画：

```js
$("lottie").pause();
```

# lottie.stop()

停止动画：

```js
$("lottie").stop();
```

# lottie.update()

强制重画当前帧：

```js
$("lottie").update();
```

# lottie.progressForFrame(frame)

获得帧对应的进度：

```js
let progress = $("lottie").progressForFrame(0);
```

# lottie.frameForProgress(progress)

获得进度对应的帧：

```js
let frame = $("lottie").frameForProgress(0.5);
```

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
json | json | 只写 | 使用 json 加载 Lottie
data | $data | 只写 | 使用 data 加载 Lottie
src | string | 只写 | 使用路径或网址加载 Lottie
playing | bool | 只读 | 是否正在播放
loop | bool | 读写 | 是否循环播放
autoReverse | bool | 读写 | 是否自动反转
progress | number | 读写 | 播放进度
frameIndex | number | 读写 | 播放帧数
speed | number | 读写 | 播放速度
duration | number | 只读 | 动画时长

请参考这个项目来了解更多：https://github.com/cyanzhong/xTeko/tree/master/extension-demos/lottie-example

---

## 文件内容解读与示例

### 组件用途与背景知识

`lottie` 组件用于在你的界面中渲染和播放 **Lottie 动画**。那么，什么是 Lottie？

Lottie 是由 Airbnb 开发的一个开源库，它能够实时渲染由 Adobe After Effects 设计的矢量动画。设计师可以将复杂的动画导出为一个轻量的 `.json` 文件，而开发者则可以通过 `lottie` 组件直接在应用中播放这个文件。这极大地简化了在应用中实现高质量、流畅动画的流程。

简而言之，`lottie` 组件是你为 UI 添加精美动效、提升用户体验的利器。你可以从 [LottieFiles](https://lottiefiles.com/) 等社区找到海量的免费和付费 Lottie 动画资源。

### 核心概念

1.  **加载动画**: 你有三种方式来加载 Lottie 的 `.json` 文件：
    - `src`: 最常用的方式，提供一个网络 URL 或本地文件路径。
    - `json`: 如果你已经有了一个解析好的 JSON 对象，可以直接赋值给它。
    - `data`: 如果你通过 `$file.read` 等方式读取了文件的原始二进制数据（一个 `$data` 对象），可以赋值给它。

2.  **播放控制**: 
    - **属性控制**: 通过 `loop` (是否循环), `autoReverse` (是否自动反向播放), `speed` (播放速度) 等属性可以配置动画的基本行为。
    - **方法控制**: `play()`, `pause()`, `stop()` 三个核心方法提供了对播放过程的精确控制。

3.  **分段播放**: `play()` 方法非常强大，它支持传入 `fromProgress`/`toProgress` 或 `fromFrame`/`toFrame` 参数。这让你能只播放动画的某一个片段，对于实现交互式动画（例如，一个开关从“关”到“开”可能只对应动画的 0% -> 50% 进度）至关重要。

### 示例代码：Lottie 动画播放控制器

下面的示例将加载一个网络上的 Lottie 动画，并创建几个按钮来演示如何控制它的播放。

```javascript
const lottieURL = "https://assets1.lottiefiles.com/packages/lf20_v7ppxlfr.json"; // 一个火箭动画

$ui.render({
  props: {
    title: "Lottie 组件示例"
  },
  views: [
    {
      type: "lottie",
      props: {
        id: "rocket-lottie",
        src: lottieURL,
        loop: true
      },
      layout: (make, view) => {
        make.top.inset(20);
        make.centerX.equalTo(view.super);
        make.size.equalTo($size(250, 250));
      }
    },
    {
      // 控制按钮的容器
      type: "stack",
      props: {
        axis: $stackViewAxis.horizontal,
        distribution: $stackViewDistribution.fillEqually,
        spacing: 10
      },
      layout: (make, view) => {
        make.top.equalTo($("rocket-lottie").bottom).offset(20);
        make.left.right.inset(20);
        make.height.equalTo(40);
      },
      views: [
        { type: "button", props: { title: "播放" }, events: { tapped: () => $("rocket-lottie").play() } },
        { type: "button", props: { title: "暂停" }, events: { tapped: () => $("rocket-lottie").pause() } },
        { type: "button", props: { title: "停止" }, events: { tapped: () => $("rocket-lottie").stop() } },
        {
          type: "button",
          props: { title: "播放一半" },
          events: {
            tapped: () => {
              // 从头播放到 50% 的进度
              $("rocket-lottie").play({ toProgress: 0.5 });
            }
          }
        }
      ]
    }
  ]
});
```

**代码解读**：

1.  我们首先创建了一个 `lottie` 视图，通过 `src` 属性加载了一个来自 LottieFiles 网站的动画，并设置了 `loop: true` 让它默认循环播放。
2.  下方我们用一个 `stack` 视图横向排列了四个按钮。
3.  每个按钮的 `tapped` 事件都直接调用了 `lottie` 视图实例 (`$("rocket-lottie")`) 的一个方法：
    - “播放”按钮调用 `play()`。
    - “暂停”按钮调用 `pause()`。
    - “停止”按钮调用 `stop()`，这会将动画重置到第 0 帧。
    - “播放一半”按钮则演示了 `play()` 方法的高级用法，通过 `{ toProgress: 0.5 }` 参数，让动画从当前位置播放到 50% 进度时自动停止。

通过 `lottie` 组件，你可以用极低的成本为你的脚本增添生动有趣的动态效果，是提升产品质感和用户体验的绝佳选择。 
