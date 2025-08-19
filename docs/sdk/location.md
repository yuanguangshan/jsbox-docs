> 用于与 GPS 模块进行交互，例如获取位置或追踪位置

# $location.available

检查位置服务是否可用：

```js
let available = $location.available;
```

# $location.fetch(object)

获取地理位置：

```js
$location.fetch({
  handler: function(resp) {
    const lat = resp.lat;
    const lng = resp.lng;
    const alt = resp.alt;
  }
})
```

# $location.startUpdates(object)

监听地理位置变化：

```js
$location.startUpdates({
  handler: function(resp) {
    const lat = resp.lat;
    const lng = resp.lng;
    const alt = resp.alt;
  }
})
```

# $location.trackHeading(object)

监听罗盘数据变化：

```js
$location.trackHeading({
  handler: function(resp) {
    const magneticHeading = resp.magneticHeading;
    const trueHeading = resp.trueHeading;
    const headingAccuracy = resp.headingAccuracy;
    const x = resp.x;
    const y = resp.y;
    const z = resp.z;
  }
})
```

# $location.stopUpdates()

停止更新。

# $location.select(object)

从 iOS 内置的地图上选取一个点：

```js
$location.select({
  handler: function(result) {
    const lat = result.lat;
    const lng = result.lng;
  }
})
```

# $location.get()

获取当前位置，和 $location.fetch 类似但使用 async await 实现。

```js
const location = await $location.get();
```

# $location.snapshot(object)

生成地图的一个截图：

```js
const loc = await $location.get();
const lat = loc.lat;
const lng = loc.lng;
const snapshot = await $location.snapshot({
  region: {
    lat,
    lng,
    // distance: 10000
  },
  // size: $size(256, 256),
  // showsPin: false,
  // style: 0 (0: unspecified, 1: light, 2: dark)
});
```

---

## 文件内容解读与示例

### 用途说明

`$location` API 提供了与 iOS 设备**地理位置服务（GPS）**和**罗盘数据**进行交互的能力。它允许你的脚本获取设备的当前位置、持续追踪位置变化、获取设备朝向，甚至让用户在地图上选择一个地点或生成地图快照。这对于构建基于位置的服务、导航辅助、地理围栏应用或任何需要地理信息的脚本都非常有用。

### 核心概念：权限与异步操作

-   **权限**: 所有位置服务都需要用户授权。首次调用相关接口时，系统会弹出权限请求。你的脚本应妥善处理用户拒绝授权的情况。
-   **异步**: 大多数 `$location` 操作都是异步的，结果通过 `handler` 回调或 `await` Promise 返回。这确保了 UI 在等待用户操作或处理位置数据时不会卡顿。

### 主要功能与方法详解

#### 1. 位置服务可用性

-   **`$location.available`**: 布尔值，检查设备的位置服务是否可用（例如，用户是否已开启 GPS）。

#### 2. 获取地理位置

-   **`$location.fetch(options)`**: 获取设备当前的地理位置（经纬度、海拔）。使用回调函数处理结果。
-   **`$location.get()`**: 类似于 `fetch`，但返回 Promise，推荐与 `async/await` 配合使用。
    -   **返回**: `resp` 对象，包含 `lat`（纬度）、`lng`（经度）、`alt`（海拔）。

**示例**：获取当前位置并显示

```javascript
async function getCurrentLocation() {
  if (!$location.available) {
    $ui.alert("位置服务不可用，请在系统设置中开启。");
    return;
  }
  $ui.toast("正在获取位置...");
  try {
    const loc = await $location.get();
    $ui.alert(`当前位置:\n纬度: ${loc.lat}\n经度: ${loc.lng}\n海拔: ${loc.alt} 米`);
  } catch (error) {
    $ui.alert(`获取位置失败: ${error.localizedDescription || error}`);
  }
}
// getCurrentLocation();
```

#### 3. 持续追踪位置与罗盘

-   **`$location.startUpdates(options)`**: 开始持续监听设备位置的变化。`handler` 会在每次位置更新时被调用。
-   **`$location.trackHeading(options)`**: 开始持续监听设备的朝向（罗盘数据）。`handler` 会在每次朝向更新时被调用。
-   **`$location.stopUpdates()`**: 停止所有位置和朝向的更新，以节省电量。

**示例**：持续显示位置和朝向

```javascript
let locationUpdateTimer = null;
let headingUpdateTimer = null;

function startTracking() {
  if (!$location.available) {
    $ui.alert("位置服务不可用。");
    return;
  }

  $location.startUpdates({
    handler: (resp) => {
      $("location-label").text = `位置: ${resp.lat.toFixed(4)}, ${resp.lng.toFixed(4)}`;
    }
  });

  $location.trackHeading({
    handler: (resp) => {
      $("heading-label").text = `朝向: ${resp.trueHeading.toFixed(1)}°`;
    }
  });

  $ui.toast("开始追踪位置和朝向...");
}

function stopTracking() {
  $location.stopUpdates();
  $ui.toast("停止追踪。");
}

$ui.render({
  props: { title: "位置追踪" },
  views: [
    {
      type: "label", props: { id: "location-label", text: "位置: 未知" },
      layout: make => make.top.left.right.inset(20)
    },
    {
      type: "label", props: { id: "heading-label", text: "朝向: 未知" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.right.inset(20)
    },
    {
      type: "button", props: { title: "开始追踪" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(20).centerX.equalTo(view.super).width.equalTo(120),
      events: { tapped: startTracking }
    },
    {
      type: "button", props: { title: "停止追踪" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).centerX.equalTo(view.super).width.equalTo(120),
      events: { tapped: stopTracking }
    }
  ]
});
```

#### 4. 用户在地图上选择地点

-   **`$location.select(options)`**: 弹出 iOS 内置地图界面，让用户手动选择一个地点。返回 Promise，解析为包含 `lat` 和 `lng` 的结果对象。

**示例**：

```javascript
async function selectLocationOnMap() {
  try {
    const result = await $location.select();
    $ui.alert(`你选择了:\n纬度: ${result.lat}\n经度: ${result.lng}`);
  } catch (error) {
    $ui.toast("选择地点取消或失败。");
  }
}
// selectLocationOnMap();
```

#### 5. 生成地图快照

-   **`$location.snapshot(options)`**: 根据指定的区域和尺寸，生成一张地图的静态图片。返回 Promise，解析为 `$image` 对象。
    -   `region`: 定义地图区域，包含 `lat`, `lng` 和 `distance`（距离）或 `lat_span`, `lng_span`（经纬度范围）。
    -   `size`: 快照的尺寸。
    -   `showsPin`: 是否在中心显示大头针。

**示例**：生成当前位置的地图快照

```javascript
async function generateMapSnapshot() {
  const loc = await $location.get();
  if (loc) {
    $ui.toast("正在生成地图快照...");
    const snapshot = await $location.snapshot({
      region: { lat: loc.lat, lng: loc.lng, distance: 1000 }, // 以当前位置为中心，1公里范围
      size: $size(300, 200),
      showsPin: true, // 显示大头针
      style: 1 // Light 模式
    });
    if (snapshot) {
      $ui.alert({ title: "地图快照", image: snapshot });
    } else {
      $ui.alert("地图快照生成失败。");
    }
  } else {
    $ui.toast("无法获取当前位置。");
  }
}
// generateMapSnapshot();
```

### 总结

`$location` API 为你的 JSBox 脚本提供了与设备地理位置和罗盘功能进行深度交互的能力。它对于构建位置感知应用、导航辅助工具或任何需要地理信息的脚本都至关重要。 
