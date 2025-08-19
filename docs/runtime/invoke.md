> 通过 JSBox 提供的接口，动态的与原生接口进行交互

# invoke(methodName, arguments ...)

当我们通过 `$objc("")` 拿到 Objective-C 类时，可以通过 `invoke` 方法动态的执行他的方法：

```js
const label = $objc("UILabel").invoke("alloc.init");
label.invoke("setText", "Runtime")
```

上面的代码动态创建了一个 `label`，同时把文字设置成了 `Runtime`。

`invoke("alloc.init")` 也等价于 `invoke("alloc").invoke("init")`。

如果你希望一次性导入多个 Objective-C 类，可以这么做：

```js
$objc("UIColor, UIApplication, NSIndexPath");

const color = UIColor.$redColor();
const application = UIApplication.$sharedApplication();
```

# selector

以 `NSIndexPath` 为例，Objective-C 中：

```objc
NSIndexPath *indexPath = [NSIndexPath indexPathForRow:0 inSection:0];
```

在 JSBox Runtime 中这么实现：

```js
const indexPath = $objc("NSIndexPath").invoke("indexPathForRow:inSection:", 0, 0);
```

`indexPathForRow:inSection:` 就是这个方法的 selector，而后面的就是参数列表。

---

## 文件内容解读与示例

### 用途说明

`invoke()` 方法是 JSBox Runtime 环境中**最核心、最常用**的操作。它允许你的 JavaScript 脚本**动态地调用 Objective-C 类或对象的方法**。这是实现与 iOS 原生框架深度交互的关键，也是 Runtime“黑魔法”得以施展的基础。

### 核心概念：Objective-C 方法调用

-   **作用对象**: `invoke()` 方法总是作用于一个通过 `$objc()` 获取的 Objective-C 类（如 `UILabel`）或一个 Objective-C 对象实例（如 `UILabel` 的一个实例）。
-   **方法选择器 (Selector)**: `invoke()` 的第一个参数是 Objective-C 的“方法选择器”（Method Selector）。它是一个字符串，精确地指定了要调用的方法，包括参数的冒号 `:`。
    -   **无参数方法**: 方法选择器就是方法名本身，例如 `"new"`、`"redColor"`。
    -   **带参数方法**: 方法选择器由方法名和每个参数标签组成，每个参数标签后跟一个冒号 `:`。例如，`"setText:"`、`"initWithFrame:"`、`"indexPathForRow:inSection:"`。
-   **参数传递**: `invoke()` 的后续参数会按照顺序传递给 Objective-C 方法。JSBox 会自动进行 JavaScript 类型到 Objective-C 类型的桥接（例如 `string` 到 `NSString *`，`number` 到 `NSNumber *`）。
-   **返回值**: `invoke()` 调用通常会返回一个 Objective-C 对象。如果需要将其用于 JavaScript 逻辑，需要使用 `.jsValue()` 方法进行转换。

### `invoke()` 的特性

-   **链式调用**: 多个 `invoke()` 调用可以链式连接，例如 `object.invoke("method1").invoke("method2")`。这在 Objective-C 中很常见，例如 `[[[UILabel alloc] init] setText:@"Hello"]`。
-   **批量导入类**: `$objc("Class1, Class2, ...")` 可以一次性导入多个 Objective-C 类，方便后续使用，例如 `$objc("UIColor, UIApplication");`。

### 示例代码：动态创建和操作原生 `UILabel`

下面的示例将演示如何使用 `invoke()` 方法动态创建一个原生的 `UILabel`，设置其文本和颜色，并将其添加到 JSBox 的 UI 中。

```javascript
// 1. 获取 UILabel 类
const UILabel = $objc("UILabel");

// 2. 创建 UILabel 实例
// 链式调用 alloc 和 init 方法
const myLabel = UILabel.invoke("alloc.init");

// 3. 设置 UILabel 的属性
// 设置文本：调用 setText: 方法
myLabel.invoke("setText:", "Hello from Runtime!");

// 设置字体：调用 setFont: 方法，需要一个 UIFont 对象
// $objc("UIFont").invoke("systemFontOfSize:", 24) 创建一个系统字体
myLabel.invoke("setFont:", $objc("UIFont").invoke("systemFontOfSize:", 24));

// 设置文本颜色：调用 setTextColor: 方法，需要一个 UIColor 对象
// $objc("UIColor").invoke("redColor") 获取一个红色 UIColor 对象
myLabel.invoke("setTextColor:", $objc("UIColor").invoke("redColor"));

// 设置文本居中对齐：调用 setTextAlignment: 方法
// 1 对应 NSTextAlignmentCenter
myLabel.invoke("setTextAlignment:", 1);

// 4. 将原生 UILabel 添加到 JSBox UI 中
$ui.render({
  props: { title: "Invoke 示例" },
  views: [
    {
      type: "runtime", // 使用 runtime 组件来显示原生视图
      props: {
        view: myLabel // 将原生 UILabel 实例赋值给 view 属性
      },
      layout: make => {
        make.center.equalTo(view.super);
        make.size.equalTo($size(300, 50)); // 设置 runtime 容器的大小
      }
    }
  ]
});
```

**代码解读**：

-   `$objc("UILabel")` 获取了 `UILabel` 这个 Objective-C 类。
-   `UILabel.invoke("alloc.init")` 链式调用了 `alloc` 和 `init` 方法来创建 `UILabel` 的实例。
-   `myLabel.invoke("setText:", "Hello from Runtime!")` 调用了 `setText:` 方法，将 JavaScript 字符串 `"Hello from Runtime!"` 桥接为 `NSString *` 传递给原生方法。
-   `myLabel.invoke("setFont:", $objc("UIFont").invoke("systemFontOfSize:", 24))` 演示了如何将另一个 Runtime 调用（创建 `UIFont` 对象）的结果作为参数传递。
-   最后，通过 `runtime` 组件将这个原生的 `myLabel` 实例显示在 JSBox 的 UI 中。

`invoke()` 是 JSBox Runtime 的核心。掌握如何正确使用方法选择器和参数传递，是解锁 iOS 原生 API 强大功能的关键。 
