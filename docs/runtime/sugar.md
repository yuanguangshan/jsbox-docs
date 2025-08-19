# 语法糖

Runtime 的本质是在一门语言里面写另一门语言，也就是通过字符串的方式来反射调用，类似这样的做法写起来极为繁琐：

```js
const app = $objc("UIApplication").invoke("sharedApplication");
const url = $objc("NSURL").invoke("URLWithString", "https://sspai.com");
app.invoke("openURL", url);
```

所以我们需要一种语法糖来缓解这种复杂，在 v1.24.0 以后，我们引入了一种新的语法结构：

```js
const app = $objc("UIApplication").$sharedApplication();
const url = $objc("NSURL").$URLWithString("https://sspai.com");
app.$openURL(url);
```

这不仅可以少写很多代码，更能让反射代码看起来更像原生的代码（虽然本质上是一样的）。

# 规则

这个语法规则十分简单，一共只有三个规则：

- 使用 `$` 符号（美元符）来表示这是一个 Runtime 调用
- 使用 `_` 符号（下划线）来表示 Objective-C 方法名里面的 `:` 号
- 使用 `__` 符号（两个下划线）来表示 Objective-C 方法名里面的 `_` 号

例如：

```js
const app = $objc("UIApplication").$sharedApplication();
app.$sendAction_to_from_forEvent(action, target, null, null);
```

这个代码等同于 Objective-C 里面的：

```objc
UIApplication *app = [UIApplication sharedApplication];
[app sendAction:action to:target from:nil forEvent:nil];
```

更完善的例子，请参考这个使用新语法实现的 2048 小游戏：https://github.com/cyanzhong/xTeko/tree/master/extension-scripts/%242048

# 自动生成的类

有些 Objective-C 十分常用，所以 JSBox 现在默认支持，而无需再使用 $objc("ClassName") 去获得：

类名 | 参考
---|---
NSDictionary | https://developer.apple.com/documentation/foundation/nsdictionary
NSMutableDictionary | https://developer.apple.com/documentation/foundation/nsmutabledictionary
NSArray | https://developer.apple.com/documentation/foundation/nsarray
NSMutableArray | https://developer.apple.com/documentation/foundation/nsmutablearray
NSSet | https://developer.apple.com/documentation/foundation/nsset
NSMutableSet | https://developer.apple.com/documentation/foundation/nsmutableset
NSString | https://developer.apple.com/documentation/foundation/nsstring
NSMutableString | https://developer.apple.com/documentation/foundation/nsmutablestring
NSData | https://developer.apple.com/documentation/foundation/nsdata
NSMutableData | https://developer.apple.com/documentation/foundation/nsmutabledata
NSNumber | https://developer.apple.com/documentation/foundation/nsnumber
NSURL | https://developer.apple.com/documentation/foundation/nsurl
NSEnumerator | https://developer.apple.com/documentation/foundation/nsenumerator

这些你可以直接使用，就像是这样：

```js
const url = NSURL.$URLWithString("https://sspai.com");
```

其他的类你需要通过 $objc 定义，你可以使用它的返回值，但同时他也会生成一个同名的对象：

```js
const appClass = $objc("UIApplication");

var app = appClass.$sharedApplication();

// Or
var app = UIApplication.$sharedApplication();
```

也即 $objc("UIApplication") 会生成一个叫做 `UIApplication` 的对象方便你下次使用。

---

## 文件内容解读与示例

### 用途说明

本文档介绍了 JSBox Runtime 环境中的**语法糖（Syntactic Sugar）**。语法糖是一种编程语言特性，它不会增加语言的功能，但能让代码更简洁、更易读。在 JSBox 中，Runtime 语法糖极大地简化了 Objective-C 方法的调用方式，使其看起来更接近原生的 Objective-C 或 Swift 代码，从而提升了开发效率和代码的可维护性。

### 核心概念：简化 Runtime 调用

原始的 `invoke()` 调用虽然功能强大，但其语法（如 `invoke("methodName:with:"`）相对冗长且不直观。JSBox 的 Runtime 语法糖通过一套简单的规则，将这种复杂的反射调用转换为更简洁、更符合 JavaScript 习惯的点语法。

