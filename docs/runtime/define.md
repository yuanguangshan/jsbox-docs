> 在 JSBox 中可以动态创建类和方法，用于实现更灵活的功能

# $define(object)

通过 `$define` 方法可以动态创建一个 Objective-C Class:

```js
$define({
  type: "MyHelper: NSObject",
  events: {
    instanceMethod: function() {
      $ui.alert("instance")
    },
    "indexPathForRow:inSection:": function(row, section) {
      $ui.alert(`row: ${row}, section: ${section}`)
    }
  },
  classEvents: {
    classMethod: function() {
      $ui.alert("class")
    }
  }
})
```

主要有三个部分：

- `type` 表示类名，声明方式与 Objective-C 相同
- `events` 中实现所有的实例方法
- `classEvents` 中实现所有的类方法

定义之后可以这样使用这个类：

```js
$objc("MyHelper").invoke("alloc.init").invoke("instanceMethod")
$objc("MyHelper").invoke("classMethod")
```

分别会弹出 `instance` 和 `class` 两个提示。

# $delegate(object)

在 Runtime 代码中，我们可能经常会需要创建一个 Delegate 对象，通过 $delegate 可以快速做到：

```js
const textView = $objc("UITextView").$new();
textView.$setDelegate($delegate({
  type: "UITextViewDelegate",
  events: {
    "textViewDidChange:": sender => {
      console.log(sender.$text().jsValue());
    }
  }
}));
```

简单来说，$delegate 使用和 $define 类似的定义。但无需指定类名，并且会自动创建实例。

---

## 文件内容解读与示例

### 用途说明

`$define` 和 `$delegate` 是 JSBox Runtime 环境中**极其高级且强大**的功能。它们允许你的脚本在运行时**动态地创建 Objective-C 类和协议遵循者**，从而实现对 iOS 原生框架的深度扩展和自定义行为。这对于实现 JSBox 未直接封装的原生 UI 组件、自定义数据源、或与复杂原生框架进行交互至关重要。

### 核心概念：动态创建原生类与协议遵循者

-   **动态性**: 传统的 Objective-C 类是在编译时定义的。而 `$define` 允许你在脚本运行时动态地创建新的类，并为其添加方法。
-   **桥接**: 这些动态创建的类和方法，可以被原生 Objective-C 代码调用，也可以在 JavaScript 中通过 `$objc` 模块进行操作。
-   **警告**: 这是非常高级的功能，要求开发者对 Objective-C 语言、Runtime 机制和 iOS 框架有深入理解。不当使用可能导致应用崩溃。

### `$define(options)` 详解

-   **用途**: 动态创建一个新的 Objective-C 类。这个类可以继承自任何现有的 Objective-C 类（如 `NSObject`, `UIView` 等），并实现自定义的实例方法和类方法。
-   **`options` 对象**: 
    -   **`type`**: 必填，新类的名称及其父类（例如 `"MyCustomView: UIView"`）。类名必须是唯一的。
    -   **`events`**: 一个 JavaScript 对象，键是 Objective-C **实例方法**的选择器（Selector），值是实现该方法的 JavaScript 函数。例如，`"sayHello"` 对应 `-[MyCustomClass sayHello]`。
    -   **`classEvents`**: 类似 `events`，但用于实现 **类方法**（静态方法）。

**示例**：动态创建一个简单的 Objective-C 类

```javascript
// 1. 动态定义一个继承自 NSObject 的 Objective-C 类
$define({
  type: "MyLogger: NSObject", // 类名:父类
  events: {
    // 实例方法：打印消息
    "logMessage:": function(message) {
      console.log(`[MyLogger Instance] ${message.jsValue()}`);
    },
    // 实例方法：计算两个数的和
    "addNumber:toNumber:": function(num1, num2) {
      return num1 + num2; // 返回值会自动桥接回原生
    }
  },
  classEvents: {
    // 类方法：获取共享实例
    "sharedLogger": function() {
      if (!this._sharedInstance) {
        this._sharedInstance = $objc("MyLogger").invoke("new");
      }
      return this._sharedInstance;
    }
  }
});

// 2. 使用动态创建的类
const loggerInstance = $objc("MyLogger").invoke("new"); // 创建实例
loggerInstance.invoke("logMessage:", "Hello from dynamic class!"); // 调用实例方法

const sum = loggerInstance.invoke("addNumber:toNumber:", 10, 20).jsValue(); // 调用带参数方法并获取返回值
console.log("Sum from dynamic class:", sum);

const sharedLogger = $objc("MyLogger").invoke("sharedLogger"); // 调用类方法
sharedLogger.invoke("logMessage:", "Hello from shared instance!");
```

### `$delegate(options)` 详解

-   **用途**: `$delegate` 是 `$define` 的一个便捷特例，专门用于快速创建**协议遵循者（Delegate）**或**数据源（DataSource）**对象。它会自动处理类名和实例创建，你只需关注要遵循的协议和实现的方法。
-   **`options` 对象**: 
    -   **`type`**: 必填，要遵循的 Objective-C 协议名称（例如 `"UITextViewDelegate"`, `"UITableViewDataSource"`）。
    -   **`events`**: 一个 JavaScript 对象，键是协议中定义的方法的选择器，值是实现该方法的 JavaScript 函数。

**示例**：创建一个 `UITextViewDelegate` 来监听文本变化

```javascript
// 1. 创建一个原生的 UITextView 实例
const textView = $objc("UITextView").invoke("new");
textView.invoke("setFrame", $rect(0, 0, 300, 200));
textView.invoke("setBackgroundColor:", $objc("UIColor").invoke("whiteColor"));
textView.invoke("setText:", "请在这里输入一些文字...");
textView.invoke("setEditable:", true);

// 2. 动态创建并设置 delegate
textView.invoke("setDelegate:", $delegate({
  type: "UITextViewDelegate", // 遵循 UITextViewDelegate 协议
  events: {
    // 实现协议方法：当文本内容改变时调用
    "textViewDidChange:": function(sender) {
      // sender 是 UITextView 实例本身
      console.log("文本改变:", sender.invoke("text").jsValue());
    },
    // 实现协议方法：当文本视图开始编辑时调用
    "textViewShouldBeginEditing:": function(sender) {
      console.log("开始编辑...");
      return true; // 返回 true 允许开始编辑
    }
  }
}));

// 3. 将原生 UITextView 显示在 JSBox UI 中
$ui.render({
  props: { title: "$delegate 示例" },
  views: [
    {
      type: "runtime", // 使用 runtime 组件显示原生视图
      props: { view: textView },
      layout: $layout.fill
    }
  ]
});
```

**代码解读**：

1.  我们首先创建了一个原生的 `UITextView` 实例。
2.  然后，通过 `textView.invoke("setDelegate:", ...)` 方法，我们将一个由 `$delegate()` 创建的对象设置为 `UITextView` 的代理。
3.  `$delegate()` 内部，我们指定了 `type: "UITextViewDelegate"`，并实现了协议中定义的 `textViewDidChange:` 和 `textViewShouldBeginEditing:` 方法。当 `UITextView` 发生文本变化或开始编辑时，这些 JavaScript 函数就会被调用。

### 总结

`$define` 和 `$delegate` 是 JSBox 中与 iOS Runtime 深度交互的强大工具。它们为高级开发者提供了在运行时扩展原生功能、实现自定义行为的能力。然而，它们的使用需要对 Objective-C 语言、Runtime 机制和 iOS 框架有深入的理解。 
