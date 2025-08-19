# $editor

在 JSBox 里，你可以通过 $editor 实现代码编辑器插件，用来控制编辑器的行为，起到辅助编辑的作用。

你可以通过这些接口实现很多功能，例如自己的代码排版规则，或是文字编码工具。

# $editor.text

获取或设置编辑器内的全部文本：

```js
const text = $editor.text;

$editor.text = "Hey!";
```

# $editor.view

获取当前编辑器正在使用的视图：

```js
const editorView = $editor.view;
editorView.alpha = 0.5;
```

# $editor.selectedRange

获取或设置编辑器内选中的范围：

```js
const range = $editor.selectedRange;

$editor.selectedRange = $(0, 10);
```

# $editor.selectedText

获取或设置编辑器内选中的文本：

```js
const text = $editor.selectedText;

$editor.selectedText = "Hey!";
```

# $editor.hasText

检查编辑器是否有内容：

```js
const hasText = $editor.hasText;
```

# $editor.isActive

获取编辑器当前是否处于激活状态：

```js
const isActive = $editor.isActive;
```

# $editor.canUndo

判断当前的编辑器是否可以执行撤销操作：

```js
const canUndo = $editor.canUndo;
```

# $editor.canRedo

判断当前的编辑器是否可以执行重做操作：

```js
const canRedo = $editor.canRedo;
```

# $editor.save()

保存当前编辑器的改动：

```js
$editor.save();
```

# $editor.undo()

对当前编辑器执行撤销操作：

```js
$editor.undo();
```

# $editor.redo()

对当前编辑器执行重做操作：

```js
$editor.redo();
```

# $editor.activate()

激活当前的编辑器：

```js
$editor.activate()
```

# $editor.deactivate()

结束当前编辑器的激活状态：

```js
$editor.deactivate()
```

# $editor.insertText(text)

插入文本到当前选中区域：

```js
$editor.insertText("Hello");
```

# $editor.deleteBackward()

删除光标前的字符：

```js
$editor.deleteBackward();
```

# $editor.textInRange(range)

获取当前编辑器里某个范围内的文本：

```js
const text = $editor.textInRange($range(0, 10));
```

# $editor.setTextInRange(text, range)

设置当前编辑器里某个范围内的文本：

```js
$editor.setTextInRange("Hey!", $range(0, 10));
```

---

## 文件内容解读与示例

### 用途说明

`$editor` API 是 JSBox 中一个非常独特且强大的模块，它允许你的脚本**直接与 JSBox 内置的代码编辑器进行交互和控制**。通过这个 API，你可以实现各种编辑器插件，例如：

-   代码格式化工具
-   快速插入代码片段
-   文本编码/解码工具
-   批量修改文本内容
-   自定义代码辅助功能

它将 JSBox 的开发环境本身也变成了可编程的对象，极大地扩展了脚本的自动化能力。

### 核心概念与属性

`$editor` 是一个全局对象，它提供了对当前编辑器状态的访问和操作方法。

-   **`$editor.text`**: 获取或设置编辑器中的**全部文本内容**。这是进行全局操作的基础。
-   **`$editor.selectedRange`**: 获取或设置编辑器中当前**选中的文本范围**。它返回一个 `$range` 对象（`{ location: number, length: number }`）。这是对选中区域进行操作的关键。
-   **`$editor.selectedText`**: 获取或设置编辑器中当前**选中的文本内容**。如果你只想获取或替换选中部分，这个属性很方便。
-   **`$editor.insertText(text)`**: 在当前光标位置插入文本，如果当前有选中区域，则替换选中区域。
-   **`$editor.setTextInRange(text, range)`**: 在指定的 `range` 范围内替换文本。这是进行精确修改的强大方法。
-   **`$editor.undo()` / `$editor.redo()`**: 执行编辑器的撤销和重做操作。
-   **`$editor.save()`**: 保存当前编辑器的改动。

### 示例代码：为选中代码添加注释

下面的示例将创建一个简单的编辑器插件。当你选中一段代码并运行此脚本时，它会为选中的每一行代码添加 `// ` 注释。

```javascript
function commentSelectedCode() {
  const range = $editor.selectedRange; // 获取当前选中范围
  const selectedText = $editor.selectedText; // 获取选中文本

  if (selectedText.length === 0) {
    $ui.toast("请先选择一段代码！");
    return;
  }

  // 将选中文本按行分割
  const lines = selectedText.split('\n');
  // 为每一行添加注释前缀
  const commentedLines = lines.map(line => `// ${line}`);
  // 将处理后的行重新合并成字符串
  const newText = commentedLines.join('\n');

  // 将处理后的文本替换回编辑器中的选中区域
  $editor.setTextInRange(newText, range);

  $ui.toast("代码已注释！");
}

// 运行这个函数
commentSelectedCode();
```

**代码解读**：

1.  脚本首先通过 `$editor.selectedRange` 和 `$editor.selectedText` 获取用户在编辑器中选中的内容及其位置。
2.  它将选中的文本按行分割，然后使用 `map` 函数为每一行添加 `// ` 前缀。
3.  最后，使用 `$editor.setTextInRange(newText, range)` 将处理后的文本替换回编辑器中原来的选中区域。这个方法非常精确，它会根据你提供的 `range` 来替换内容，而不是简单地在光标处插入。

`$editor` API 为 JSBox 脚本的自动化和定制化打开了新的大门，让你可以根据自己的需求，打造更高效的开发体验。 
