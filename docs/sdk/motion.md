> 用于与系统自带的传感器交互，例如获取加速度

# $motion.startUpdates(object)

监听 motion 数据变化：

```js
$motion.startUpdates({
  interval: 0.1,
  handler: function(resp) {

  }
})
```

返回的数据请参考 [CMDeviceMotion](https://developer.apple.com/documentation/coremotion/cmdevicemotion)。

# $motion.stopUpdates()

停止更新 motion 数据。

---

## 文件内容解读与示例

### 用途说明

`$motion` API 提供了与 iOS 设备**运动传感器**进行交互的能力。它允许你的脚本获取设备的加速度计、陀螺仪、磁力计以及设备姿态（attitude）等数据。这对于构建运动追踪、游戏控制、或任何需要感知设备物理运动的脚本都非常有用。

### 核心概念：传感器数据流与异步处理

-   **传感器数据**: `$motion` 模块提供的是原始的传感器数据流，这些数据会持续地、高频率地更新。
-   **异步流**: 数据通过 `handler` 回调函数以流的形式提供。这意味着你的脚本会不断地接收到新的传感器读数。
-   **权限**: 访问运动传感器通常不需要显式权限请求，但过度使用可能会影响设备性能和电池续航。

### 主要功能与方法详解

#### 1. 开始监听运动数据：`$motion.startUpdates(options)`

-   **用途**: 启动对设备运动数据的持续监听。一旦启动，`handler` 回调函数会按照指定的 `interval` 频率被调用，并传入最新的运动数据。
-   **参数**: 
    -   `interval`: 数据更新的频率，单位为秒。例如 `0.1` 表示每 0.1 秒（100 毫秒）更新一次。
    -   `handler`: 回调函数，接收一个 `resp` 对象作为参数，其中包含了各种运动传感器数据。`resp` 对象的结构与原生 `CMDeviceMotion` 类似，通常包含：
        -   `resp.acceleration`: 加速度计数据（`x`, `y`, `z` 轴的加速度）。
        -   `resp.rotationRate`: 陀螺仪数据（`x`, `y`, `z` 轴的旋转速率）。
        -   `resp.magneticField`: 磁力计数据（`x`, `y`, `z` 轴的磁场强度）。
        -   `resp.attitude`: 设备姿态数据（`roll`, `pitch`, `yaw`，表示设备在三维空间中的倾斜角度）。
        -   `resp.gravity`: 重力向量（`x`, `y`, `z`）。
        -   `resp.userAcceleration`: 用户加速度（不包含重力影响的加速度）。

**示例**：实时显示设备的加速度

```javascript
let motionIntervalId = null;

function startMotionTracking() {
  if (motionIntervalId) return; // 避免重复启动

  motionIntervalId = $motion.startUpdates({
    interval: 0.05, // 每 50 毫秒更新一次
    handler: (resp) => {
      const acc = resp.acceleration;
      $("accel-x").text = `X: ${acc.x.toFixed(3)}`;
      $("accel-y").text = `Y: ${acc.y.toFixed(3)}`;
      $("accel-z").text = `Z: ${acc.z.toFixed(3)}`;

      // 简单摇晃检测：如果任一轴的加速度超过阈值
      const shakeThreshold = 1.5;
      if (Math.abs(acc.x) > shakeThreshold || Math.abs(acc.y) > shakeThreshold || Math.abs(acc.z) > shakeThreshold) {
        $("shake-status").text = "状态: 摇晃中!";
        $device.taptic(0); // 轻微震动反馈
      } else {
        $("shake-status").text = "状态: 静止";
      }
    }
  });
  $ui.toast("开始监听运动数据...");
}

function stopMotionTracking() {
  if (motionIntervalId) {
    $motion.stopUpdates();
    motionIntervalId = null;
    $ui.toast("停止监听运动数据。");
    $("shake-status").text = "状态: 已停止";
  }
}

$ui.render({
  props: { title: "运动传感器" },
  views: [
    {
      type: "label", props: { text: "加速度计数据:" },
      layout: make => make.top.left.inset(20)
    },
    {
      type: "label", props: { id: "accel-x", text: "X: 0.000" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(5).left.inset(20)
    },
    {
      type: "label", props: { id: "accel-y", text: "Y: 0.000" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(5).left.inset(20)
    },
    {
      type: "label", props: { id: "accel-z", text: "Z: 0.000" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(5).left.inset(20)
    },
    {
      type: "label", props: { id: "shake-status", text: "状态: 未开始" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(20).centerX.equalTo(view.super)
    },
    {
      type: "button", props: { title: "开始追踪" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(20).centerX.equalTo(view.super).width.equalTo(120),
      events: { tapped: startMotionTracking }
    },
    {
      type: "button", props: { title: "停止追踪" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).centerX.equalTo(view.super).width.equalTo(120),
      events: { tapped: stopMotionTracking }
    }
  ]
});
```

#### 2. 停止监听运动数据：`$motion.stopUpdates()`

-   **用途**: 停止对设备运动数据的持续监听。这对于节省设备电量和系统资源非常重要。在不再需要运动数据时，务必调用此方法。

### 总结

`$motion` API 为你的 JSBox 脚本提供了访问设备运动传感器数据的能力，从而能够实现对设备物理运动的感知和响应。在开发过程中，请务必注意合理设置 `interval` 和及时调用 `stopUpdates()`，以优化电池续航和应用性能。 
