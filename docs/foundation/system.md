> 和 iOS 系统本身相关的接口

# $system.brightness

设置屏幕，例如：

```js
$system.brightness = 0.5
```

# $system.volume

获取或设置设备音量 (0.0 ~ 1.0)：

```js
$system.volume = 0.5
```

# $system.call(number)

拨打电话，效果等同于 `$app.openURL("tel:number")`。

# $system.sms(number)

发送短信，效果等同于 `$app.openURL("sms:number")`。

# $system.mailto(email)

发送邮件，效果等同于 `$app.openURL("mailto:email")`。

# $system.facetime(number)

FaceTime，效果等同于 `$app.openURL("facetime:number")`。

# $system.makeIcon(object)

为链接创建一个桌面图标：

```js
$system.makeIcon({
  title: "Title",
  url: "https://sspai.com",
  icon: image
})
```

---

## 文件内容解读与示例

### 用途说明

`$system` API 提供了与 iOS 操作系统进行**系统级交互**的能力。它允许你的脚本直接控制设备的一些硬件特性（如屏幕亮度、音量），以及触发系统内置的通信功能（如拨打电话、发送短信和邮件）。这些接口通常是 `$app.openURL` 的便捷封装，但提供了更直观的调用方式。

### 核心属性与方法详解

#### 1. 设备控制

-   **`$system.brightness`**: 获取或设置设备的屏幕亮度。值范围是 `0.0`（最暗）到 `1.0`（最亮）。
-   **`$system.volume`**: 获取或设置设备的媒体音量。值范围是 `0.0`（静音）到 `1.0`（最大音量）。

**示例**：通过滑块调节屏幕亮度

```javascript
$ui.render({
  props: { title: "亮度调节" },
  views: [
    {
      type: "label",
      props: { id: "brightness-label", text: `亮度: ${Math.round($system.brightness * 100)}%` },
      layout: make => make.centerX.equalTo(view.super).top.inset(50)
    },
    {
      type: "slider",
      props: {
        min: 0.0,
        max: 1.0,
        value: $system.brightness, // 初始值为当前系统亮度
        continuous: true
      },
      layout: make => {
        make.top.equalTo($("brightness-label").bottom).offset(20);
        make.left.right.inset(20);
      },
      events: {
        changed: sender => {
          $system.brightness = sender.value; // 设置系统亮度
          $("brightness-label").text = `亮度: ${Math.round(sender.value * 100)}%`;
        }
      }
    }
  ]
});
```

#### 2. 通信功能

这些方法本质上是调用 `$app.openURL` 并使用特定的 URL Scheme 来触发系统应用。

-   **`$system.call(number)`**: 拨打指定电话号码。
-   **`$system.sms(number)`**: 向指定电话号码发送短信，会打开短信应用并预填收件人。
-   **`$system.mailto(email)`**: 发送邮件，会打开邮件应用并预填收件人。
-   **`$system.facetime(number)`**: 发起 FaceTime 通话。

**示例**：快速联系

```javascript
$ui.render({
  props: { title: "快速联系" },
  views: [
    {
      type: "button",
      props: { title: "拨打电话 (10086)" },
      layout: make => make.top.left.right.inset(20).height.equalTo(40),
      events: { tapped: () => $system.call("10086") }
    },
    {
      type: "button",
      props: { title: "发送短信 (10086)" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.right.inset(20).height.equalTo(40),
      events: { tapped: () => $system.sms("10086") }
    },
    {
      type: "button",
      props: { title: "发送邮件 (support@jsbox.com)" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.right.inset(20).height.equalTo(40),
      events: { tapped: () => $system.mailto("support@jsbox.com") }
    }
  ]
});
```

#### 3. 桌面图标

-   **`$system.makeIcon(object)`**: 为指定的 URL 创建一个桌面快捷方式图标。用户点击后可以直接打开该 URL。
    -   `title`: 桌面图标下方的标题。
    -   `url`: 点击图标后要打开的 URL。
    -   `icon`: 图标图片，一个 `$image` 对象。

**示例**：创建 JSBox 官网快捷方式

```javascript
// 假设你有一个名为 "jsbox_icon.png" 的图片文件在脚本 assets 目录下
// const jsboxIcon = $image("assets/jsbox_icon.png");

// $system.makeIcon({
//   title: "JSBox 官网",
//   url: "https://www.jsbox.com",
//   icon: jsboxIcon // 替换为你的 $image 对象
// });
// $ui.alert("请在 Safari 中确认添加桌面图标。");
```

### 总结

`$system` API 提供了一系列便捷的方法，用于与 iOS 系统进行直接交互。虽然许多功能可以通过 `$app.openURL` 实现，但 `$system` 模块提供了更具语义化的接口，使代码更清晰易懂。 
