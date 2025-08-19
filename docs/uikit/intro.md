# 基本概念

JSBox 对 [UIKit](https://developer.apple.com/documentation/uikit) 进行了简单的封装和抽象，让用户可以通过简单的 `JavaScript` 在 JSBox 里面绘制界面，从而提供更好的交互。

JSBox 中界面绘制采用的方案和 [React Native](https://facebook.github.io/react-native/) 以及 [Weex](https://weex.incubator.apache.org/) 不同，不需要你掌握 `HTML` 和 `CSS`，只需要定义 JavaScript Object 就能完成所有工作。

PS: 当然，你写的扩展可以没有任何界面，不提供任何用户交互也是完全可以的。

# 样例

正如前文样例提到的那样，我们可以通过下面的代码在屏幕上创建一个按钮：

```js
$ui.render({
  views: [
    {
      type: "button",
      props: {
        title: "Button"
      },
      layout: function(make, view) {
        make.center.equalTo(view.super)
        make.width.equalTo(64)
      },
      events: {
        tapped: function(sender) {
          $ui.toast("Tapped")
        }
      }
    }
  ]
})
```

这段代码囊括了 view 的四个核心：`type`, `props`, `layout` 和 `events`。

# type

type 指定了这个 view 的类型，例如上述的 button，在之后的文档里面我们会详细介绍每种 view 如何使用。

# props

props 里面可以对 view 进行各种属性设置，例如 button 支持的 `title` 属性，不同类型的 view 支持的属性不尽相同，在之后会分别介绍。

# layout

对 view 进行布局，JSBox 沿用了 iOS 里面的 [Auto Layout](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/AutolayoutPG/index.html) 语法，但你无需对原生的 Auto Layout 有深入的了解（那很难用），因为我们提供的是基于 Masonry 的 DSL：https://github.com/SnapKit/Masonry 在之后我们会示例如何简单的使用 Masonry。

PS: 使用这套布局引擎实际上是一个权衡的方案，之后时间充裕的话可能会提供其他的布局引擎，例如 [Flexbox](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout/Using_CSS_flexible_boxes)。

# events

在这里响应这个 view 的各种事件，例如 button 支持的 `tapped` 事件，在事件回调方法里面进行各种处理，在之后的文档里面会详细介绍每种 view 支持的 events。

# views

在某个 view 里面定义 `views` 用于描述他的子 view:

```js
{
  views: [

  ]
}
```

子 view 的结构定义和 view 完全一样，递归定义构成一个视图树。

# 类型转换

Native 代码里面很多数据类型是 JavaScript 不具备的，所以 JSBox 提供了一系列的转换方法，这在绘制界面的时候尤其重要：[数据转换](data/intro.md)

---

## 文件内容解读与示例

### 用途说明

本文档是 **JSBox UI 框架的基石**，介绍了构建用户界面的最核心、最基本的概念。它解释了 JSBox 如何将简单的 JavaScript 对象映射为原生的 iOS 界面元素（基于 UIKit），以及构成一个可视化视图（View）的四个基本要素。

### 核心理念：声明式 UI

JSBox 采用了一种**声明式**的方法来构建 UI。与传统的命令式编程（如手动创建对象、设置属性、添加到父视图）不同，你只需要**描述你想要的 UI 是什么样子的**，而不需要关心具体的创建过程。你通过一个层层嵌套的 JavaScript 对象来描述整个视图树，JSBox 的渲染引擎会负责将其转换为真实的、用户可见的界面。

这种方法类似于 React，但更简化，无需学习 JSX、HTML 或 CSS。

### 构成视图的四大核心

文档中的样例代码精炼地展示了定义一个视图所需的四个核心部分，理解这四部分是掌握 JSBox UI 编程的关键：

1.  **`type` (类型)**
    -   **作用**: 定义这个视图是什么。它是一个字符串，指定了视图的种类，例如 `"button"`, `"label"`, `"image"`, `"list"` 等。
    -   **重要性**: 这是最基本的属性，决定了视图的外观和基本行为。

2.  **`props` (属性)**
    -   **作用**: 定义这个视图长什么样。它是一个对象，包含了所有用于配置视图外观和内在数据的属性。
    -   **示例**: 对于 `button`，`props` 可以有 `title`；对于 `label`，可以有 `text`, `font`, `textColor`；对于 `view`，可以有 `bgcolor` (背景色)。每种 `type` 的视图都有其支持的一套 `props`。

3.  **`layout` (布局)**
    -   **作用**: 定义这个视图放在哪里以及多大。它是一个函数，用于描述视图的位置和尺寸约束。
    -   **核心**: JSBox 使用了 iOS 中强大且灵活的 **Auto Layout** 系统，并通过一个更简洁的 DSL (领域特定语言，基于 Masonry) 来定义约束。你描述的是视图与其他视图（父视图、兄弟视图）的**相对关系**（如“在父视图中居中”、“宽度与另一个视图相等”），而不是写死的坐标。
    -   **函数签名**: `function(make, view) { ... }`
        -   `make`: 约束构建器，用于创建约束链（如 `make.center.equalTo(...)`）。
        -   `view`: 正在布局的视图实例本身，可以用来引用其属性（如 `view.super` 指向父视图）。

4.  **`events` (事件)**
    -   **作用**: 定义这个视图如何响应用户操作。它是一个对象，包含了各种事件处理器函数。
    -   **示例**: 对于 `button`，最常用的事件是 `tapped` (点击)；对于 `slider`，是 `changed` (值改变)。当用户执行相应操作时，对应的函数就会被调用。

### 视图的嵌套 (`views`)

-   **作用**: 一个复杂的界面是由简单的视图组合而成的。通过在任意一个视图定义中添加一个 `views` 数组，你就可以为其添加子视图。
-   **递归结构**: `views` 数组中的每一个对象都是一个完整的、遵循上述四要素的视图定义。这种递归嵌套的结构形成了一个**视图树**，清晰地描述了整个界面的层级关系。

### 示例：一个简单的登录表单

```javascript
$ui.render({
  props: { // 根视图的 props
    title: "登录"
  },
  views: [ // 根视图的子视图数组
    {
      type: "label", // 类型：标签
      props: { text: "用户名:" }, // 属性：文本内容
      layout: (make) => { // 布局
        make.top.equalTo(20);
        make.left.equalTo(20);
      }
    },
    {
      type: "input", // 类型：输入框
      props: { id: "username", placeholder: "请输入用户名" }, // 属性：ID和占位符
      layout: (make) => {
        make.top.equalTo($("label").bottom).offset(10);
        make.left.right.inset(20);
        make.height.equalTo(40);
      }
    },
    {
      type: "button", // 类型：按钮
      props: { title: "登录" }, // 属性：标题
      layout: (make) => { // 布局
        make.top.equalTo($("input").bottom).offset(20);
        make.centerX.equalTo(view.super);
        make.width.equalTo(100);
      },
      events: { // 事件
        tapped: (sender) => {
          const username = $("username").text;
          $ui.alert(`你好, ${username}!`);
        }
      }
    }
  ]
});
```

### 总结

本文档是学习 JSBox UI 开发的起点。深刻理解 `type`, `props`, `layout`, `events` 这四个核心概念，以及如何通过 `views` 数组嵌套它们来构建视图层级，是创建任何复杂界面的基础。后续的所有 UI 相关文档都是在这一核心框架下的扩展和细化。