### 语法糖规则详解

1.  **`$` 符号前缀**: 所有通过语法糖调用的 Runtime 方法都以 `$` 符号开头。这是区分语法糖调用与普通 JavaScript 方法的关键。
    -   **示例**: `invoke("sharedApplication")` 变为 `.$sharedApplication()`。

2.  **`_` (单下划线) 替代 `:` (冒号)**: Objective-C 方法名中的冒号 `:` 用于分隔参数。在语法糖中，每个冒号都被一个单下划线 `_` 替换。
    -   **示例**: Objective-C 的 `[obj doSomething:param1 with:param2]` 变为 `obj.$doSomething_with(param1, param2)`。

3.  **`__` (双下划线) 替代 `_` (下划线)**: 如果 Objective-C 方法名本身包含下划线 `_`，在语法糖中需要用双下划线 `__` 来表示，以避免与冒号的转换规则冲突。
    -   **示例**: Objective-C 的 `[obj some_method]` 变为 `obj.$some__method()`。

### 自动生成的类 (便捷访问)

JSBox 自动将一些常用的 Objective-C 类（如 `NSDictionary`, `NSArray`, `NSString`, `NSURL` 等）作为全局变量暴露出来，可以直接使用，无需再通过 `$objc("ClassName")` 获取。

**重要提示**: 即使是其他类，当你第一次调用 `$objc("ClassName")` 时，JSBox 也会自动创建一个同名的全局变量，方便后续直接使用。

### 示例代码：对比 `invoke` 与语法糖

下面的示例将演示如何使用 `invoke()` 和新的语法糖来打开一个 URL，清晰地展示两者在代码简洁性上的差异。

```javascript
// 目标 URL
const targetUrl = "https://www.jsbox.com";

// --- 传统方式：使用 invoke() ---
console.log("--- 使用 invoke() ---");
const UIApplicationClass = $objc("UIApplication");
const NSURLClass = $objc("NSURL");

const appInstance_invoke = UIApplicationClass.invoke("sharedApplication");
const urlInstance_invoke = NSURLClass.invoke("URLWithString:", targetUrl);

appInstance_invoke.invoke("openURL:", urlInstance_invoke);
console.log("已尝试使用 invoke() 打开 URL。");

// --- 语法糖方式：更简洁 ---
console.log("\n--- 使用语法糖 ---");
// 常用类自动可用，无需 $objc("ClassName")
// const appInstance_sugar = UIApplication.$sharedApplication(); // 也可以这样写
// const urlInstance_sugar = NSURL.$URLWithString(targetUrl); // 也可以这样写

// 第一次 $objc("UIApplication") 会自动生成全局变量 UIApplication
const appInstance_sugar = UIApplication.$sharedApplication();
const urlInstance_sugar = NSURL.$URLWithString(targetUrl);

appInstance_sugar.$openURL(urlInstance_sugar);
console.log("已尝试使用语法糖打开 URL。");
```

**代码解读**：

-   **传统 `invoke()` 方式**: 代码相对冗长，需要多次调用 `invoke`，并且方法名需要包含冒号。
-   **语法糖方式**: 
    -   `UIApplication.$sharedApplication()`: `sharedApplication` 是一个无参数的类方法，直接在类名后加 `.$` 调用。
    -   `NSURL.$URLWithString(targetUrl)`: `URLWithString:` 是一个带参数的类方法，冒号 `:` 被替换为下划线 `_`，参数直接跟在括号内。
    -   `appInstance_sugar.$openURL(urlInstance_sugar)`: `openURL:` 是一个带参数的实例方法，同样冒号 `:` 被替换为下划线 `_`。

通过对比可以看出，语法糖极大地提升了代码的简洁性和可读性，使得 Runtime 调用更符合 JavaScript 的习惯。

### 总结

Runtime 语法糖极大地提升了 JSBox 中 Runtime 代码的可读性和编写效率。虽然它只是语法上的简化，但对于频繁与原生 API 交互的开发者来说，是不可或缺的工具。它使得 JSBox 脚本在调用原生功能时，能够写出更接近原生语言风格的代码。 
