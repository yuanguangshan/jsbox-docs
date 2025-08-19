# 2021.11.26

- 改进配置项相关接口：[详情](foundation/prefs.md)
- 改进菜单相关接口：[详情](uikit/context-menu.md)
- 增加 Safari 扩展文档：[详情](safari-extension/intro.md)
- 增加钥匙串相关接口：[详情](foundation/keychain.md)

# 2020.10.03

- 增加桌面小组件相关：[详情](home-widget/intro.md)

# 2020.03.13

- 增加 Dark Mode 相关：[详情](uikit/dark-mode.md)

# 2020.03.04

- 增加富文本相关：[详情](component/text.md)

# 2020.01.29

- 增加图像处理相关：[详情](media/imagekit.md)

# 2020.01.28

- 添加关于内置模块的描述：[详情](package/builtin.md)

# 2019.12.28

- 增加代码视图相关：[详情](component/code.md)

# 2019.11.25

- 增加视图层级相关：[详情](uikit/view.md)

# 2019.11.21

- 增加 context menu 支持：[详情](uikit/context-menu.md)

# 2019.10.27

- $text.speech 增加语音设置：[详情](extend/text.md)

# 2019.09.02

- 增加应用配置相关：[详情](foundation/prefs.md)

# 2018.12.23

- 增加编辑器插件相关：[详情](extend/editor.md)

# 2018.11.09

- 增加 Apple Pencil 相关：[详情](uikit/view.md)

# 2018.10.21

- 增加 XML 相关：[详情](extend/xml.md)

# 2018.10.05

- 增加 Lottie View 相关：[详情](component/lottie.md)

# 2018.08.26

- 添加 Web Socket 相关：[详情](network/socket.md)
- 添加 Web Server 相关：[详情](network/server.md)

# 2018.08.11

- 增加 SQLite 相关：[详情](sqlite/intro.md)

# 2018.07.29

- 增加 Shortcuts 相关：[详情](shortcuts/intro.md)

# 2018.07.22

- 增加内建函数索引：[详情](function/intro.md)

# 2018.07.18

- $contact 增加分组相关操作：[详情](sdk/contact.md)

# 2018.07.08

- Runtime 语法糖：[详情](runtime/sugar.md)

# 2018.06.27

- matrix 增加长按排序：[详情](component/matrix.md)
- matrix 增加 itemSize 方法：[详情](component/matrix.md)
- matrix 和 list 增加 forEachItem 方法：[详情](component/matrix.md)

# 2018.06.17

- $audio.play 增加 events: [详情](media/audio.md)

# 2018.06.14

- 增加 $widget 接口文档：[详情](widget/method.md)

# 2018.06.08

- 增加对 Lua 虚拟机的文档：[详情](vm/lua.md)

# 2018.05.15

- 完善 Runtime 相关文档：[详情](runtime/blocks.md)
- 增加 C 函数相关文档：[详情](runtime/c.md)

# 2018.04.18

- 添加 Promise 相关文档：[详情](promise/intro.md)

# 2018.04.07

- 添加 SSH 相关接口：[详情](ssh/intro.md)

# 2018.04.06

- 添加 markdown 相关接口：[详情](extend/text.md)

# 2018.04.01

- 添加 view debugging 文档：[详情](uikit/render.md)

# 2018.03.25

- 添加 ifa_data 相关文档：[详情](foundation/network.md)
- 添加 ping 相关文档：[详情](foundation/network.md)

# 2018.03.18

- 添加关于键盘的文档：[详情](keyboard/method.md)

# 2018.03.11

- 添加 $location.select 接口

# 2018.03.09

- 为 $push 接口添加新内容：[详情](extend/push.md)
- 添加 $http.startServer 相关内容：[详情](foundation/network.md)
- $text.speech 新增语言

# 2018.02.28

- 新增安装包格式：[详情](package/intro.md)

# 2017.07.15

- 初始版本

---

## 文件内容解读

### 文件用途

本文档是一个 **更新日志 (Changelog)**，它按照时间倒序记录了 JSBox 应用和相关文档的所有重要更新。通过此文件，开发者和用户可以清晰地了解到每个版本的新增功能、改进内容以及修复的问题。这对于跟进应用发展、学习新特性以及在开发中利用最新功能至关重要。

