> 在 JSBox 的视图系统里面，还提供了很多接口用于与用户进行交互

# $ui.pop()

可以将 push 进来的视图弹出去一层。

# $ui.popToRoot()

直接回到视图栈的第一层。

# $ui.get(id)

获得一个 view 实例，等同于 `$(id)`。

> 如果页面里面只有一个该类型的 view 则可以使用 type 获得，例如 $("list")，否则需要指定 id。

# $ui.alert(object)

给用户一个 alert 框，用于提示或者选择：

```js
$ui.alert({
  title: "Hello",
  message: "World",
  actions: [
    {
      title: "OK",
      disabled: false, // Optional
      style: $alertActionType.default, // Optional
      handler: function() {

      }
    },
    {
      title: "Cancel",
      style: $alertActionType.destructive, // Optional
      handler: function() {

      }
    }
  ]
})
```

创建一个有标题、信息和按钮的 alert，如果不提供 `actions` 的话，一个默认的确定按钮会被自动提供。

`title` 和 `message` 都是可选项，同时如果偷懒的话可以只：

```js
$ui.alert("Haha!")
```

通常可以这样来测试得到的数据，因为目前还没有提供 debug 用的组件。

# $ui.action(object)

和 `$ui.alert` 基本一样，不同的是将会采用 `action sheet` 风格提供选项，目前 iPad 上不可使用。

# $ui.menu(object)

用一种更偷懒的方式创建一个菜单：

```js
$ui.menu({
  items: ["A", "B", "C"],
  handler: function(title, idx) {

  },
  finished: function(cancelled) {
    
  }
})
```

# $ui.popover(object)

使用 Popover 的样式弹出一个浮窗，该接口提供两种使用方式。

方式一，构建一个简单的列表选择浮窗：

```js
const {index, title} = await $ui.popover({
  sourceView: sender,
  sourceRect: sender.bounds, // default
  directions: $popoverDirection.up, // default
  size: $size(320, 200), // fits content by default
  items: ["Option A", "Option B"],
  dismissed: () => {},
});
```

此方式通过 `items` 指定每个选项的标题，返回一个 Promise。

方式二，使用自定义的 `views` 构建一个浮窗：

```js
const popover = $ui.popover({
  sourceView: sender,
  sourceRect: sender.bounds, // default
  directions: $popoverDirection.any, // default
  size: $size(320, 200), // fits screen width by default
  views: [
    {
      type: "button",
      props: {
        title: "Button"
      },
      layout: (make, view) => {
        make.center.equalTo(view.super);
        make.size.equalTo($size(100, 36));
      },
      events: {
        tapped: () => {
          popover.dismiss();
        }
      }
    }
  ]
});
```

此方式通过 `views` 来绘制自定义的界面，返回 popover 本身，可以调用 `dismiss` 方法将其关闭。

其中 `sourceView` 为弹出 popover 所必需的来源，它通常是一个 button，或是 navButtons 回调中的 sender。`sourceRect` 则为 popover 箭头所指向的位置（默认为 sourceView.bounds），`directions` 表示箭头允许的方向。

请参考我们提供的 demo 项目了解更多：https://gist.github.com/cyanzhong/313b2c8d138691233658f1b8a52f02c6

# $ui.toast(message)

显示一个悬浮的提示信息，几秒后自动消失：

```js
$ui.toast("Hey!")
```

`toast` 也支持设置显示时间：

```js
$ui.toast("Hey!", 10)
```

此 Toast 将会显示持续 10 秒钟。

> 你可以通过 $ui.clearToast() 来清除已经显示的 toast。

# $ui.success(string)

与 `toast` 类似，但背景色为绿色，以表示成功：

```js
$ui.success("Done");
```

# $ui.warning(string)

与 `toast` 类似，但背景色为黄色，以表示警告：

```js
$ui.warning("Be careful!");
```

# $ui.error(string)

与 `toast` 类似，但背景色为红色，以表示错误：

```js
$ui.error("Something went wrong!");
```

# $ui.loading(boolean)

在主界面显示一个正在加载的提示框：

```js
$ui.loading(true)
```

PS: 由于历史原因这个接口设计有点问题，除了使用 `true` 和 `false` 来开始或停止一个提示框以外，也可以使用是一个字符串来进行提示：

```js
$ui.loading("加载中...")
```

# $ui.progress(number)

显示加载进度，number 不处于 [0, 1] 之间时自动消失：

```js
$ui.progress(0.5)
```

同样，也支持通过一个（可选的）参数来设置提示语：

```js
$ui.progress(0.5, "下载中...")
```

# $ui.preview(object)

为文件/数据预览提供一个更便捷的方法：

```js
$ui.preview({
  title: "URL",
  url: "https://images.apple.com/v/ios/what-is/b/images/performance_large.jpg"
})
```

支持的属性：

属性 | 类型 | 说明
---|---|---
title | string | 标题
url | string | 链接
html | string | html
text | string | 文本

# $ui.create(object)

手动创建一个 view，object 参数和 $ui.render 一致：

```js
const canvas = $ui.create({
  type: "image",
  props: {
    bgcolor: $color("clear"),
    tintColor: $color("gray"),
    frame: $rect(0, 0, 100, 100)
  }
});
```

请注意，这个时候 view 还没有父 view，所以在这个时候不能使用他的 `layout` 方法。

取而代之的是，应该在把它添加到父 view 之后，手动进行 layout:

```js
const subview = $ui.create(...);
superview.add(subview);
subview.layout((make, view) => {

});
```

