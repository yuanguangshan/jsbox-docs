> 对于简单的音频播放，JSBox 通过 $audio 提供了支持

# $audio.play(object)

播放一个音频：

```js
// 通过 sound id 播放系统音频
$audio.play({
  id: 1000
})
```

iOS 系统内置 sound id 请参考：https://github.com/TUNER88/iOSSystemSoundsLibrary 。

```js
// 播放网络上的音频
$audio.play({
  url: "https://"
})
```

```js
// 播放本地音频
$audio.play({
  path: "audio.wav"
})
```

```js
// 暂停播放
$audio.pause()
```

```js
// 恢复播放
$audio.resume()
```

```js
// 停止播放
$audio.stop()
```

```js
// 设置播放进度（秒）
$audio.seek(60)
```

```js
// 获得当前的状态
const status = $audio.status;
// 0: 已停止, 1: 等待播放, 2: 正在播放
```

```js
// 获取时长
const duration = $audio.duration;
```

```js
// 获取当前的播放时间
const offset = $audio.offset;
```

# 事件消息

音频播放过程中可以监听一些消息，例如：

```js
$audio.play({
  events: {
    itemEnded: function() {},
    timeJumped: function() {},
    didPlayToEndTime: function() {},
    failedToPlayToEndTime: function() {},
    playbackStalled: function() {},
    newAccessLogEntry: function() {},
    newErrorLogEntry: function() {},
  }
})
```

参考：https://developer.apple.com/documentation/avfoundation/avplayeritem?language=objc

---

## 文件内容解读与示例

### 用途说明

`$audio` API 提供了在 JSBox 脚本中**播放音频**的简洁方法。它支持播放 iOS 系统内置的音效、网络上的音频文件以及本地存储的音频文件。这对于为脚本添加声音提示、播放背景音乐或简单的音频内容非常有用。

### 核心方法：`$audio.play(options)`

这是启动音频播放的主要方法。`options` 对象用于指定音频来源和配置播放行为。

#### 1. 音频来源

-   **`id` (系统音效)**: 播放 iOS 系统内置的短促音效。你需要提供一个系统音效 ID。例如，`1000` 可能是某个默认的点击音。
-   **`url` (网络音频)**: 播放来自互联网的音频文件（如 MP3, WAV 等）。
-   **`path` (本地音频)**: 播放脚本沙盒内或通过特殊协议（如 `shared://`）访问的本地音频文件。

#### 2. 播放控制

-   **`$audio.pause()`**: 暂停当前正在播放的音频。
-   **`$audio.resume()`**: 从暂停处继续播放。
-   **`$audio.stop()`**: 停止播放，并将播放头重置到音频的开头。
-   **`$audio.seek(seconds)`**: 将播放头跳转到音频的指定时间点（单位：秒）。

#### 3. 状态与进度

-   **`$audio.status`**: 获取当前播放器的状态。
    -   `0`: 已停止
    -   `1`: 等待播放
    -   `2`: 正在播放
-   **`$audio.duration`**: 获取当前音频的总时长（单位：秒）。
-   **`$audio.offset`**: 获取当前播放时间点（单位：秒）。

#### 4. 事件监听

在 `play` 方法的 `events` 选项中，你可以监听播放过程中的各种事件，例如 `itemEnded`（音频播放结束）、`didPlayToEndTime`（播放到末尾）等，以便在特定事件发生时执行自定义逻辑。

### 示例代码：一个简单的音频播放器

下面的示例将创建一个简单的 UI，用于播放一个本地音频文件，并提供播放、暂停、停止和进度控制功能。

```javascript
// 假设你的脚本 assets 目录下有一个名为 "sample.mp3" 的音频文件
// 你可以替换成任何有效的本地或网络音频文件
const AUDIO_FILE_PATH = "assets/sample.mp3"; 

$ui.render({
  props: { title: "音频播放器" },
  views: [
    {
      type: "label",
      props: {
        id: "time-label",
        text: "00:00 / 00:00",
        align: $align.center
      },
      layout: make => {
        make.top.inset(50);
        make.left.right.inset(20);
      }
    },
    {
      type: "slider",
      props: {
        id: "progress-slider",
        min: 0,
        max: 1,
        value: 0,
        continuous: true
      },
      layout: make => {
        make.top.equalTo($("time-label").bottom).offset(20);
        make.left.right.inset(20);
      },
      events: {
        changed: sender => {
          // 当用户拖动滑块时，跳转到对应的时间点
          if ($audio.duration > 0) {
            $audio.seek(sender.value * $audio.duration);
          }
        }
      }
    },
    {
      type: "stack",
      props: {
        axis: $stackViewAxis.horizontal,
        distribution: $stackViewDistribution.fillEqually,
        spacing: 10
      },
      layout: make => {
        make.top.equalTo($("progress-slider").bottom).offset(20);
        make.left.right.inset(20);
        make.height.equalTo(40);
      },
      views: [
        { type: "button", props: { title: "播放" }, events: { tapped: playAudio } },
        { type: "button", props: { title: "暂停" }, events: { tapped: () => $audio.pause() } },
        { type: "button", props: { title: "停止" }, events: { tapped: () => $audio.stop() } }
      ]
    }
  ]
});

let updateTimer; // 用于更新进度条和时间的定时器

function formatTime(seconds) {
  const minutes = Math.floor(seconds / 60);
  const remainingSeconds = Math.floor(seconds % 60);
  return `${minutes.toString().padStart(2, '0')}:${remainingSeconds.toString().padStart(2, '0')}`;
}

function updateProgress() {
  const offset = $audio.offset;
  const duration = $audio.duration;

  if (duration > 0) {
    $("progress-slider").value = offset / duration;
    $("time-label").text = `${formatTime(offset)} / ${formatTime(duration)}`;
  } else {
    $("progress-slider").value = 0;
    $("time-label").text = "00:00 / 00:00";
  }
}

function playAudio() {
  $audio.play({
    path: AUDIO_FILE_PATH,
    events: {
      itemEnded: () => {
        // 播放结束后停止定时器并重置 UI
        if (updateTimer) updateTimer.invalidate();
        $("progress-slider").value = 0;
        $("time-label").text = `${formatTime(0)} / ${formatTime($audio.duration)}`;
        $ui.toast("播放结束");
      },
      didPlayToEndTime: () => {
        // 确保 itemEnded 也会被触发，这里可以做一些额外处理
      }
    }
  });

  // 启动定时器，每 0.5 秒更新一次进度
  if (updateTimer) updateTimer.invalidate(); // 先清除旧的
  updateTimer = $timer.schedule({
    interval: 0.5,
    handler: updateProgress
  });
}
```

**代码解读**：

1.  **音频源**: `AUDIO_FILE_PATH` 指定了要播放的本地音频文件。
2.  **UI 联动**: `time-label` 显示当前播放时间和总时长，`progress-slider` 显示播放进度。
3.  **播放控制**: “播放”、“暂停”、“停止”按钮分别调用 `$audio.play()`、`$audio.pause()`、`$audio.stop()`。
4.  **进度更新**: `updateProgress` 函数通过 `$audio.offset` 和 `$audio.duration` 获取当前播放时间和总时长，并更新 UI。一个 `$timer.schedule` 定时器每 0.5 秒调用一次 `updateProgress`，实现流畅的进度显示。
5.  **播放结束**: `itemEnded` 事件在音频播放结束后触发，用于停止定时器并重置 UI 状态。

`$audio` API 为你的脚本提供了基础的音频播放能力，适用于各种需要声音反馈或播放媒体内容的场景。 
