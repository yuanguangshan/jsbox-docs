# 运行在 Widget 上的扩展

由于 Widget 的交互限制，并不是所有的扩展都可以运行在 Widget 之上，以下是不支持运行在 Widget 的 API 黑名单：

API | 说明
---|---
$ui.action | 弹出 action sheet
$ui.preview | 预览文件或网页
$message.* | 消息相关接口
$photo.take | 拍照
$photo.pick | 选取图片
$photo.prompt | 询问用户获得图片
$share.sheet | 弹出 share sheet
$text.lookup | 查询词典
$picker.* | 选择器
$qrcode.scan | 扫描二维码
$input.speech | 语音识别

当扩展中包含上述接口时，在通知中心运行可能会出现问题。

当你需要进行文字输入时，请尽可能使用 `$input.text` 接口，而不是通过 input 控件。

# Widget 上支持的交互

尽管 Widget 上支持的交互很有限，但对于 `$ui` 还是有少量的接口可以用于提示用户：

API | 说明
---|---
$ui.alert | 弹出 alert
$ui.menu | 弹出菜单
$ui.toast | 显示消息
$ui.loading | 显示加载状态
$input.text | 输入文字

---

## 文件内容解读与示例

### 用途说明

本文档是为 JSBox 开发 **今日视图小组件（Today Widget）** 的关键指南。它明确了在 Widget 这种特殊环境中运行的**核心限制**和**可用功能**。Widget 提供了一种从锁屏或主屏幕快速访问信息和轻量级交互的方式，但其运行环境受到 iOS 系统的严格限制。

### 核心概念：受限的运行环境

Widget 的首要设计目标是**快速提供信息**，而不是完整的应用体验。因此，iOS 对其施加了诸多限制，以保证系统性能和流畅度。理解这些限制是成功开发 Widget 的前提。

-   **禁止长时间运行和高资源消耗**: Widget 的生命周期非常短暂，且内存和 CPU 使用都受到严格控制。
-   **有限的交互**: Widget 的交互模型非常受限。不允许弹出新的视图控制器（View Controller）或进行复杂的界面跳转。所有交互都应是轻量且即时的。

### API 黑名单：哪些不能用？

文档的核心内容是一个 **API 黑名单**，列出了所有**不应该**在 Widget 中使用的 JSBox API。这些 API 的共同点是它们会尝试呈现复杂的、模态的或需要长时间等待用户输入的界面，这在 Widget 环境中是不被允许的。

-   **模态 UI**: `$ui.action`, `$ui.preview`, `$share.sheet`, `$picker.*` 等都会尝试弹出一个全新的界面，这会破坏 Widget 的体验，因此被禁用。
-   **硬件访问与系统服务**: `$photo.take` (相机), `$qrcode.scan` (相机), `$input.speech` (麦克风) 等需要访问敏感硬件或启动常驻服务的接口被禁用。
-   **消息服务**: `$message.*` (iMessage/SMS) 接口被禁用。

**关键建议**: 如果需要在 Widget 中获取用户文本输入，应使用 `$input.text`，它会弹出一个简单的、系统标准的输入框。不要使用 UI 组件中的 `type: "input"`，因为它是一个功能更完整的视图，不适合在 Widget 中独立使用。

### 可用的交互 API

尽管限制很多，JSBox 仍然提供了一些轻量级的交互方式，让你可以在 Widget 中给予用户反馈或提供简单选项：

-   `$ui.alert`: 可以弹出一个标准的系统警告框。
-   `$ui.menu`: 可以弹出一个简单的列表菜单供用户选择。
-   `$ui.toast` / `$ui.loading`: 可以显示非阻塞的提示信息或加载状态。
-   `$input.text`: 如上所述，用于获取文本输入。

### 示例：一个简单的剪贴板工具 Widget

这个 Widget 将显示当前剪贴板的内容，并提供一个“清除”按钮。

```javascript
// Widget: Clipboard Manager

function showContent() {
  const text = $clipboard.text || "剪贴板为空";
  $("contentLabel").text = text;
}

$ui.render({
  props: {
    title: "剪贴板管理"
  },
  views: [
    {
      type: "label",
      props: {
        id: "contentLabel",
        lines: 0, // 允许多行显示
        font: $font(14)
      },
      layout: (make, view) => {
        make.left.top.right.inset(10);
      }
    },
    {
      type: "button",
      props: {
        title: "清除剪贴板"
      },
      layout: (make, view) => {
        make.top.equalTo($("contentLabel").bottom).offset(10);
        make.centerX.equalTo(view.super);
        make.bottom.inset(10); // 决定 Widget 高度
      },
      events: {
        tapped: async () => {
          // 使用 $ui.alert 进行确认，这是 Widget 中允许的交互
          const { index } = await $ui.alert({
            title: "确认",
            message: "要清除剪贴板吗？",
            actions: [ { title: "确定" }, { title: "取消" } ]
          });

          if (index === 0) {
            $clipboard.clear();
            showContent(); // 更新显示
            $ui.toast("已清除"); // 给出反馈
          }
        }
      }
    }
  ]
});

// 初始化显示
showContent();
```

### 总结

开发 JSBox Widget 的关键在于**“做减法”**。你需要时刻牢记其运行环境的限制，避免使用黑名单中的 API，并选择最轻量、最直接的方式来展示信息和处理交互。通过合理利用允许的少数交互接口，你依然可以创造出非常实用和便捷的 Widget。