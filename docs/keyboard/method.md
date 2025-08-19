# 键盘上运行的脚本

从 v1.12.0 开始，JSBox 支持让用户通过脚本直接编写一个键盘扩展，你可以通过 JavaScript 来实现对输入区域的操作，可以用于制作辅助输入的插件。

你完全无须关心这个机制是如何实现的，你只需通过 `$keyboard` 相关接口准备好你的脚本。

这里有两个完整的例子用于演示如何构建一个键盘插件：

https://github.com/cyanzhong/xTeko/tree/master/extension-demos/keyboard

[点击这里安装体验](https://xteko.com/redir?url=https://github.com/cyanzhong/xTeko/raw/master/extension-demos/keyboard.box)

https://github.com/cyanzhong/xTeko/tree/master/extension-demos/emoji-key

[点击这里安装体验](https://xteko.com/redir?url=https://github.com/cyanzhong/xTeko/raw/master/extension-demos/emoji-key.box)

# $keyboard.insert(string)

向当前的输入框插入一段文字：

```js
$keyboard.insert("Hey!")
```

# $keyboard.delete()

删除当前选中的内容或者向后删除一个字符：

```js
$keyboard.delete()
```

# $keyboard.moveCursor(number)

移动当前光标（向后移动 5 个位置）：

```js
$keyboard.moveCursor(5)
```

# $keyboard.playInputClick()

播放键盘按下去的音效：

```js
$keyboard.playInputClick()
```

# $keyboard.hasText

判断当前输入区域是否有文字：

```js
const hasText = $keyboard.hasText;
```

# $keyboard.selectedText

获取当前选中的文字（iOS 11 以上）：

```js
const selectedText = $keyboard.selectedText;
```

# $keyboard.textBeforeInput

输入前的文字：

```js
const textBeforeInput = $keyboard.textBeforeInput;
```

# $keyboard.textAfterInput

输入后的文字：

```js
const textAfterInput = $keyboard.textAfterInput;
```

# $keyboard.getAllText(handler)

获取当前的全部文字（iOS 11 以上）：

```js
$keyboard.getAllText(text => {
  
})
```

# $keyboard.next()

切换到下个输入法：

```js
$keyboard.next()
```

# $keyboard.send()

模拟键盘上的发送事件：

```js
$keyboard.send()
```

# $keyboard.dismiss()

将键盘收起来：

```js
$keyboard.dismiss()
```

# $keyboard.barHidden

是否隐藏底部的工具栏，这是一个属性，应该尽早设置。

以上所有的接口，都只能在键盘上面运行，但用户尝试将其运行在别的环境时，会得到运行时错误。

# $keyboard.height

获取和设置键盘的高度：

```js
let height = $keyboard.height;

$keyboard.height = 500;
```

---

## 文件内容解读与示例

### 用途说明

`$keyboard` API 是 JSBox 中一个非常特殊的模块，它专门用于**创建自定义键盘扩展**。通过这个 API，你的脚本可以完全控制 iOS 系统的键盘行为，实现各种辅助输入功能，例如：

-   快速插入预设短语或常用信息
-   制作表情符号键盘
-   对输入文本进行实时转换或格式化
-   实现自定义的计算器键盘

### 核心概念：键盘扩展的运行环境

-   **专属环境**: `$keyboard` API 只能在脚本作为**键盘扩展**运行时才能生效。如果你尝试在主应用或其他环境中调用这些接口，将会导致运行时错误。
-   **输入区域操作**: 键盘扩展的核心能力是读取和修改当前光标所在的文本输入区域的内容。这使得你的脚本能够直接影响用户正在输入的文本。

### 主要功能与方法详解

#### 1. 文本插入与删除

-   **`$keyboard.insert(string)`**: 在当前光标位置插入指定的文本。如果当前有选中的文本，则会替换选中内容。
-   **`$keyboard.delete()`**: 删除当前选中的内容。如果没有选中，则删除光标前的一个字符（模拟退格键）。

**示例**：自定义键盘按钮，插入常用短语

```javascript
// 假设这是你的自定义键盘 UI 中的一个按钮的 tapped 事件
// { type: "button", props: { title: "你好" }, events: { tapped: () => $keyboard.insert("你好，世界！") } }
```

#### 2. 光标与文本内容访问

-   **`$keyboard.moveCursor(offset)`**: 移动当前光标的位置。`offset` 为正数表示向后移动，负数表示向前移动。
-   **`$keyboard.selectedText`**: 获取当前输入区域中被选中的文本内容（iOS 11+）。
-   **`$keyboard.textBeforeInput` / `$keyboard.textAfterInput`**: 分别获取光标前和光标后的文本内容。
-   **`$keyboard.getAllText(handler)`**: 异步获取当前输入区域的全部文本内容（iOS 11+）。

**示例**：将选中文字用引号包裹

```javascript
// 假设这是你的自定义键盘 UI 中的一个按钮的 tapped 事件
// { type: "button", props: { title: "加引号" }, events: { tapped: () => {
//   const selected = $keyboard.selectedText;
//   if (selected) {
//     $keyboard.delete(); // 先删除选中内容
//     $keyboard.insert(`"${selected}"`); // 再插入带引号的内容
//   }
// } } }
```

#### 3. 键盘行为控制

-   **`$keyboard.next()`**: 模拟键盘上的“地球仪”键，切换到下一个输入法。
-   **`$keyboard.send()`**: 模拟键盘上的“发送”或“回车”键事件。
-   **`$keyboard.dismiss()`**: 收起键盘。
-   **`$keyboard.playInputClick()`**: 播放键盘按键音效。
-   **`$keyboard.height`**: 获取或设置自定义键盘的高度。
-   **`$keyboard.barHidden`**: 控制键盘底部工具栏的显示/隐藏。

**示例**：自定义键盘上的“发送”和“收起”按钮

```javascript
// { type: "button", props: { title: "发送" }, events: { tapped: () => $keyboard.send() } }
// { type: "button", props: { title: "收起" }, events: { tapped: () => $keyboard.dismiss() } }
```

### 示例代码：一个简单的自定义键盘

下面的示例将创建一个非常简单的自定义键盘，包含一个插入当前时间戳的按钮和一个切换到下一个键盘的按钮。

```javascript
// 键盘扩展的 UI 布局
$ui.render({
  views: [
    {
      type: "view",
      props: {
        bgcolor: $color("#EFEFF4"), // 键盘背景色
        height: 250 // 键盘高度
      },
      layout: $layout.fill,
      views: [
        {
          type: "stack",
          props: {
            axis: $stackViewAxis.horizontal,
            distribution: $stackViewDistribution.fillEqually,
            spacing: 10
          },
          layout: make => {
            make.top.left.right.inset(10);
            make.height.equalTo(50);
          },
          views: [
            {
              type: "button",
              props: { title: "插入时间戳" },
              events: {
                tapped: () => {
                  $keyboard.insert(new Date().toLocaleString());
                  $keyboard.playInputClick(); // 播放按键音
                }
              }
            },
            {
              type: "button",
              props: { title: "下一个键盘" },
              events: {
                tapped: () => {
                  $keyboard.next();
                  $keyboard.playInputClick();
                }
              }
            }
          ]
        },
        {
          type: "label",
          props: {
            text: "(请在系统设置中启用此键盘)",
            font: $font(12),
            textColor: $color("gray"),
            align: $align.center
          },
          layout: make => {
            make.top.equalTo(view.prev.bottom).offset(20);
            make.centerX.equalTo(view.super);
          }
        }
      ]
    }
  ]
});
```

**代码解读**：

1.  这个脚本定义了一个简单的键盘 UI，包含两个按钮。
2.  “插入时间戳”按钮的 `tapped` 事件调用 `$keyboard.insert()` 将当前时间插入到输入框中。
3.  “下一个键盘”按钮的 `tapped` 事件调用 `$keyboard.next()` 来切换到下一个输入法。
4.  两个按钮都调用了 `$keyboard.playInputClick()` 来模拟按键音效，提升用户体验。

`$keyboard` API 为 JSBox 脚本提供了强大的自定义输入能力，让你能够创建高度个性化和自动化的键盘扩展。 
