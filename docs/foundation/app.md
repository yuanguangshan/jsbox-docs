> 和 JSBox 本身相关或和正在运行的脚本相关的接口

# $app.theme

为脚本指定 `theme`，用于 [Dark Mode](uikit/dark-mode.md) 相关，可选值为 `light` / `dark` / `auto`。

如果某个页面指定了 `theme`，则这个全局的值会被覆盖。

# $app.minSDKVer

指定此扩展可用的最低 SDK 版本（SDK 版本即 JSBox 的版本）：

```js
$app.minSDKVer = "3.1.0"
```

# $app.minOSVer

指定此扩展可用的最低 iOS 版本：

```js
$app.minOSVer = "10.3.3"
```

注：版本号都是小数点分割的两个数字或三个数字，从前往后依次比较大小。

# $app.tips(string)

给用户展示一个使用提示，效果如同 `$ui.alert`，但请注意这个提示只会运行一次：

```js
$app.tips("在 App Store 内通过分享面板使用")
```

# $app.info

返回 app 本身的信息，例如：

```js
{
  "bundleID": "app.cyan.jsbox",
  "version": "3.0.0",
  "build": "9527",
}
```

# $app.idleTimerDisabled

设置成 true 时屏幕不会自动休眠：

```js
$app.idleTimerDisabled = true
```

# $app.close(delay)

关闭当前的扩展，请注意扩展被关闭之后的代码都不会再被执行。

`delay` 表示延迟的秒数，可以缺省，缺省即立即关闭。

> 注：当一个扩展没有界面时，建议在逻辑完成之后手动调用 $app.close()，这样可以把引擎关闭。

# $app.isDebugging

检查当前是否在调试状态：

```js
if ($app.isDebugging) {
  
}
```

# $app.env

获得此扩展当前运行的环境：

```js
const env = $app.env;
```

参数 | 说明
---|---
$env.app | 主应用
$env.today | 通知中心小组件
$env.action | Action 扩展
$env.safari | Safari 扩展
$env.notification | 通知
$env.keyboard | 键盘扩展
$env.siri | Siri 环境
$env.widget | 桌面小组件
$env.all | 所有环境（默认值）

我们可以如此来判断扩展是否运行在通知中心：

```js
if ($app.env == $env.today) {

}
```

# $app.widgetIndex

因为 JSBox 支持多个小组件，你可以通过这个值判断哪个小组件正在被使用：

```js
const index = $app.widgetIndex;

// 0 ~ 2，其他值表示不是小组件
```

# $app.autoKeyboardEnabled

在某些滚动列表里面，可能会出现键盘遮挡住输入框的情况，开启之后将会自动避免这个问题：

```js
$app.autoKeyboardEnabled = true
```

# $app.keyboardToolbarEnabled

同样在滚动列表里，为键盘展示一个工具栏将会让输入更加方便，尤其是对于多个输入框：

```js
$app.keyboardToolbarEnabled = true
```

# $app.rotateDisabled

设置为 true 时屏幕将不可以旋转：

```js
$app.rotateDisabled = true
```

# $app.openURL(string)

打开一个 URL，例如打开微信：

```js
$app.openURL("weixin://")
```

# $app.openBrowser(object)

在用户安装了某个浏览器的情况下，通过某个浏览器打开 URL，例如：

```js
$app.openBrowser({
  type: 10000,
  url: "https://apple.com"
})
```

type | 浏览器
---|---
10000 | Chrome
10001 | UC
10002 | Firefox
10003 | QQ
10004 | Opera
10005 | Quark
10006 | iCab
10007 | Maxthon
10008 | Dolphin
10009 | 2345

> 由于各浏览器频繁改动其接口，上述接口并不保证能正确运行。

# $app.openExtension(string)

打开另一个已安装的 JSBox 脚本，例如：

```js
$app.openExtension("demo.js")
```

# $app.listen(object)

监听消息，目前支持 `ready` 和 `exit` 等：

```js
$app.listen({
  // 在应用启动之后调用
  ready: function() {

  },
  // 在应用退出之前调用
  exit: function() {
    
  },
  // 在应用停止响应后调用
  pause: function() {

  },
  // 在应用恢复响应后调用
  resume: function() {

  }
});
```

# $app.notify(object)

发出一个自定义的消息，可被 listen 接收：

```js
$app.listen({
  eventName: function(object) {
    console.log(object);
  }
});

$app.notify({
  name: "eventName",
  object: {"a": "b"}
});
```

---

## 文件内容解读与示例

### 用途说明

`$app` API 是你的 JSBox 脚本与**JSBox 应用程序本身**以及**当前脚本运行环境**进行交互的核心接口。它提供了获取应用信息、控制应用行为、与其他应用或脚本进行通信以及监听应用生命周期事件的能力。理解 `$app` 对于编写健壮、智能且能与其他应用无缝协作的 JSBox 脚本至关重要。

### 核心属性详解

#### 1. 应用信息与环境

-   **`$app.info`**: 返回 JSBox 应用本身的元数据，如 `bundleID`、`version`（版本号）和 `build`（构建号）。
-   **`$app.env`**: 获取当前脚本的运行环境（例如，是在主应用中、Today Widget 中还是 Action Extension 中）。这通常与 `$env` 常量配合使用，是编写上下文感知脚本的关键。
-   **`$app.minSDKVer` / `$app.minOSVer`**: 用于声明脚本运行所需的最低 JSBox 版本和最低 iOS 版本。通常在脚本的 `info.json` 文件中配置。
-   **`$app.isDebugging`**: 判断当前脚本是否处于调试模式。
-   **`$app.widgetIndex`**: 当脚本作为多个 Today Widget 或桌面小组件运行时，用于区分是哪个小组件实例在运行。

