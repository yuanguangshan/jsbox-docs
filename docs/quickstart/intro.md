# JavaScript

`JavaScript` 是一门强大而灵活的编程语言，本文假设读者具有基本的 JavaScript 基础。

> 相关资源
> - [w3schools.com](https://www.w3schools.com/js/)
> - [mozilla](https://developer.mozilla.org/en-US/docs/Web/JavaScript)

# 相似的项目

JSBox 从理念上来说并没有太多的开创性，有很多 `可以运行脚本` 的产品，他们的形态和理念深深地影响着 JSBox 的开发，比如 `Editorial` 和 `Automator`，他们都是可以通过脚本来设计小程序的产品。

但 JSBox 开发过程中并没有模仿他们设计和实现，并且采用了完全不同的道路，就是基于 `JavaScript`，JSBox 与他们唯一的相似之处是能够运行脚本。

另外，市面上有大量的通过 JavaScript 描述 `Native UI` 的框架，例如 Facebook 的 `React Native` 和阿里的 `Weex`，JSBox 在设计和实现的过程中考虑过直接采用他们的方案，但最终考虑到灵活度的问题没有这样做，主要原因有三点：

- React Native 和 Weex 都是为了跨平台需求而设计的框架
- JSBox 是完全为了 iOS 而生，能更好的描述 iOS 的 Native 控件
- JSBox 有一个更重要的目标是：`尽可能简单和易学`，不会引入 JavaScript 之外的概念

# API 设计原则

- API 都以 `$` 符号开始，例如 `$clipboard`
- 与 `Cocoa` 的哲学相反，JSBox 的 API 会尽可能的短，因为在移动平台上面编写、修改代码不是一件容易的事情
- 绝大部分的时候，参数会是一个 `JavaScript Object`，除非只有一个参数
- 每个扩展有自己的运行环境，这个环境产生的数据是与其他的扩展隔绝的，也会随着扩展的删除而消失

以上是对于 JSBox 的基本介绍，*已经对 JavaScript 有所了解，想要看看[简单样例 >](quickstart/sample.md)。*

---

## 文件内容解读与示例

### 用途说明

本文档是 JSBox **快速入门指南的引言**，旨在为新用户提供一个关于 JSBox 的宏观认识。它阐述了 JSBox 的核心理念、设计哲学以及 API 设计原则，帮助开发者在开始编写脚本之前，对 JSBox 的定位和工作方式有一个清晰的理解。

### 核心理念：JavaScript 驱动的 iOS 原生体验

-   **JavaScript 作为基石**: JSBox 以 JavaScript 作为唯一的编程语言，极大地降低了 iOS 自动化和应用开发的门槛。它假设读者具备基本的 JavaScript 基础，并推荐了相关的学习资源。
-   **与相似项目的异同**: 
    -   **相似之处**: JSBox 与 `Editorial`、`Automator` 等产品一样，都允许用户通过脚本来设计和运行小程序。
    -   **不同之处**: JSBox 并非模仿 `React Native` 或 `Weex`。它有明确的设计目标：
        1.  **专注于 iOS**: JSBox 是完全为 iOS 而生，能够更好地描述和利用 iOS 的原生控件和系统能力，不追求跨平台。
        2.  **简单易学**: JSBox 的一个重要目标是“尽可能简单和易学”，它避免引入 JavaScript 之外的复杂概念，让开发者能够快速上手。

### API 设计原则详解

JSBox 的 API 设计遵循一套独特的原则，旨在适应移动平台上的开发特点：

1.  **API 都以 `$` 符号开始**: 这是 JSBox API 的标志性特征。例如 `$clipboard`、`$ui`、`$http`。这有助于区分 JSBox 特有的功能与标准的 JavaScript 内建函数。
2.  **API 尽可能短**: 考虑到在移动设备上编写和修改代码可能不便，JSBox 的 API 名称设计得非常简洁，以减少输入量。
3.  **参数通常是 `JavaScript Object`**: 绝大部分 API 接受一个 JavaScript 对象作为参数。这种方式使得参数传递更灵活、可读性更高，并且易于扩展。
4.  **沙盒隔离**: 每个 JSBox 脚本都有自己的独立运行环境（“沙盒”）。脚本之间的数据是相互隔离的，并且会随着脚本的删除而消失。这保证了安全性和稳定性，避免了脚本间的相互干扰。

### 示例代码：体现 API 设计原则

下面的简单脚本演示了 JSBox API 设计原则的实际应用：

```javascript
// 1. API 都以 $ 符号开始：$ui, $font, $color, $layout
$ui.render({
  props: {
    title: "JSBox 快速入门"
  },
  views: [
    {
      type: "label",
      // 2. 参数是 JavaScript Object：font, textColor, layout
      props: {
        text: "欢迎来到 JSBox 的世界！",
        font: $font("bold", 24), // $font 函数创建字体对象
        textColor: $color("tintColor") // $color 函数创建颜色对象
      },
      layout: $layout.center // $layout 提供了预定义的布局函数
    }
  ]
});

// 3. 沙盒隔离：这个脚本的 $prefs 不会影响其他脚本
// 假设我们在这里保存一个设置项
$prefs.set("first_run_completed", true);
console.log("设置项 'first_run_completed' 已保存到当前脚本的沙盒中。");
```

**代码解读**：

-   脚本中使用的 `$ui`、`$font`、`$color`、`$layout`、`$prefs` 都以 `$` 开头，符合 JSBox 的命名约定。
-   `font`、`textColor`、`layout` 等属性都接受 JavaScript 对象作为参数，体现了参数传递的灵活性。
-   `$prefs.set()` 操作的数据是存储在当前脚本的独立沙盒中，不会与其他 JSBox 脚本混淆，体现了沙盒隔离的原则。

### 总结

JSBox 的设计哲学是“用 JavaScript 赋能 iOS”，其 API 设计旨在提供简洁、高效且符合移动平台特点的开发体验。理解这些基本介绍和设计原则，将为你后续深入学习 JSBox 的具体 API 模块打下坚实的基础。 
