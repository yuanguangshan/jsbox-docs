> 原生类型与 Runtime 类型的相互转换

# 类型转换

当使用上述 `invoke` 接口时，返回的数据类型是一个 Objective-C 类型，而 JSBox 其他的接口生成的都是 JavaScript 类型。

这两种代码要混合在一起的时候，需要通过下面两个方法进行转换：

- `jsValue()` 从 Objective-C 值转换成 JavaScript 值
- `ocValue()` 将 JavaScript 值封装成 Objective-C 值

# 例子

例如 JSBox 提供的 `$color` 生成的是一个 JavaScript 值：

```js
const color1 = $color("red");
```

这种值被应用在 invoke 方法时需要：`color1.ocValue()` 才行。

而 invoke 生成的都是 ocValue:

```js
const color2 = $objc("UIColor").invoke("grayColor");
```

这种值在应用在 JSBox 接口时需要调用 `color2.jsValue()` 才行，例如：

```js
props: {
  bgcolor: color2.jsValue()
}
```

通过这两个方法，我们可以在 Runtime 环境和原始环境中穿梭，从而将两种代码混合起来。

---

## 文件内容解读与示例

### 用途说明

本文档是 JSBox Runtime 环境中**类型转换**的关键指南。它详细阐述了如何在通过 `$objc` 模块获取的 Objective-C 值与 JSBox 高级 API 生成的 JavaScript 值之间进行相互转换。理解这些转换机制是编写混合使用 Runtime 和 JSBox 现有 API 的脚本的基础。

### 核心概念：双向桥接与类型不匹配

JSBox 脚本同时运行在 JavaScript 环境和 Objective-C Runtime 环境中。虽然 JSBox 自动桥接了基本数据类型（如 `string` 到 `NSString`，`number` 到 `NSNumber`），但对于更复杂的对象（如颜色、字体、图片），`$objc` 返回的是原生 Objective-C 对象，而 JSBox 高级 API（如 `$color()`, `$image()`）返回的是 JSBox 封装的 JavaScript 对象。这两种对象不能直接互用，需要显式转换。

-   **`jsValue()`**: 将一个 Objective-C 对象或值转换为对应的 JavaScript 值。
-   **`ocValue()`**: 将一个 JSBox 封装的 JavaScript 值转换为对应的 Objective-C 值。

### `jsValue()` 详解

-   **用途**: 当你通过 `invoke()` 调用原生方法得到一个 Objective-C 对象，但需要将其用于 JSBox 的 UI 组件属性（如 `bgcolor`）、`console.log`、或进行 JavaScript 运算时。
-   **示例**: 
    ```javascript
    // 获取原生的灰色 UIColor 对象
    const nativeGrayColor = $objc("UIColor").invoke("grayColor");

    // 错误用法：直接将原生对象赋值给 JSBox UI 属性
    // $ui.render({ views: [{ type: "view", props: { bgcolor: nativeGrayColor } }] }); 

    // 正确用法：使用 .jsValue() 转换为 JavaScript 值
    $ui.render({
      views: [
        {
          type: "view",
          props: { bgcolor: nativeGrayColor.jsValue() },
          layout: $layout.fill
        }
      ]
    });
    ```

### `ocValue()` 详解

-   **用途**: 当你使用 JSBox 高级 API 创建了一个对象（如 `$color()`, `$image()`），但需要将其作为参数传递给一个通过 `invoke()` 调用的原生 Objective-C 方法时。
-   **示例**: 
    ```javascript
    // 创建一个 JSBox 封装的蓝色 $color 对象
    const jsBlueColor = $color("blue");

    // 错误用法：直接将 JSBox 对象作为参数传递给原生方法
    // $objc("UIView").invoke("setBackgroundColor:", jsBlueColor);

    // 正确用法：使用 .ocValue() 转换为 Objective-C 值
    const myView = $objc("UIView").invoke("new");
    myView.invoke("setBackgroundColor:", jsBlueColor.ocValue());

    // 将原生视图添加到 JSBox UI 中显示
    $ui.render({
      views: [
        {
          type: "runtime",
          props: { view: myView },
          layout: $layout.fill
        }
      ]
    });
    ```

### 示例代码：混合使用与类型转换

下面的示例将创建一个 UI，其中包含一个 JSBox `view` 和一个通过 `runtime` 组件显示的原生 `UILabel`。我们将演示如何使用 `jsValue()` 和 `ocValue()` 在两者之间传递颜色。

```javascript
// 1. 创建一个原生的 UILabel 实例
const nativeLabel = $objc("UILabel").invoke("new");
nativeLabel.invoke("setText:", "原生标签");
nativeLabel.invoke("setTextAlignment:", 1); // 居中

$ui.render({
  props: { title: "类型转换示例" },
  views: [
    {
      type: "view",
      props: {
        id: "jsbox-view",
        bgcolor: $color("red"), // JSBox 颜色
        cornerRadius: 10
      },
      layout: make => {
        make.top.inset(50);
        make.centerX.equalTo(view.super);
        make.size.equalTo($size(150, 50));
      }
    },
    {
      type: "runtime", // 用于显示原生 UILabel
      props: {
        view: nativeLabel
      },
      layout: make => {
        make.top.equalTo($("jsbox-view").bottom).offset(20);
        make.centerX.equalTo(view.super);
        make.size.equalTo($size(150, 50));
      }
    },
    {
      type: "button",
      props: { title: "交换颜色" },
      layout: make => {
        make.top.equalTo(view.prev.bottom).offset(30);
        make.centerX.equalTo(view.super);
        make.width.equalTo(100);
      },
      events: {
        tapped: () => {
          // 获取 JSBox 视图的当前背景色 (JS 值)
          const jsboxViewColor = $("jsbox-view").bgcolor;

          // 获取原生标签的当前文本颜色 (OC 值，需要 .jsValue() 转换)
          const nativeLabelColorOC = nativeLabel.invoke("textColor");
          const nativeLabelColorJS = nativeLabelColorOC.jsValue();

          // 交换颜色：
          // 将原生标签的颜色赋给 JSBox 视图 (OC -> JS)
          $("jsbox-view").bgcolor = nativeLabelColorJS;

          // 将 JSBox 视图的颜色赋给原生标签 (JS -> OC)
          nativeLabel.invoke("setTextColor:", jsboxViewColor.ocValue());
        }
      }
    }
  ]
});

// 初始设置原生标签的颜色
nativeLabel.invoke("setTextColor:", $objc("UIColor").invoke("blueColor"));
```

**代码解读**：

1.  `jsbox-view` 的 `bgcolor` 属性直接接受 JSBox 封装的 `$color("red")`。
2.  `nativeLabel` 是一个原生 `UILabel`。当我们需要获取它的 `textColor` 时，`nativeLabel.invoke("textColor")` 返回的是一个 Objective-C `UIColor` 对象，因此需要调用 `.jsValue()` 才能赋值给 JSBox 视图的 `bgcolor`。
3.  反之，当我们需要将 JSBox 视图的 `bgcolor`（一个 JSBox `$color` 对象）赋值给原生 `UILabel` 的 `textColor` 时，需要调用 `.ocValue()` 将其转换为 Objective-C `UIColor` 对象。

`jsValue()` 和 `ocValue()` 是 JSBox Runtime 中进行类型转换的“显式桥梁”。理解何时以及如何使用它们，是编写混合使用 JSBox 高级 API 和底层 Runtime 功能的脚本的关键。 
