> 在这里可以获取和设备有关的一些信息，例如设备语言、设备型号等等。

# $device.info

返回设备的基本信息，例如：

```js
{
  "model": "string",
  "language": "string",
  "version": "string",
  "name": "cyan's iPhone",
  "screen": {
    "width": 240,
    "height": 320,
    "scale": 2.0,
    "orientation": 1,
  },
  "battery": {
    "state": 1, // 0: unknown 1: normal 2: charging 3: charging & fully charged
    "level": 0.9399999976158142
  }
}
```

# $device.ssid

获取当前 Wi-Fi 的 SSID 信息：

```js
const ssid = $device.ssid;
```

返回数据样例：

```json
{
  "SSIDDATA": {},
  "BSSID": "aa:bb:cc:dd:ee:ff",
  "SSID": "SSID"
}
```

注：在 iOS 13 上，使用此接口需要应用具备地理位置权限，你可以通过 `$location` 相关接口获得权限。

# $device.networkType

获取当前设备的网络类型：

```js
const networkType = $device.networkType;
```

数值 | 说明
---|---
0 | 无网络
1 | Wi-Fi
2 | 蜂窝数据

# $device.space

获取设备的内存/磁盘空间：

```js
const space = $device.space;
```

返回数据样例：

```json
{
  "disk": {
    "free": {
      "bytes": 87409733632,
      "string": "87.41 GB"
    },
    "total": {
      "bytes": 127989493760,
      "string": "127.99 GB"
    }
  },
  "memory": {
    "free": {
      "bytes": 217907200,
      "string": "207.8 MB"
    },
    "total": {
      "bytes": 3221225472,
      "string": "3 GB"
    }
  }
}
```

# $device.taptic(number)

在有 Taptic Engine 的设备上触发一个轻微的振动，例如：

```js
$device.taptic(0)
```

参数 | 类型 | 说明
---|---|---
level | number | 0 ~ 2 表示振动等级

# $device.wlanAddress

获得局域网 IP 地址：

```js
const address = $device.wlanAddress;
```

# $device.isDarkMode

检查设备是否处于 Dark Mode 状态：

```js
if ($device.isDarkMode) {
  
}
```

# $device.isXXX

快速检查设备屏幕类型：

```js
const isIphoneX = $device.isIphoneX;
const isIphonePlus = $device.isIphonePlus;
const isIpad = $device.isIpad;
const isIpadPro = $device.isIpadPro;
```

# $device.hasTouchID

检查是否支持 Touch ID:

```js
const hasTouchID = $device.hasTouchID;
```

# $device.hasFaceID

检查是否支持 Face ID:

```js
const hasFaceID = $device.hasFaceID;
```

# $device.isJailbroken

检查设备是否越狱：

```js
const isJailbroken = $device.isJailbroken;
```

# $device.isVoiceOverOn

检查是否在使用 VoiceOver:

```js
const isVoiceOverOn = $device.isVoiceOverOn;
```

---

## 文件内容解读与示例

### 用途说明

`$device` API 提供了访问 iOS 设备**硬件和系统状态信息**的能力。通过它，你的脚本可以获取设备的型号、操作系统版本、屏幕参数、网络连接状态、电池电量，甚至生物识别能力和越狱状态等。这对于编写能够根据设备特性进行适配、提供个性化体验或进行系统诊断的脚本非常有用。

### 核心属性与方法详解

#### 1. 基本设备信息

-   **`$device.info`**: 返回一个包含设备基本信息的对象，包括：
    -   `model`: 设备型号（如 `iPhone`、`iPad`）。
    -   `language`: 设备语言（如 `zh-Hans`）。
    -   `version`: iOS 版本号（如 `17.5.1`）。
    -   `name`: 设备名称（如 `John's iPhone`）。
    -   `screen`: 屏幕信息，包含 `width`、`height`（点单位）、`scale`（像素比）、`orientation`（方向）。
    -   `battery`: 电池信息，包含 `state`（状态，如充电中、正常）和 `level`（电量，0.0-1.0）。

**示例**：显示设备概览

```javascript
const info = $device.info;
console.log(`设备型号: ${info.model}`);
console.log(`系统版本: iOS ${info.version}`);
console.log(`屏幕分辨率: ${info.screen.width}x${info.screen.height} @${info.screen.scale}x`);
console.log(`电池电量: ${Math.round(info.battery.level * 100)}%`);
```

#### 2. 网络信息

-   **`$device.ssid`**: 获取当前连接的 Wi-Fi 网络的 SSID（名称）和 BSSID。**注意：在 iOS 13 及更高版本上，使用此接口需要应用具备地理位置权限。**
-   **`$device.networkType`**: 获取当前网络连接类型（`0`: 无网络, `1`: Wi-Fi, `2`: 蜂窝数据）。
-   **`$device.wlanAddress`**: 获取设备的局域网 IP 地址。

**示例**：检查网络状态

```javascript
const networkType = $device.networkType;
let status = "";
if (networkType === 0) {
  status = "无网络连接";
} else if (networkType === 1) {
  status = `Wi-Fi: ${$device.ssid ? $device.ssid.SSID : "未知"} (IP: ${$device.wlanAddress})`;
} else if (networkType === 2) {
  status = "蜂窝数据";
}
console.log(`当前网络状态: ${status}`);
```

#### 3. 存储与内存

-   **`$device.space`**: 返回设备磁盘和内存的详细使用情况，包括可用空间和总空间（字节数和格式化后的字符串）。

**示例**：显示存储空间

```javascript
const space = $device.space;
console.log(`可用磁盘空间: ${space.disk.free.string}`);
console.log(`总内存: ${space.memory.total.string}`);
```

#### 4. 设备特性与状态

-   **`$device.taptic(level)`**: 在支持 Taptic Engine 的设备上触发轻微的触觉反馈（震动）。`level` 参数（0-2）控制震动强度。
-   **`$device.isDarkMode`**: 检查设备当前是否处于深色模式。
-   **`$device.isIphoneX` / `isIphonePlus` / `isIpad` / `isIpadPro`**: 快速判断设备的屏幕类型或设备型号。
-   **`$device.hasTouchID` / `$device.hasFaceID`**: 检查设备是否支持指纹识别或面容 ID。
-   **`$device.isJailbroken`**: 检查设备是否已越狱。这对于需要更高安全性的脚本可能有用。
-   **`$device.isVoiceOverOn`**: 检查 VoiceOver 辅助功能是否开启。

**示例**：根据深色模式触发震动

```javascript
if ($device.isDarkMode) {
  $device.taptic(2); // 强震动
  $ui.toast("当前是深色模式");
} else {
  $device.taptic(0); // 轻微震动
  $ui.toast("当前是浅色模式");
}
```

### 总结

`$device` API 是编写智能、自适应脚本的基石。通过获取设备信息，你的脚本可以更好地理解其运行环境，从而提供更符合用户习惯和设备能力的体验。 
