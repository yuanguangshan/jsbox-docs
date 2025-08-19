# type: "runtime"

可以使用 Runtime 来初始化一个 view:

```js
const label = $objc("UILabel").$new();
label.$setText("Hey!");

$ui.render({
  views: [
    {
      type: "runtime",
      props: {
        view: label
      },
      layout: function(make, view) {
        make.center.equalTo(view.super);
      }
    }
  ]
});
```

通过这种方式，你可以将 JSBox 提供的组件与 Runtime 生成的组件混合在一起。

---

## 文件内容解读与示例

### 组件用途与重要警告

`runtime` 组件是一个**专家级**的功能，它扮演着一个特殊的“容器”或“桥梁”角色，允许你将在 JavaScript 中通过 **Objective-C Runtime** 创建的**原生 iOS 视图**，无缝地嵌入到 JSBox 的 UI 布局中。

当你需要使用某个 JSBox 没有直接封装的 iOS 原生组件（例如 `UISwitch`, `UIStepper` 等）时，`runtime` 组件就成了你的终极解决方案。

> **⚠️ 重要警告**: 此功能面向对原生 iOS 开发 (Objective-C/Swift) 和 Runtime 机制有相当了解的高级用户。不恰当地使用原生 API 极易导致应用闪退或出现无法预料的错误。在尝试使用前，请确保你清楚地知道你在做什么。

### 核心概念：将原生视图“嫁接”到 JSBox

其工作流程可以分为三步：

1.  **创建原生视图**: 使用 `$objc()` 相关的 Runtime API 来获取一个原生 UI 类的引用（如 `$objc("UISwitch")`），并调用其方法（如 `.$new()`）来创建实例。
2.  **配置原生视图**: 继续使用 Runtime 语法调用该实例的方法，对其进行配置（例如 `.$setOn(true)`）。对于交互，你需要使用原生的 Target-Action 模式来绑定事件。
3.  **嵌入到 JSBox**: 将创建好的原生视图实例，赋值给 `runtime` 组件的 `props: { view: ... }` 属性。之后，你就可以像对待普通 JSBox 组件一样，使用 `$layout` 函数来约束它的位置和大小。

### 示例代码：嵌入一个原生开关 (UISwitch)

JSBox 提供了 `switch` 组件，但为了演示，我们将来创建一个原生的 `UISwitch` 并响应它的值改变事件。

```javascript
// 1. 创建一个 JS 对象，用于接收原生事件
const target = $objc_js("MyTarget");

target.implementation = {
  // 定义一个方法，用于被原生 UISwitch 调用
  switchChanged: sender => {
    const isOn = sender.$isOn();
    $("status-label").text = `原生开关状态: ${isOn ? "ON" : "OFF"}`;
  }
};

// 2. 创建原生 UISwitch 实例
const nativeSwitch = $objc("UISwitch").$new();
// 设置初始状态为 ON
nativeSwitch.$setOn(true);
// 添加 Target-Action，当值改变时，调用 target 对象的 switchChanged: 方法
nativeSwitch.$addTarget_action_forControlEvents(target, "switchChanged:", 1 << 12); // 1 << 12 is UIControlEventValueChanged

// 3. 渲染 UI
$ui.render({
  props: { title: "Runtime 组件示例" },
  views: [
    {
      type: "label",
      props: {
        id: "status-label",
        text: "原生开关状态: ON",
        align: $align.center
      },
      layout: make => {
        make.centerX.equalTo(view.super);
        make.top.inset(50);
      }
    },
    {
      // 4. 使用 runtime 组件嵌入原生 switch
      type: "runtime",
      props: {
        view: nativeSwitch
      },
      layout: (make, view) => {
        make.centerX.equalTo(view.super);
        make.top.equalTo($("status-label").bottom).offset(20);
      }
    }
  ]
});
```

**代码解读**：

1.  **Target-Action**: 这是原生 iOS 的事件处理机制。我们创建了一个名为 `MyTarget` 的 JS 对象，并为其 `implementation` 添加了 `switchChanged` 方法。这是响应原生事件的关键步骤。
2.  **创建与配置**: 我们通过 `$objc("UISwitch").$new()` 创建了开关实例，并通过 `$addTarget...` 方法将它的“值改变”事件（`UIControlEventValueChanged`）与我们 `target` 对象的 `switchChanged` 方法绑定起来。
3.  **嵌入与交互**: `runtime` 组件将 `nativeSwitch` 显示在界面上。当你拨动这个开关时，原生的 `UIControlEventValueChanged` 事件被触发，从而调用到你在 JS 中定义的 `switchChanged` 方法，该方法进而更新了上方的 JSBox `label` 组件的文本。这展示了从原生组件到 JSBox 组件的完整通信链路。

`runtime` 组件是 JSBox 强大扩展能力的极致体现，它为你打开了通往原生世界的大门，但也要求你具备相应的原生开发知识。 
