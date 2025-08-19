# Objective-C Blocks

Block 是 Objective-C 里面一种特殊的类型，虽然不完全相同但也很类似其他语言里面的闭包，如果用更通俗一点的语言来解释的话，你可以理解成 Block 很像一个函数，他能捕获外层的变量，并且能作为参数、变量甚至返回值。

关于 Block 的更多内容请参考 Apple 的官方文档：https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithBlocks/WorkingwithBlocks.html

本文档的内容是介绍如何在 JSBox 的 Runtime 环境里面使用 Blocks。

# $block

JSBox 里面使用 $block 来定义一个 block，例如：

```js
const handler = $block("void, UITableViewRowAction *, NSIndexPath *", (action, indexPath) => {
  $ui.alert("Action")
});
```

即使用一个字符串来按顺序声明函数的返回值和参数数据类型，然后使用一个函数来定义 Block 的函数体，这个 Block 在 Objective-C 实现的时候长这样：

```objc
void(^handler)(UITableViewRowAction *, NSIndexPath *) = ^(UITableViewRowAction *action, NSIndexPath *indexPath) {
  // Alert
};
```

请注意返回值一定要声明，否则将不能正确地识别。

这里有一个完整的使用 Runtime 构建 TableView 的例子：

```js
//-- Create window --//

$ui.render()

//-- Cell --//

$define({
  type: "TableCell: UITableViewCell"
})

//-- TableView --//

$define({
  type: "TableView: UITableView",
  events: {
    "init": function() {
      self = self.invoke("super.init")
      self.invoke("setTableFooterView", $objc("UIView").invoke("new"))
      self.invoke("registerClass:forCellReuseIdentifier:", $objc("TableCell").invoke("class"), "identifier")
      return self
    }
  }
})

//-- Manager --//

$define({
  type: "Manager: NSObject <UITableViewDelegate, UITableViewDataSource>",
  events: {
    "tableView:numberOfRowsInSection:": function(tableView, section) {
      return 5
    },
    "tableView:cellForRowAtIndexPath:": function(tableView, indexPath) {
      const cell = tableView.invoke("dequeueReusableCellWithIdentifier:forIndexPath:", "identifier", indexPath);
      cell.invoke("textLabel").invoke("setText", `Row: ${indexPath.invoke("row")}`)
      return cell
    },
    "tableView:didSelectRowAtIndexPath:": function(tableView, indexPath) {
      tableView.invoke("deselectRowAtIndexPath:animated:", indexPath, true)
      const cell = tableView.invoke("cellForRowAtIndexPath:", indexPath);
      const text = cell.invoke("textLabel.text").jsValue();
      $ui.alert(`Tapped: ${text}`)
    },
    "tableView:editActionsForRowAtIndexPath:": function(tableView, indexPath) {
      const handler = $block("void, UITableViewRowAction *, NSIndexPath *", (action, indexPath) => {
        $ui.alert("Action")
      });
      const action = $objc("UITableViewRowAction").invoke("rowActionWithStyle:title:handler:", 1, "Foobar", handler);
      return [action]
    }
  }
})

const window = $ui.window.ocValue();
const manager = $objc("Manager").invoke("new");

const table = $objc("TableView").invoke("new");
table.invoke("setFrame", window.invoke("bounds"))
table.invoke("setDelegate", manager)
table.invoke("setDataSource", manager)
window.invoke("addSubview", table)
```

---

## 文件内容解读与示例

### 用途说明

本文档介绍了 **Objective-C Blocks** 这一概念，以及如何在 JSBox 的 Runtime 环境中定义和使用它们。Blocks 是 Objective-C 中一种强大的特性，类似于其他语言中的闭包或匿名函数，它们能够捕获外部变量，并作为参数、变量或返回值传递。在 iOS 开发中，Blocks 广泛用于异步操作的回调、动画完成处理、以及委托（Delegate）模式的实现。

### 核心概念：JavaScript 函数与 Objective-C Block 的桥接

