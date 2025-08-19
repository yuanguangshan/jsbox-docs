# 基本概念

[Objective-C Runtime](https://developer.apple.com/documentation/objectivec/objective_c_runtime) 是一把利刃，类似于其他语言中的反射机制，却有着更灵活的运行时特性。

利用 Runtime 我们在 Objective-C 里面可以做到：

- 动态调用方法
- 动态创建类
- 动态创建实例
- 替换已有的方法

总之 Runtime 可以做到太多太多的事情，是 Objective-C 语言里面的黑魔法之一。

# 为什么要引入 Runtime

最根本的原因是为有缺陷的 API 提供一个备用方案，正如 Runtime 绝大多时候的应用场景一样，它是一把屠龙的刀，屠龙之技是不会用在日常生活中的。

假设你有一个功能要实现，JSBox 提供的 API 没有满足你，或者是满足的有缺陷，你可以考虑通过 JSBox 提供的 Runtime 接口直接调用 Objective-C 的 API。

当然，在你可以实现功能的时候，应该尽量避免使用 Runtime 接口，因为调用起来比较复杂，查错也相对困难。

# JSBox 提供的接口

- `$objc(className)` 用于动态获取一个 Objective-C 的类
- `invoke(methodName, arguments ...)` 用于动态调用一个 Objective-C 的方法
- `$define(object)` 用于动态创建一个 Objective-C 的类
- `jsValue()` 用于将 Objective-C 数据转换成 JavaScript 值
- `ocValue()` 用于将 JavaScript 值封装成 Objective-C 值
- `$objc_retain(object)` 维持 object
- `$objc_release(object)` 释放 object

# 一个例子

这个例子在 JSBox 的窗口中创建一个按钮和标签，点击按钮后会打开微信：

```js
$define({
  type: "MyHelper",
  classEvents: {
    open: function(scheme) {
      const url = $objc("NSURL").invoke("URLWithString", scheme);
      $objc("UIApplication").invoke("sharedApplication.openURL", url)
    }
  }
})

$ui.render({
  views: [
    {
      type: "button",
      props: {
        bgcolor: $objc("UIColor").invoke("blackColor").jsValue(),
        titleColor: $color("#FFFFFF").ocValue().jsValue(),
        title: "WeChat"
      },
      layout: function(make, view) {
        make.center.equalTo(view.super)
        make.size.equalTo($size(100, 32))
      },
      events: {
        tapped: function(sender) {
          $objc("MyHelper").invoke("open", "weixin://")
        }
      }
    }
  ]
})

const window = $ui.window.ocValue();
const label = $objc("UILabel").invoke("alloc.init");
label.invoke("setTextAlignment", 1)
label.invoke("setText", "Runtime")
label.invoke("setFrame", { x: $device.info.screen.width * 0.5 - 50, y: 240, width: 100, height: 32 })
window.invoke("addSubview", label)
```

这个例子混合了原始的 JSBox 调用和 Runtime 调用两种方式，展示了两种数据类型如何处理。

一个更完善也更复杂的例子，完全使用 Runtime 构建一个 2048 小游戏：https://github.com/cyanzhong/xTeko/tree/master/extension-scripts/2048

---

## 文件内容解读与示例

### 用途说明

本文档介绍了 JSBox 中**Objective-C Runtime** 的概念及其在脚本开发中的应用。Runtime 是 iOS 平台 Objective-C 语言的底层机制，它允许程序在运行时动态地检查、修改和创建类、对象和方法。这使得 Objective-C 具有极高的灵活性和“黑魔法”般的特性。

### 核心概念：Runtime 的“黑魔法”与 JSBox 的桥接

-   **什么是 Runtime**: Runtime 类似于其他语言中的反射机制，但更为强大。它允许你在程序运行时，而不是编译时，去探索和操作类的结构、方法、属性等。例如，你可以动态地给一个已存在的类添加新的方法，或者替换掉一个方法的实现。
-   **为何在 JSBox 中引入**: JSBox 引入 Runtime 的最根本原因是为那些**JSBox 现有 API 无法满足或存在缺陷的功能**提供一个“备用方案”或“逃生舱口”。当你的脚本需要调用某个未被 JSBox 封装的 iOS 原生 API，或者需要实现非常底层的系统交互时，Runtime 就成了你的选择。
-   **重要警告**: Runtime 是一把“双刃剑”。它功能强大，但使用复杂，调试困难，且不当使用极易导致应用崩溃。因此，**在有其他解决方案时，应尽量避免使用 Runtime 接口**。

### JSBox 提供的 Runtime 接口

JSBox 封装了一系列接口，让 JavaScript 能够与 Objective-C Runtime 进行交互：

-   **`$objc(className)`**: 这是 Runtime 操作的入口。它用于获取一个 Objective-C 类（如 `"NSString"`, `"UIApplication"`）或一个 Objective-C 对象的引用。
-   **`invoke(methodName, arguments ...)`**: 这是执行原生操作的核心。通过它，你可以动态调用 Objective-C 类或对象的方法。`methodName` 是 Objective-C 的方法选择器（Selector），`arguments` 是方法的参数。
-   **`jsValue()`**: 将 Objective-C 对象或值转换为对应的 JavaScript 值。例如，将 `NSString` 转换为 `string`，`NSNumber` 转换为 `number`。
-   **`ocValue()`**: 将 JavaScript 值封装成 Objective-C 值。例如，将 `string` 转换为 `NSString`，`number` 转换为 `NSNumber`。
-   **`$define(object)`**: 动态创建一个 Objective-C 类。这允许你定义一个全新的原生类，并为其添加方法和属性。
-   **`$objc_retain(object)` / `$objc_release(object)`**: 用于手动管理 Objective-C 对象的内存引用计数。这是非常高级的用法，通常在处理原生对象的生命周期问题时使用，需要对 Objective-C 的内存管理有深入理解。

### 示例代码：使用 Runtime 获取设备电池状态

虽然 JSBox 提供了 `$device.info.battery.level` 来获取电池电量，但我们可以通过 Runtime 来演示如何直接调用原生 API 获取相同的信息，以理解其工作原理。

```javascript
// 1. 获取 UIDevice 类
const UIDevice = $objc("UIDevice");

// 2. 获取当前设备实例
const currentDevice = UIDevice.invoke("currentDevice");

// 3. 启用电池监控 (原生方法: setBatteryMonitoringEnabled:)
currentDevice.invoke("setBatteryMonitoringEnabled:", true);

// 4. 获取电池电量 (原生方法: batteryLevel)
const batteryLevelOC = currentDevice.invoke("batteryLevel");

// 5. 将 Objective-C 的浮点数转换为 JavaScript 的数字
const batteryLevelJS = batteryLevelOC.jsValue();

// 6. 获取电池状态 (原生方法: batteryState)
const batteryStateOC = currentDevice.invoke("batteryState");
const batteryStateJS = batteryStateOC.jsValue();

let stateString = "未知";
if (batteryStateJS === 0) stateString = "未知";
else if (batteryStateJS === 1) stateString = "未充电";
else if (batteryStateJS === 2) stateString = "充电中";
else if (batteryStateJS === 3) stateString = "已充满";

$ui.alert({
  title: "电池状态 (Runtime 获取)",
  message: `电量: ${Math.round(batteryLevelJS * 100)}%\n状态: ${stateString}`
});

// 7. 禁用电池监控 (可选)
currentDevice.invoke("setBatteryMonitoringEnabled:", false);
```

**代码解读**：

1.  `$objc("UIDevice")` 获取了 `UIDevice` 这个原生类。
2.  `UIDevice.invoke("currentDevice")` 调用了 `UIDevice` 的类方法 `currentDevice`，获取到当前设备的单例对象。
3.  `currentDevice.invoke("setBatteryMonitoringEnabled:", true)` 调用了实例方法 `setBatteryMonitoringEnabled:`，并传入 `true`。注意 Objective-C 方法名中的冒号 `:` 在 `invoke` 中对应一个参数。
4.  `currentDevice.invoke("batteryLevel")` 和 `currentDevice.invoke("batteryState")` 调用了实例方法获取电池电量和状态。
5.  **`jsValue()` 是关键**。由于 `batteryLevel` 和 `batteryState` 返回的是 Objective-C 的 `NSNumber` 或 `NSInteger` 类型，我们需要使用 `.jsValue()` 将它们转换为 JavaScript 可以直接操作的数字。

### 总结

Runtime 是 JSBox 赋予高级开发者的一把“利刃”，能够突破 JSBox API 的限制，直接与 iOS 底层交互。但它是一项高风险、高回报的技术，需要谨慎使用，并对 Objective-C 语言和其 Runtime 机制有深入的理解。 
