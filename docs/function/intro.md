# 内建函数

除了标准的 JavaScript 内建函数，例如 encodeURI, setTimeout 等，JSBox 还提供了很多内建函数，这些函数可以直接在全局使用。

例如：

```js
const text = $l10n("MAIN_TITLE");
```

可以获得一个本地化的字符串。

为了与 JavaScript 内置的函数有所区分，JSBox 提供的内建函数都以 `$` 开头。

---

## 文件内容解读与示例

### 用途说明：JSBox 的“超能力”函数

本文档是“内建函数”章节的**引言**。它解释了 JSBox 如何在标准的 JavaScript 运行时之上，提供一套独特的、以 `$` 符号开头的**全局函数**。这些函数是 JSBox 的“超能力”，它们封装了与 iOS 系统、JSBox 应用本身以及各种底层服务进行交互的复杂逻辑，使得开发者能够用简洁的 JavaScript 代码实现强大的原生功能。

### 核心概念：`$` 前缀的意义

-   **区分标准 JS 函数**: 为了避免与 JavaScript 语言本身或浏览器环境中的内置函数（如 `setTimeout`, `encodeURI`, `alert`）发生命名冲突，JSBox 约定所有其提供的特殊内建函数都以 `$` 符号作为前缀（例如 `$ui`, `$http`, `$file`）。
-   **全局可用**: 这些函数是全局可用的，这意味着你可以在脚本的任何地方直接调用它们，无需 `import` 或 `require`。
-   **桥接原生能力**: 每个 `$` 函数都代表着对 iOS 某个原生功能或服务的封装。它们将复杂的原生 API 调用简化为易于理解和使用的 JavaScript 接口。

### 示例：`$` 函数的组合使用

下面的例子展示了如何组合使用几个 `$ `函数来完成一个简单的任务：获取设备信息，然后弹出一个提示，并在延时后播放一个声音。

```javascript
// 1. 获取设备信息：使用 $device 模块
const deviceModel = $device.info.model; // 获取设备型号
const osVersion = $device.info.version; // 获取 iOS 版本

// 2. 显示 UI 提示：使用 $ui 模块
$ui.alert({
  title: "设备信息",
  message: `你的设备型号是：${deviceModel}\n运行 iOS 版本：${osVersion}`,
  actions: [
    { title: "好的" }
  ]
});

// 3. 延时执行任务：使用 $delay 函数
$delay(2, () => {
  // 4. 播放声音：使用 $audio 模块
  // 假设你的脚本 assets 目录下有一个名为 "ding.mp3" 的声音文件
  // $audio.play("local://assets/ding.mp3");
  $ui.toast("声音已播放 (如果文件存在)！");
});

console.log("脚本执行完毕，请等待延时任务。");
```

**代码解读**：

-   `$device.info.model` 和 `$device.info.version` 演示了如何通过 `$device` 模块获取设备信息。
-   `$ui.alert` 演示了如何通过 `$ui` 模块显示一个原生的提示框。
-   `$delay` 演示了如何延迟执行一段代码。
-   `$audio.play` 演示了如何通过 `$audio` 模块播放本地声音文件。

这个例子虽然简单，但它清晰地展示了 `$ `函数如何作为 JSBox 脚本与 iOS 系统交互的入口，让你可以用 JavaScript 轻松地调用各种原生功能。

### 总结

“内建函数”是 JSBox 强大功能的核心体现。它们是连接 JavaScript 逻辑与 iOS 原生能力的桥梁。在学习 JSBox 的过程中，你将不断地接触和使用这些以 `$` 开头的函数，它们将是你实现各种自动化和应用功能的关键工具。 
