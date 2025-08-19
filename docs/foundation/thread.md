> 在这里涉及到线程切换、延时执行、重复执行等内容。

# $thread.background(object)

在子线程执行一段代码：

```js
$thread.background({
  delay: 0,
  handler: function() {

  }
})
```

# $thread.main(object)

在主线程执行一段代码：

```js
$thread.main({
  delay: 0.3,
  handler: function() {
    
  }
})
```

参数 | 类型 | 说明
---|---|---
delay | number | 延迟执行的时间，可选
handler | function | 回调

如果不需要延迟执行，也可以简单地写为：

```js
$thread.main(() => {
  
});
```

# $delay(number, function)

一种更简单的执行延时任务的方法：

```js
$delay(3, () => {
  $ui.alert("Hey!")
})
```

3 秒钟后弹出一个 alert。

你可以通过这个方式取消执行：

```js
const task = $delay(10, () => {

});

// Cancel it
task.cancel();
```

# $wait(sec)

与 $delay 类似但支持 Promise:

```js
await $wait(2);

alert("Hey!");
```

# $timer.schedule(object)

用于执行一个重复的任务：

```js
const timer = $timer.schedule({
  interval: 3,
  handler: function() {
    $ui.toast("Hey!")
  }
});
```

上述代码每隔三秒显示一次 `Hey!`，已经设置的任务可以取消：

```js
timer.invalidate()
```

---

## 文件内容解读与示例

### 用途说明

`$thread` API 及其相关的 `$delay`、`$wait` 和 `$timer` 函数是 JSBox 中用于**控制代码执行流程和线程管理**的核心工具。在移动应用开发中，为了保持用户界面的流畅响应，尤其是在执行耗时操作时，合理地使用线程和异步执行机制至关重要。

### 核心概念

-   **主线程 (Main Thread)**: 负责处理用户界面（UI）的更新和所有用户交互事件。所有对 UI 的操作（如 `$ui.render`、修改 `label.text`）都**必须**在主线程执行。长时间阻塞主线程会导致界面卡顿、无响应（“ANR”）。
-   **子线程 (Background Thread)**: 用于执行耗时操作，如网络请求、复杂计算、大量文件读写等。这些操作在子线程中执行，可以避免阻塞主线程，从而保持 UI 的流畅。
-   **异步执行**: 大多数耗时操作都应该是异步的，即它们不会立即返回结果，而是通过回调函数或 Promise 在操作完成后通知你。

### 方法详解与示例

#### 1. 线程切换：`$thread.background()` 和 `$thread.main()`

-   **`$thread.background(options)`**: 将 `handler` 中的代码放到**子线程**执行。适用于所有耗时操作。
-   **`$thread.main(options)`**: 将 `handler` 中的代码放到**主线程**执行。适用于所有 UI 更新操作。
-   **`delay`**: 两个方法都支持 `delay` 参数，用于延迟执行 `handler` 中的代码。

**示例**：在子线程执行耗时操作，并在主线程更新 UI

```javascript
$ui.render({
  props: { title: "线程示例" },
  views: [
    {
      type: "label",
      props: { id: "status-label", text: "点击按钮开始任务" },
      layout: make => make.center.equalTo(view.super)
    },
    {
      type: "button",
      props: { title: "开始耗时任务" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(20).centerX.equalTo(view.super)
      ,
      events: {
        tapped: () => {
          $("status-label").text = "任务进行中... (UI 不会卡顿)";

          $thread.background({
            handler: () => {
              console.log("子线程: 开始执行耗时计算...");
              // 模拟一个非常耗时的计算
              let sum = 0;
              for (let i = 0; i < 500000000; i++) {
                sum += i;
              }
              console.log("子线程: 计算完成，结果: ", sum);

              // 计算完成后，需要回到主线程更新 UI
              $thread.main(() => {
                $("status-label").text = "任务完成！";
                $ui.toast("任务已完成！");
              });
            }
          });
        }
      }
    }
  ]
});
```

#### 2. 延时执行：`$delay()` 和 `$wait()`

-   **`$delay(seconds, handler)`**: 在指定秒数后执行一次性任务。返回一个任务对象，可以通过 `task.cancel()` 取消。
-   **`$wait(seconds)`**: 类似于 `$delay`，但它返回一个 Promise。这使得它非常适合与 `async/await` 语法结合使用，让异步代码看起来像同步代码一样简洁。

**示例**：

```javascript
// 使用 $delay
$delay(2, () => {
  $ui.toast("这个提示在 2 秒后出现。");
});

// 使用 $wait (需要 async 函数)
async function showDelayedAlert() {
  await $wait(3); // 等待 3 秒
  $ui.alert("这个提示在 3 秒后出现 (使用 await)。");
}
showDelayedAlert();
```

#### 3. 重复执行：`$timer.schedule()`

-   **`$timer.schedule(options)`**: 创建一个重复执行的定时器。`options` 包含 `interval`（间隔秒数）和 `handler`（每次执行的回调函数）。
-   **`timer.invalidate()`**: 停止定时器，防止其继续执行。

**示例**：一个简单的倒计时器

```javascript
let countdown = 5;
const countdownTimer = $timer.schedule({
  interval: 1, // 每秒执行一次
  handler: () => {
    countdown--;
    $("countdown-label").text = `倒计时: ${countdown} 秒`;
    if (countdown <= 0) {
      countdownTimer.invalidate(); // 停止定时器
      $ui.alert("时间到！");
      $("countdown-label").text = "倒计时结束";
    }
  }
});

$ui.render({
  props: { title: "倒计时" },
  views: [
    {
      type: "label",
      props: { id: "countdown-label", text: `倒计时: ${countdown} 秒`, font: $font("bold", 30) },
      layout: make => make.center.equalTo(view.super)
    }
  ]
});
```

### 总结

`$thread` 及其相关的 `$delay`、`$wait` 和 `$timer` 是构建高性能、响应式 JSBox 脚本不可或缺的工具。始终记住：**耗时操作在子线程，UI 更新在主线程**。合理利用这些 API，可以显著提升你的脚本的用户体验。 