JSBox 通过 `$block()` 函数，允许你用 JavaScript 函数来定义一个 Objective-C Block，从而实现 JS 和原生代码之间的无缝回调。这使得你可以直接与那些需要 Block 作为参数的原生 iOS API 进行交互。

### `$block(signature, handler)` 详解

-   **`signature` (签名字符串)**: 
    -   **用途**: 这是最关键的部分。它是一个字符串，**严格定义了 Block 的返回值类型和参数数据类型**，顺序必须与 Objective-C 的 Block 定义一致。例如，`"void, id, NSString *"` 表示一个没有返回值，接收一个 `id` 类型对象和一个 `NSString *` 类型字符串作为参数的 Block。
    -   **重要性**: 必须准确无误地声明类型，否则 Block 将无法正确工作或导致崩溃。你需要了解 Objective-C 的类型编码。

-   **`handler` (处理函数)**: 
    -   **用途**: 一个标准的 JavaScript 函数，它实现了 Block 的具体逻辑。当原生代码调用这个 Block 时，`handler` 函数就会被执行。
    -   **参数**: `handler` 函数的参数必须与 `signature` 中声明的参数类型和顺序一致。
    -   **上下文**: 在 `handler` 内部，你可以像普通的 JavaScript 函数一样编写逻辑，并访问外部作用域的变量。

### 示例代码：使用 Block 处理原生 Alert 按钮点击

下面的示例将演示如何使用 `$block` 来为原生的 `UIAlertController` 的按钮添加点击事件处理。这比 JSBox 封装的 `$ui.alert` 更底层，但能让你完全控制原生 Alert 的行为。

```javascript
// 1. 定义一个 Block 来处理 Alert 按钮点击事件
// 签名: void (没有返回值), UIAlertAction * (第一个参数), NSInteger (第二个参数，通常是按钮索引)
const alertActionHandler = $block("void, UIAlertAction *, NSInteger", (action, buttonIndex) => {
  const buttonTitle = action.invoke("title").jsValue(); // 获取被点击按钮的标题
  $ui.alert(`你点击了按钮: ${buttonTitle} (索引: ${buttonIndex})`);
});

// 2. 创建原生的 UIAlertController
const alertController = $objc("UIAlertController").invoke("alertControllerWithTitle:message:preferredStyle:",
  "原生 Alert",
  "这是一个通过 Runtime 创建的原生 Alert。",
  0 // UIAlertControllerStyleAlert
);

// 3. 添加按钮到 Alert
// 添加一个“确定”按钮
const okAction = $objc("UIAlertAction").invoke("actionWithTitle:style:handler:",
  "确定",
  0, // UIAlertActionStyleDefault
  alertActionHandler // 传入我们定义的 Block
);
alertController.invoke("addAction:", okAction);

// 添加一个“取消”按钮
const cancelAction = $objc("UIAlertAction").invoke("actionWithTitle:style:handler:",
  "取消",
  1, // UIAlertActionStyleCancel
  alertActionHandler // 传入我们定义的 Block
);
alertController.invoke("addAction:", cancelAction);

// 4. 显示 Alert
$ui.vc.presentViewController_animated_completion(alertController, true, null);
```

**代码解读**：

1.  我们首先使用 `$block()` 定义了 `alertActionHandler`。其 `signature` 字符串 `"void, UIAlertAction *, NSInteger"` 严格声明了 Block 的返回值类型和两个参数的类型。
2.  `handler` 函数的参数 `action` 和 `buttonIndex` 与 `signature` 中的类型对应。`action` 是被点击的 `UIAlertAction` 对象，`buttonIndex` 是按钮的索引。
3.  我们通过 `$objc()` 创建了原生的 `UIAlertController` 和 `UIAlertAction` 实例。
4.  在添加 `UIAlertAction` 时，我们将 `alertActionHandler` 作为 `handler` 参数传入。当用户点击 Alert 上的按钮时，原生系统会调用这个 Block，从而执行我们 JavaScript 中定义的 `handler` 函数。

`$block` 是 JSBox 中与 Objective-C Runtime 深度交互的关键。它允许你将 JavaScript 函数作为回调传递给原生 API，从而实现对 iOS 底层功能的精细控制。 
