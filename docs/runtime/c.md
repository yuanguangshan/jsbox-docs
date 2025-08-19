# $defc

JSBox 也提供了调用 C 函数的方法，对于 iOS 提供的 C 函数，你可以通过这个方式来引入：

```js
$defc("NSClassFromString", "Class, NSString *")
```

第一个参数是函数名，第二个字符串依次是返回值类型和各个参数的类型（类似 $block 的定义）。

定义之后你就可以这样使用这个 C 函数了：

```js
NSClassFromString("NSURL")
```

类似的，对于这样一个 C 函数：`int func(void *ptr, NSObject *obj, float num)` 则需要这样定义：

```js
$defc("func", "int, void *, NSObject *, float")
```

注：目前不支持在 C 函数里面传递 struct 和 block 类型。

---

## 文件内容解读与示例

### 用途说明

`$defc` 是 JSBox Runtime 环境中一个**极其高级且底层**的功能，它允许你的 JavaScript 脚本直接调用 iOS 系统中暴露的 **C 语言函数**。这为那些 JSBox 或 Objective-C Runtime 无法直接满足的特定需求提供了最终的“逃生舱口”，例如访问某些未被封装的系统底层功能。然而，它要求开发者对 C 语言、iOS 底层 API 和类型编码有深入的理解。

### 核心概念：C 函数的桥接

-   **C 函数**: iOS 操作系统底层有大量的 C 语言函数，它们是系统服务和库的基础。`$defc` 的作用就是为这些 C 函数创建 JavaScript 包装器，使得它们可以直接在 JS 中被调用。
-   **类型签名**: 由于 JavaScript 是动态类型语言，而 C 语言是静态类型语言，因此在调用 C 函数时，必须通过一个**签名字符串**明确告知 JSBox 该 C 函数的返回值类型和所有参数的类型。这是确保正确调用和数据转换的关键。

### `$defc(functionName, signature)` 详解

-   **`functionName`**: 
    -   **用途**: 你想要调用的 C 函数的名称。这个函数必须是系统库中已有的、可导出的函数。
    -   **示例**: `"NSLog"`, `"dispatch_time"`。

-   **`signature` (签名字符串)**: 
    -   **用途**: 这是最关键的部分。它是一个字符串，严格定义了 C 函数的**返回值类型**和**所有参数的类型**，类型之间用逗号 `,` 分隔。顺序必须与 C 函数的实际定义一致。
    -   **类型编码**: 你需要了解 Objective-C 的类型编码（Type Encoding），例如：
        -   `v`: `void` (无返回值)
        -   `i`: `int` (整数)
        -   `f`: `float` (浮点数)
        -   `d`: `double` (双精度浮点数)
        -   `@`: `id` (Objective-C 对象)
        -   `#`: `Class` (Objective-C 类)
        -   `*`: `char *` (C 字符串)
        -   `B`: `bool` (布尔值)
    -   **示例**: `"void, NSString *"` 表示一个没有返回值，接收一个 `NSString *` 类型参数的函数。

-   **限制**: 目前 `$defc` 不支持在 C 函数中传递 `struct` 和 `block` 类型作为参数。

### 示例代码：调用 `NSLog` 函数

`NSLog` 是 Objective-C 中一个常用的 C 函数，用于向控制台输出日志。虽然 JSBox 提供了 `console.log`，但通过 `NSLog` 可以更好地理解 `$defc` 的用法。

```javascript
// 1. 定义 NSLog C 函数
// 签名: void (无返回值), id (第一个参数，通常是格式化字符串), ... (可变参数)
// 这里我们简化为只接受一个 NSString * 参数
$defc("NSLog", "void, NSString *");

// 2. 调用 NSLog 函数
// 注意：JSBox 会自动将 JavaScript 字符串转换为 NSString *
NSLog("Hello from NSLog via $defc!");

// 也可以传递变量
const message = "这是一个来自 JSBox 的 C 函数调用示例。";
NSLog(message);

// 尝试调用一个需要多个参数的 C 函数 (概念性示例)
// 假设有一个 C 函数 `void printNumbers(int a, int b)`
// $defc("printNumbers", "void, int, int");
// printNumbers(10, 20);
```

**代码解读**：

1.  我们使用 `$defc("NSLog", "void, NSString *")` 来定义 `NSLog` 函数。`"void"` 表示没有返回值，`"NSString *"` 表示它接受一个 Objective-C 字符串指针作为参数。JSBox 会自动将我们传入的 JavaScript 字符串转换为 `NSString *`。
2.  定义完成后，我们就可以像调用普通 JavaScript 函数一样直接调用 `NSLog()`。

### 总结

`$defc` 是 JSBox 中一个非常底层且强大的工具，它为高级开发者提供了直接访问 iOS 系统 C 函数的能力。然而，它的使用需要对 C 语言和 Objective-C 的类型编码有深入的理解，不当使用可能导致应用崩溃。 