# $ui.window

获得 $ui.push 页面的 window 对象。

# $ui.controller

获得应用内最顶部的 view controller 对象。

# $ui.title

获取或设置应用内最顶部视图的标题。

# $ui.selectIcon()

从图标库中选择图标：

```js
var icon = await $ui.selectIcon();
```

---

## 文件内容解读与示例

### 用途说明

本文档是 **JSBox 全局 UI 方法**的集合，这些方法提供了与用户进行交互的各种标准化、便捷的接口。它们不依附于任何特定视图，可以从代码的任何地方调用，用于处理导航、显示模态对话框（如提醒、菜单）、提供反馈（如 Toast、加载指示器）以及动态创建视图等。

### API 详解 (按功能分类)

#### 1. 导航 (Navigation)

-   **`$ui.pop()`**: 在一个由 `$ui.push` 构建的导航栈中，返回上一级页面。
-   **`$ui.popToRoot()`**: 直接返回到导航栈的第一个页面（根页面）。

#### 2. 模态对话框 (Modal Dialogs)

-   **`$ui.alert({title, message, actions})`**: 显示一个标准的**警告框**。用于向用户显示重要信息或提供需要用户做出选择的操作（如“确定”、“取消”）。可以快速调用 `$ui.alert("message")` 来进行简单的信息提示。
-   **`$ui.action({title, message, actions})`**: 显示一个**操作表 (Action Sheet)**，从屏幕底部滑出选项列表。通常用于提供与当前上下文相关的一系列操作。在 iPad 上表现为 Popover。
-   **`$ui.menu({items, handler, finished})`**: 一个更快捷的创建简单列表菜单的方式。它会显示一个列表，用户选择后，`handler` 会被调用并返回选择的标题和索引。
-   **`$ui.popover({sourceView, views, items, ...})`**: 显示一个**浮窗 (Popover)**，通常用于 iPad 或作为按钮的附加菜单。它功能强大，支持两种模式：
    1.  **列表模式**: 提供 `items` 数组，快速创建一个选择列表。
    2.  **自定义视图模式**: 提供 `views` 数组，可以在浮窗内绘制任意自定义的 UI。

#### 3. 用户反馈 (User Feedback)

-   **`$ui.toast(message, duration)`**: 在屏幕下方显示一个短暂的、非阻塞的**轻提示**，几秒后自动消失。用于告知用户某个操作已完成。
-   **`$ui.success(message)` / `$ui.warning(message)` / `$ui.error(message)`**: 带有状态颜色的 `toast`，分别显示绿色（成功）、黄色（警告）、红色（错误），能更直观地传达操作结果。
-   **`$ui.loading(true/false)` 或 `$ui.loading("message")`**: 显示一个覆盖全屏的**加载指示器**，阻止用户操作。用于执行网络请求等耗时任务时，明确告知用户程序正在处理中。
-   **`$ui.progress(value, message)`**: 显示一个带进度的加载指示器。`value` 是一个 0 到 1 之间的小数。当 `value` 超出这个范围时，指示器自动消失。非常适合用于显示文件下载、上传等任务的进度。

#### 4. 内容预览与视图操作

-   **`$ui.preview({url, html, text})`**: 使用系统原生的 `QLPreviewController` 快速预览内容，支持预览 URL、HTML 字符串或纯文本。
-   **`$ui.create({type, props, ...})`**: **动态创建**一个视图实例，但不立即显示它。返回的视图对象可以被存储在变量中，之后再通过 `view.add(object)` 方法添加到某个父视图中。这是实现动态添加 UI 元素的核心方法。
-   **`$ui.get(id)` 或 `$(id)`**: 通过 `id` 在当前视图层级中查找并返回一个视图实例，是获取和操作已存在视图最常用的方法。

#### 5. 高级接口

-   **`$ui.window` / `$ui.controller`**: 获取顶层的 `UIWindow` 和 `UIViewController` 对象，用于进行更底层的原生 UI 操作。
-   **`$ui.title`**: 快捷地获取或设置当前页面的导航栏标题。
-   **`$ui.selectIcon()`**: 弹出一个内置的图标库，让用户选择一个图标，返回图标名称。常用于应用的自定义功能中。

### 示例：一个完整的交互流程

```javascript
async function main() {
  // 1. 询问用户操作
  const {title} = await $ui.menu({ items: ["获取天气", "显示图片"] });

  if (title === "获取天气") {
    // 2. 显示加载指示器
    $ui.loading("正在获取天气...");
    const resp = await $http.get("https://api.example.com/weather");
    $ui.loading(false);

    // 3. 显示结果
    if (resp.data) {
      $ui.alert({ title: "天气预报", message: resp.data.forecast });
    } else {
      $ui.error("获取失败");
    }
  } else if (title === "显示图片") {
    // 4. 使用 Popover 提供选项
    const { index } = await $ui.popover({
      sourceView: $("someButton"), // 假设有一个按钮触发
      items: ["风景", "动物"]
    });

    const imageUrl = index === 0 ? "url_for_scenery" : "url_for_animal";
    
    // 5. 预览图片
    $ui.preview({ title: "图片预览", url: imageUrl });
  }
}
```

### 总结

`$ui` 上的全局方法是 JSBox UI 框架的“工具箱”，提供了构建交互式应用所需的一切标准组件。熟练运用这些方法，可以轻松实现页面导航、用户输入、状态反馈和动态 UI 更新，是提升脚本用户体验的关键。