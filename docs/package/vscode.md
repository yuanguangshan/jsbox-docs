# VSCode

JSBox 从一开始就提供了 VSCode 插件：https://marketplace.visualstudio.com/items?itemName=Ying.jsbox

你可以用这个插件增强你写 JSBox 脚本的体验，他可以帮你将脚本从桌面端实时同步到手机端（在同一个 Wi-Fi 下面），也提供了简单的自动补全功能。

从 0.0.6 版本开始，JSBox 的 VSCode 插件也支持了安装包格式，只要当前编辑的文件夹符合 JSBox 安装包的定义：

- **assets/**
- **scripts/**
- **strings/**
- **config.json**
- **main.js**

同步的时候则会将整个目录（安装包）同步到手机，然后运行。

---

## 文件内容解读与示例

### 用途说明

本文档介绍了 **JSBox 的 Visual Studio Code 扩展**。这个扩展旨在极大地提升开发者在桌面端编写 JSBox 脚本的体验。它通过提供**实时同步**、**自动补全**等功能，将桌面 IDE 的强大功能与移动端 JSBox 的便捷运行环境无缝结合，使得 JSBox 脚本的开发变得更加高效和愉悦。

### 核心概念：桌面端与移动端的无缝桥接

在移动设备上直接编写复杂脚本效率较低。JSBox VS Code 扩展解决了这一痛点，它允许开发者在功能更强大、工具更丰富的桌面端（如 Mac 或 Windows）编写代码，然后通过网络将代码实时同步到 iOS 设备上的 JSBox 应用中进行测试和运行。

### 主要功能

#### 1. 实时同步 (Real-time Sync)

-   **机制**: 这是插件的核心功能。当你在 VS Code 中保存脚本文件时，插件会自动检测更改，并通过 Wi-Fi 网络将更新后的脚本推送到连接到同一网络的 JSBox 应用中。
-   **支持格式**: 
    -   **单个 `.js` 文件**: 最简单的同步方式，直接同步当前编辑的 `.js` 文件。
    -   **JSBox 安装包**: 如果你当前在 VS Code 中打开的文件夹符合 JSBox 安装包的定义（即包含 `assets/`, `scripts/`, `strings/`, `config.json`, `main.js` 等目录和文件），插件会将其识别为一个完整的项目，并在同步时将**整个项目目录**打包并同步到手机，然后运行 `main.js`。
-   **优势**: 极大地加速了开发迭代周期，实现了“所见即所得”的开发体验。

#### 2. 自动补全 (Autocompletion)

-   **机制**: 插件提供了 JSBox API 的基本自动补全功能。当你输入 `$ui.`、`$http.` 等 JSBox 内置对象时，VS Code 会弹出可用的属性和方法提示。
-   **优势**: 减少拼写错误，提高编码效率，降低查阅文档的频率。

### 安装与使用

1.  **安装插件**: 在 VS Code 中，打开扩展视图（`Ctrl+Shift+X` 或 `Cmd+Shift+X`），搜索“JSBox”并安装。
2.  **建立连接**: 确保你的 Mac/PC 和 iOS 设备连接在**同一个 Wi-Fi 网络**下。在 JSBox 应用中，通常在“设置”或“关于”页面中找到“VS Code 连接”或类似的选项，点击以建立连接。
3.  **开始同步**: 在 VS Code 中打开你的 JSBox 脚本文件或整个脚本项目文件夹。当你保存文件时，插件会自动触发同步。

### 示例：使用 VS Code 实时同步开发

假设你正在开发一个名为 `MyWidget.box` 的桌面小组件项目，其结构如下：

```
MyWidget.box/
├── assets/
│   └── icon.png
├── scripts/
│   └── widget_logic.js
├── config.json
└── main.js
```

**开发流程**：

1.  在 VS Code 中打开 `MyWidget.box` 文件夹。
2.  修改 `scripts/widget_logic.js` 中的代码，例如调整小组件的显示逻辑。
3.  按下 `Ctrl+S` (或 `Cmd+S`) 保存文件。
4.  在你的 iPhone 上，JSBox 应用会自动检测到文件更新，并重新加载 `MyWidget.box` 项目，然后运行 `main.js`，你将立即看到小组件在模拟器或实际桌面上的变化。

**代码示例 (仅为说明同步机制)**：

```javascript
// main.js
const widgetLogic = require('./scripts/widget_logic');
widgetLogic.renderWidget();

// scripts/widget_logic.js
// module.exports = {
//   renderWidget: () => {
//     $widget.setTimeline(ctx => {
//       return {
//         type: "text",
//         props: {
//           text: "Hello from VS Code! " + new Date().toLocaleTimeString()
//         }
//       };
//     });
//   }
// };
```

**代码解读**：

这个示例展示了，当你修改 `widget_logic.js` 并保存时，整个 `MyWidget.box` 项目会被同步到手机，`main.js` 会被执行，进而调用 `widgetLogic.renderWidget()`，最终更新小组件的显示。这个过程是自动且实时的。

### 总结

JSBox VS Code 扩展是 JSBox 开发者不可或缺的工具。它通过将桌面 IDE 的强大功能与移动端 JSBox 的运行环境无缝结合，极大地提高了开发效率和体验。 
