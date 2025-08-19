> render 和 push 是 JSBox 里面绘制界面最主要的方法

# $ui.render(object)

通过 `$ui.render` 我们可以在画布上绘制界面，他的基本用法是：

```js
$ui.render({
  props: {
    id: "label",
    title: "Hello, World!"
  },
  views: [
    {
      type: "label",
      layout: function(make, view) {
        make.center.equalTo(view.super)
        make.size.equalTo($size(100, 100))
      }
    }
  ]
})
```

因为本质上画布也是一个 view，所以一样支持 `props`，里面的 title 表示在界面中的标题。

views 表示画布中的所有子 view，其中每个 view 也可以嵌套的包含 `views`，是一个递归的结构。

画布支持的额外 props：

属性 | 类型 | 说明
---|---|---
theme | string | 外观：light/dark/auto
title | string | 标题
titleColor | $color | 标题颜色
barColor | $color | bar 背景色
iconColor | $color | 图标颜色
debugging | bool | 显示界面调试工具
navBarHidden | bool | 隐藏导航栏
statusBarHidden | bool | 隐藏状态栏
statusBarStyle | number | 0 为黑色，1 为白色
fullScreen | bool | 是否显示成全屏模式
formSheet | bool | 是否显示成 form sheet（仅 iPad）
pageSheet | bool | 是否显示成 page sheet（iOS 13）
bottomSheet | bool | 是否显示成 bottom sheet（iOS 15）
modalInPresentation | bool | 是否阻止关闭手势
homeIndicatorHidden | bool | 是否为 iPhone X 系列隐藏 Home Indicator
clipsToSafeArea | bool | 是否以 Safe Area 为边界裁剪视图
keyCommands | array | 快捷键

# $ui.push(object)

基本上与 `$ui.render` 完全一致，只不过 render 直接绘制在画布上，而 `push` 则会滑进来一个新的画布（可以滑动返回），让整个界面变成一个视图栈，可以多次 push 用于表达查看详情等场景。

从 v1.36.0 版本开始，可以通过 $ui.push("detail.ux") 来推进一个通过可视化界面编辑器生成的页面。

# $(id)

该方法用于通过 `id` 在画布中查找一个 view，例如：

```js
const label = $("label");
```

当 id 没有被指定的时候，会通过 type 进行查找，如果画布中只有一个该类型的 view，也会被正确返回。

# 生命周期

目前对于 $ui.render 和 $ui.push，支持以下生命周期回调：

```js
events: {
  appeared: function() {

  },
  disappeared: function() {

  },
  dealloc: function() {

  }
}
```

顾名思义，分别在页面出现、消失和销毁时被调用。

# 键盘高度变化

你可以这样检测键盘高度变化：

```js
events: {
  keyboardHeightChanged: height => {

  }
}
```

# 摇一摇手势

你可以这样检测摇一摇手势：

```js
events: {
  shakeDetected: function() {

  }
}
```

# 支持外接键盘

可以通过 `keyCommands` 设置外接键盘快捷键：

```js
$ui.render({
  props: {
    keyCommands: [
      {
        input: "I",
        modifiers: 1 << 20,
        title: "Discoverability Title",
        handler: () => {
          console.log("Command+I triggered.");
        }
      }
    ]
  }
});
```

modifiers 是一个掩码，有如下取值：

属性 | 取值
---|---
Caps lock | 1 << 16
Shift | 1 << 17
Control | 1 << 18
Alternate | 1 << 19
Command | 1 << 20
NumericPad | 1 << 21

例如需要同时按住 `Command` 和 `Shift` 的话，则 modifiers 为 (1 << 20 | 1 << 17)。

---

## 文件内容解读与示例

### 用途说明

本文档介绍了 JSBox UI 框架中**最核心的两个入口函数**：`$ui.render` 和 `$ui.push`。它们是启动一个用户界面的起点。同时，文档还涵盖了页面级的生命周期事件、如何查找视图、以及如何响应键盘等系统级事件。

### 核心概念：页面 (Page) 与导航栈 (Navigation Stack)

-   **`$ui.render(object)`**: 
    -   **作用**: 渲染一个**主页面**或**根页面**。它会替换掉当前窗口的全部内容，创建一个全新的视图层级。
    -   **用途**: 通常在脚本启动时调用，用于构建应用的主界面。