### 内容结构

该日志的结构非常直观：

- **日期标题**: 每个更新都以 `YYYY.MM.DD` 格式的日期作为标题，代表更新发布的大致时间。
- **更新列表**: 在每个日期下方，使用无序列表（`-`）列出具体的更新条目。
- **详情链接**: 大部分条目后面都附有一个 `[详情](...)` 链接，点击后可以跳转到该功能更详尽的说明文档，方便用户深入学习。

### 重点功能展开说明

下面对日志中提到的一些重要或较复杂的更新点进行展开说明，并提供示例。

#### 1. 桌面小组件 (Home Widget) - 2020.10.03

这是 JSBox 的一个里程碑式的功能，它允许用户使用 JavaScript 来创建 iOS 主屏幕上的小组件。这意味着你可以将脚本的输出以图形化、可定制的视图展示在桌面上，用于信息展示、快捷启动等。

**难点解读**: 制作小组件的核心在于“渲染”和“时间线”。你不能在小组件里运行复杂的持续性任务，而是需要提供一个“快照”视图。JSBox 通过 `$widget` API 极大地简化了这一过程。

**示例**: 创建一个简单的小组件，显示当前日期。

```javascript
// 一个简单的小组件脚本
$widget.setTimeline({
  render: ctx => {
    // ctx.family 可以判断小组件的尺寸 (small, medium, large)
    return {
      type: "text",
      props: {
        text: new Date().toLocaleDateString(),
        font: $font("bold", 20),
        color: $color("primaryText")
      }
    }
  }
});
```
这个脚本定义了如何渲染小组件的视图，JSBox 会根据系统调度来更新它。

#### 2. Lottie View - 2018.10.05

Lottie 是 Airbnb 开源的一个动画库，可以实时渲染 After Effects 动画。JSBox 集成了 Lottie，让开发者可以轻松地在自己的脚本界面中加入高质量、复杂的矢量动画，而无需手动编写复杂的动画代码。

**难点解读**: 传统上，在 UI 中实现复杂动画（例如，一个人物的行走动画）是非常困难的。Lottie 将这个过程简化为“设计师导出 JSON 文件，开发者加载 JSON 文件”。你需要一个 Lottie 动画的 JSON 文件，可以从 [LottieFiles](https://lottiefiles.com/) 等社区找到。

**示例**: 加载并播放一个网络上的 Lottie 动画。

```javascript
$ui.render({
  views: [
    {
      type: "lottie",
      props: {
        // 从网络加载一个动画资源
        src: "https://assets2.lottiefiles.com/packages/lf20_Lpuvp8.json",
        loop: true, // 循环播放
        autoPlay: true // 自动播放
      },
      layout: $layout.fill
    }
  ]
});
```

#### 3. Runtime 与 C 函数 - 2018.05.15

这是 JSBox 中非常高级和强大的部分，主要面向有原生开发经验的用户。

- **Runtime**: 指的是 Objective-C 的运行时特性。JSBox 允许你通过 `$objc` 等接口直接与 iOS 底层的 Objective-C API 交互。你可以创建、调用原生对象和方法，实现许多标准 JavaScript 无法做到的事情。
- **C 函数**: 甚至可以调用 iOS 系统中的 C 语言函数。

**难点解读**: 这部分功能的难点在于，它要求开发者对 Objective-C/Swift 的 API、内存管理（如指针、引用计数）有深入的了解。错误的使用可能导致应用闪退或行为异常。它是一把双刃剑，提供了无限可能性的同时也带来了更高的复杂度。

**示例**: （概念性）使用 Runtime 获取当前设备的名称。

```javascript
// 获取 UIDevice 的单例
const device = $objc("UIDevice").$currentDevice();
// 调用 name 属性
const deviceName = device.$name().jsValue(); // 通过 .jsValue() 转换为 JS 字符串

console.log(deviceName);
```
这个例子展示了如何通过 `$objc` 调用原生代码，这比 JSBox 提供的 `$device.info.name` 要底层得多，但原理是相通的。

通过这个更新日志，我们可以看到 JSBox 从一个简单的脚本运行器，逐步成长为一个功能全面、扩展性极强的自动化和开发工具。