#### 2. 应用行为控制

-   **`$app.theme`**: 设置或获取当前脚本界面的主题模式（`light` / `dark` / `auto`），用于适配深色模式。
-   **`$app.idleTimerDisabled`**: 设置为 `true` 时，可以防止设备屏幕在脚本运行时自动休眠。适用于需要长时间显示或处理的脚本。
-   **`$app.rotateDisabled`**: 设置为 `true` 时，可以禁用当前脚本界面的屏幕旋转。
-   **`$app.autoKeyboardEnabled` / `$app.keyboardToolbarEnabled`**: 优化键盘行为，前者自动避免键盘遮挡输入框，后者为键盘提供工具栏。

### 核心方法详解

#### 1. 脚本生命周期与提示

-   **`$app.close(delay)`**: 关闭当前正在运行的脚本或扩展。对于没有 UI 的后台脚本，建议在任务完成后调用此方法，以释放资源。`delay` 参数可以延迟关闭。
-   **`$app.tips(string)`**: 显示一个一次性的提示信息给用户，通常用于首次使用时的引导。

#### 2. 应用间与脚本间交互

-   **`$app.openURL(string)`**: 打开一个 URL。可以是网页链接，也可以是其他应用的 URL Scheme（如 `weixin://`）。
-   **`$app.openBrowser(options)`**: 尝试通过用户安装的特定第三方浏览器打开 URL。**注意：此接口的可靠性不保证，因为第三方浏览器可能随时更改其接口。**
-   **`$app.openExtension(string)`**: 启动另一个已安装的 JSBox 脚本。这是实现脚本之间功能复用和流程串联的强大方式。

#### 3. 应用内通信与事件监听

-   **`$app.listen(object)`**: 监听 JSBox 应用级别的事件。你可以监听预定义的生命周期事件（`ready` 应用启动后、`exit` 应用退出前、`pause` 应用暂停响应、`resume` 应用恢复响应），也可以监听自定义事件。
-   **`$app.notify(object)`**: 发送一个自定义的应用级别消息。这个消息可以被任何正在运行的、监听了该事件的脚本接收。这是实现复杂脚本间通信的机制。

### 示例代码：环境判断与应用交互

下面的示例将演示如何判断当前运行环境，并使用 `$app` 的方法进行应用间交互和自定义消息通知。

```javascript
// 1. 监听应用生命周期事件和自定义消息
$app.listen({
  ready: () => {
    console.log("应用已启动！");
    // 判断当前运行环境
    if ($app.env === $env.app) {
      $("status-label").text = "当前运行环境: 主应用";
    } else if ($app.env === $env.action) {
      $("status-label").text = "当前运行环境: 分享扩展";
    } else if ($app.env === $env.today) {
      $("status-label").text = "当前运行环境: 今日小组件";
    }
    console.log("JSBox 版本:", $app.info.version, "构建号:", $app.info.build);
  },
  exit: () => {
    console.log("应用即将退出...");
  },
  // 监听自定义事件
  customAppEvent: (data) => {
    $ui.alert(`收到自定义消息: ${data.message}`);
  }
});

$ui.render({
  props: { title: "$app API 示例" },
  views: [
    {
      type: "label",
      props: {
        id: "status-label",
        text: "正在检测运行环境...",
        align: $align.center,
        lines: 0
      },
      layout: make => {
        make.top.left.right.inset(20);
        make.height.equalTo(60);
      }
    },
    {
      type: "button",
      props: { title: "打开 Apple 官网" },
      layout: make => {
        make.top.equalTo(view.prev.bottom).offset(20);
        make.centerX.equalTo(view.super);
        make.width.equalTo(200);
      },
      events: {
        tapped: () => {
          $app.openURL("https://www.apple.com");
        }
      }
    },
    {
      type: "button",
      props: { title: "打开另一个 JSBox 脚本" },
      layout: make => {
        make.top.equalTo(view.prev.bottom).offset(10);
        make.centerX.equalTo(view.super);
        make.width.equalTo(200);
      },
      events: {
        tapped: () => {
          // 假设你有一个名为 "Hello World.js" 的脚本
          $app.openExtension("Hello World.js");
        }
      }
    },
    {
      type: "button",
      props: { title: "发送自定义消息" },
      layout: make => {
        make.top.equalTo(view.prev.bottom).offset(10);
        make.centerX.equalTo(view.super);
        make.width.equalTo(200);
      },
      events: {
        tapped: () => {
          $app.notify({
            name: "customAppEvent",
            object: { message: "你好，我是来自 $app.notify 的消息！" }
          });
        }
      }
    }
  ]
});
```

**代码解读**：

1.  脚本启动时，`$app.listen` 中的 `ready` 事件会立即触发，我们在这里判断了当前脚本的运行环境，并显示了 JSBox 的版本信息。
2.  “打开 Apple 官网”按钮演示了如何使用 `$app.openURL` 来启动外部应用或打开网页。
3.  “打开另一个 JSBox 脚本”按钮演示了 `$app.openExtension`，它允许你从当前脚本启动另一个脚本，实现脚本间的流程跳转。
4.  “发送自定义消息”按钮演示了 `$app.notify`。它发送了一个名为 `customAppEvent` 的消息，这个消息会被 `$app.listen` 中定义的同名处理器捕获，并弹出一个提示框。

`$app` API 是 JSBox 脚本与应用本身和 iOS 系统进行深度交互的枢纽，掌握它能让你编写出更强大、更智能的自动化脚本。