-   **`$ui.push(object)`**: 
    -   **作用**: 在现有页面的基础上，**推入 (push)** 一个新的页面。新页面会从右侧滑入，并自动在导航栏上显示一个“返回”按钮。
    -   **导航栈**: 多次调用 `$ui.push` 会形成一个“页面栈”（即 UINavigationController）。你可以通过 `$ui.pop()` 返回上一页，或 `$ui.popToRoot()` 直接返回到最开始的页面。
    -   **用途**: 用于实现“主-从”式的界面流程，例如从一个列表页点击进入一个详情页。

### 页面 (`props`)

`$ui.render` 和 `$ui.push` 的参数是一个描述页面的对象。页面本身也是一个特殊的视图，因此它也有 `props` 和 `views`。`props` 中包含了一些对整个页面生效的特殊属性：

-   **导航栏**: `title`, `titleColor`, `barColor`, `iconColor`, `navBarHidden` 等，用于定制页面顶部的导航栏样式。
-   **状态栏**: `statusBarHidden`, `statusBarStyle` 用于控制设备顶部的状态栏。
-   **显示模式**: `fullScreen`, `pageSheet`, `formSheet` 等，用于控制页面的弹出和显示方式，特别是在 iPad 上。
-   **交互控制**: `modalInPresentation` 可以防止用户通过下滑手势关闭页面；`homeIndicatorHidden` 可以隐藏 iPhone X 及后续机型的 Home 横条。
-   **外接键盘**: `keyCommands` 允许你为页面定义键盘快捷键，极大地提升在 iPad 等设备上的使用体验。

### 视图查找：`$(id)`

-   **作用**: 这是一个全局的、非常便捷的视图选择器，类似于 Web 开发中的 `document.getElementById`。
-   **用法**: 
    1.  在定义视图时，为其 `props` 添加一个唯一的 `id`，如 `props: { id: "myLabel" }`。
    2.  在代码的任何地方，通过 `$("myLabel")` 即可获取到该视图的实例，然后可以读取或修改它的属性（如 `$("myLabel").text = "New Text"`）。
    -   **快捷方式**: 如果页面上某种类型的视图只有一个（例如只有一个 `list`），你可以直接用类型名查找，如 `$("list")`。

### 页面生命周期 (`events`)

在页面的 `events` 块中，可以定义几个关键的生命周期回调函数：

-   **`appeared`**: 页面完全显示在屏幕上之后调用。
-   **`disappeared`**: 页面从屏幕上完全消失之后调用。
-   **`dealloc`**: 页面被销毁、内存被回收时调用。可以在这里执行最终的清理工作。

### 系统事件

页面还可以监听一些系统级的事件：

-   **`keyboardHeightChanged: (height) => { ... }`**: 当键盘弹出或收起时触发，返回键盘的实时高度。这对于需要避免键盘遮挡输入框的场景非常有用。
-   **`shakeDetected: () => { ... }`**: 当用户摇晃设备时触发。

### 示例：一个简单的两级页面导航

```javascript
// --- 页面 1: 列表页 ---
$ui.render({
  props: {
    title: "联系人列表"
  },
  views: [
    {
      type: "list",
      props: {
        data: ["Alice", "Bob", "Charlie"]
      },
      layout: $layout.fill,
      events: {
        didSelect: (sender, indexPath, data) => {
          // 点击列表项时，推入详情页
          pushDetailPage(data);
        }
      }
    }
  ],
  events: {
    appeared: () => console.log("列表页已出现"),
    disappeared: () => console.log("列表页已消失")
  }
});

// --- 页面 2: 详情页 (由一个函数创建) ---
function pushDetailPage(name) {
  $ui.push({
    props: {
      title: name // 将名字设为标题
    },
    views: [
      {
        type: "label",
        props: {
          text: `这是 ${name} 的详情页。`,
          align: $align.center
        },
        layout: (make, view) => {
          make.center.equalTo(view.super);
        }
      }
    ],
    events: {
      appeared: () => console.log(`${name} 的详情页已出现`),
      // 页面销毁时打印日志
      dealloc: () => console.log(`${name} 的详情页已销毁`)
    }
  });
}
```

### 总结

`$ui.render` 和 `$ui.push` 是构建多页面应用的骨架。通过它们，你可以初始化 UI、组织页面间的导航关系。结合页面级的 `props` 和 `events`，可以对页面的外观、行为和生命周期进行精细的控制。而 `$(id)` 选择器则是连接静态 UI 定义和动态逻辑交互的关键桥梁。