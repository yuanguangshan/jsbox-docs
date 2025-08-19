# 在捷径应用里运行 JavaScript

JSBox 还能为捷径应用增加运行 JavaScript 的功能，当你需要在捷径应用里面处理复杂的数据时，不如通过这个方案实现。

由于捷径应用并不支持第三方应用处理输入和输出数据，所以这套逻辑借助于剪贴板实现：

- 将 JavaScript 拷贝到剪贴板
- 执行 JSBox 提供的运行 JavaScript 动作
- 读取剪贴板中的结果

简单说，JSBox 执行动作的时候会读取剪贴板中的 JavaScript，执行之后把结果写回到剪贴板，进而完成了通过 JSBox 执行 JavaScript 的过程。

同样的，异步任务需要通过 $intents.finish 来告诉捷径应用已经执行完成：

```js
const a = "Hello";
const b = "World";
const result = [a, b].join(", ");

$intents.finish(result);
```

你可以通过安装这个 Shortcut 体验这个功能：[Run JavaScript](shortcuts://import-workflow?url=https://github.com/cyanzhong/xTeko/raw/master/extension-demos/scripting.shortcut&name=Run%20JavaScript)。

---

## 文件内容解读与示例

### 用途说明

本文档介绍了如何在“快捷指令”应用中**动态执行 JavaScript 代码**。这为快捷指令提供了一种强大的扩展能力，允许用户在自动化流程中处理复杂的逻辑和数据操作，而这些是快捷指令原生功能难以实现的。

### 核心机制：基于剪贴板的数据交换

由于 iOS 的沙盒机制限制，快捷指令无法直接向 JSBox 传递代码并接收返回值。因此，JSBox 设计了一套巧妙的**基于剪贴板**的通信机制：

1.  **输入**: 在快捷指令流程中，首先使用“设置剪贴板”动作，将要执行的 JavaScript 代码（作为文本）放入系统剪贴板。
2.  **执行**: 接着，调用 JSBox 提供的“运行 JavaScript”快捷指令动作。这个动作会自动读取剪贴板中的文本，将其作为代码执行。
3.  **输出**: 脚本执行完毕后，JSBox 会将结果（通过 `$intents.finish()` 返回）写回到剪贴板。
4.  **获取结果**: 在快捷指令中，紧接着使用“获取剪贴板”动作，就能得到脚本的运行结果，并用于后续步骤。

### 关键函数: `$intents.finish(result)`

在通过这种方式运行时，`$intents.finish(result)` 是一个至关重要的函数。

-   **作用**: 它的作用是**结束脚本**并将 `result` 作为最终结果返回给快捷指令。
-   **异步支持**: 这个函数同样支持异步操作。如果你的代码包含网络请求等耗时任务，可以在任务完成后的回调函数中调用 `$intents.finish()`，以确保快捷指令能等到正确的最终结果。
-   **返回类型**: 返回的 `result` 可以是字符串、数字、数组或字典等，JSBox 会将其序列化后写入剪贴板。

### 示例：计算两个数字的和

下面的示例演示了如何创建一个快捷指令，来计算两个数字的和。

**1. 在“快捷指令”应用中创建快捷指令**：

-   打开“快捷指令”应用，创建新快捷指令。
-   添加一个“文本”动作，在其中输入以下 JavaScript 代码：
    ```javascript
    // 从输入中获取数字
    const numbers = $context.query.numbers;
    const result = numbers[0] + numbers[1];

    // 将结果返回
    $intents.finish(result);
    ```
-   添加“设置剪贴板”动作，将上面的“文本”作为内容。
-   添加 JSBox 的“运行 JavaScript”动作。
-   添加“获取剪贴板”动作。
-   添加“显示提醒”动作，将“获取剪贴板”的结果作为显示内容。

**2. 如何传递参数**

为了让这个快捷指令更通用，我们可以使用快捷指令的“每次都询问”功能来动态输入数字。

-   修改快捷指令，在“文本”动作之前，添加两个“要求输入”动作，类型都设置为“数字”。
-   在“文本”动作中，将代码修改为：
    ```javascript
    // 快捷指令的输入会通过 $context.query 传入
    const num1 = $context.query.input[0];
    const num2 = $context.query.input[1];
    $intents.finish(num1 + num2);
    ```
-   在 JSBox 的“运行 JavaScript”动作中，将“输入”参数设置为前面两个“要求输入”动作的结果。

**预期结果**：

运行此快捷指令，会依次弹出两个输入框让你输入数字。输入后，JSBox 会执行计算并将结果显示在提醒中。

### 总结

通过剪贴板作为桥梁，JSBox 极大地增强了“快捷指令”的编程能力。这使得用户可以在自动流程中嵌入复杂的、动态的 JavaScript 逻辑，实现更高级的自动化任务。