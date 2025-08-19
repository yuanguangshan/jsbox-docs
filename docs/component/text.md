# type: "text"

`text` 用于创建一个可多行编辑的文本框，目前不支持富文本（但支持 HTML)：

```js
{
  type: "text",
  props: {
    text: "Hello, World!\n\nThis is a demo for Text View in JSBox extension!\n\nCurrently we don't support attributed string in iOS.\n\nYou can try html! Looks pretty cool."
  },
  layout: $layout.fill
}
```

将会在画布上显示一大段话，并且可以编辑。

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
type | $kbType | 读写 | 键盘类型
darkKeyboard | boolean | 读写 | 是否黑色键盘
text | string | 读写 | 文本内容
styledText | object | 读写 | 带格式的文本
html | string | 只写 | 通过 html 渲染富文本
font | $font | 读写 | 字体
textColor | $color | 读写 | 文本颜色
align | $align | 读写 | 对齐方式
placeholder | string | 读写 | placeholder
selectedRange | $range | 读写 | 选中区域
editable | boolean | 读写 | 是否可编辑
selectable | boolean | 读写 | 是否可选择
insets | $insets | 读写 | 边距

# focus()

获取焦点，弹出键盘。

# blur()

模糊焦点，收起键盘。

# events: didBeginEditing

`didBeginEditing` 在开始编辑后回调：

```js
didBeginEditing: function(sender) {

}
```

# events: didEndEditing

`didEndEditing` 在结束编辑后回调：

```js
didEndEditing: function(sender) {
  
}
```

# events: didChange

`didChange` 在内容变化时回调：

```js
didChange: function(sender) {

}
```

# events: didChangeSelection

`didChangeSelection` 在选择区域变化时回调：

```js
didChangeSelection: function(sender) {

}
```

同时，`text` 继承自 `scroll`，所以 scroll 支持的全部事件和属性 text 也一样支持。

# styledText

用于设置带格式的文本，使用 Markdown 语法，支持粗体、斜体和链接，支持格式的嵌套：

```js
const text = `**Bold** *Italic* or __Bold__ _Italic_

[Inline Link](https://docs.xteko.com) <https://docs.xteko.com>

_Nested **styles**`

$ui.render({
  views: [
    {
      type: "text",
      props: {
        styledText: text
      },
      layout: $layout.fill
    }
  ]
});
```

这将使用默认的字体和颜色进行渲染，也可以进行自定义：

```js
$ui.render({
  views: [
    {
      type: "text",
      props: {
        styledText: {
          text: "",
          font: $font(15),
          color: $color("black")
        }
      },
      layout: $layout.fill
    }
  ]
});
```

若需表示 `*`, `_` 等特殊符号，请使用 `\\` 进行转义。

如果需要对格式进行更精细的控制，可以通过 `styles` 为文字的不同位置指定样式：

```js
const text = `
AmericanTypewriter Cochin-Italic

Text Color Background Color

Kern

Strikethrough Underline

Stroke

Link

Baseline Offset

Obliqueness
`;

const _range = keyword => {
  return $range(text.indexOf(keyword), keyword.length);
}

$ui.render({
  views: [
    {
      type: "text",
      props: {
        styledText: {
          text,
          font: $font(17),
          color: $color("black"),
          markdown: false, // whether to use markdown syntax
          styles: [
            {
              range: _range("AmericanTypewriter"),
              font: $font("AmericanTypewriter", 17)
            },
            {
              range: _range("Cochin-Italic"),
              font: $font("Cochin-Italic", 17)
            },
            {
              range: _range("Text Color"),
              color: $color("red")
            },
            {
              range: _range("Background Color"),
              color: $color("white"),
              bgcolor: $color("blue")
            },
            {
              range: _range("Kern"),
              kern: 10
            },
            {
              range: _range("Strikethrough"),
              strikethroughStyle: 2,
              strikethroughColor: $color("red")
            },
            {
              range: _range("Underline"),
              underlineStyle: 9,
              underlineColor: $color("green")
            },
            {
              range: _range("Stroke"),
              strokeWidth: 3,
              strokeColor: $color("black")
            },
            {
              range: _range("Link"),
              link: "https://xteko.com"
            },
            {
              range: _range("Baseline Offset"),
              baselineOffset: 10
            },
            {
              range: _range("Obliqueness"),
              obliqueness: 1
            }
          ]
        }
      },
      layout: $layout.fill
    }
  ]
});
```

属性 | 类型 | 说明
---|---|---
range | $range | 文字范围
font | $font | 字体
color | $color | 前景色
bgcolor | $color | 背景色
kern | number | 字距
strikethroughStyle | number | 删除线样式 [Refer](https://developer.apple.com/documentation/uikit/nsunderlinestyle?language=objc)
strikethroughColor | $color | 删除线颜色
underlineStyle | number | 下划线样式 [Refer](https://developer.apple.com/documentation/uikit/nsunderlinestyle?language=objc)
underlineColor | $color | 下划线颜色
strokeWidth | number | 描边宽度
strokeColor | $color | 描边颜色
link | string | 链接 URL
baselineOffset | number | 基线偏移
obliqueness | number | 字体倾斜

使用 `styles` 默认不使用 markdown 语法，也可以通过 `markdown: true` 开启。

关于下划线和删除线，请参考 Apple 提供的文档，在此做一个简单的举例：

```js
NSUnderlineStyleNone = 0x00,
NSUnderlineStyleSingle = 0x01,
NSUnderlineStyleThick = 0x02,
NSUnderlineStyleDouble = 0x09,
NSUnderlineStylePatternSolid = 0x0000,
NSUnderlineStylePatternDot = 0x0100,
NSUnderlineStylePatternDash = 0x0200,
NSUnderlineStylePatternDashDot = 0x0300,
NSUnderlineStylePatternDashDotDot = 0x0400,
NSUnderlineStyleByWord = 0x8000,
```

如果想实现细的点状下划线，请使用 `NSUnderlineStyleSingle` 和 `NSUnderlineStylePatternDot` 的组合，也既：

```js
underlineStyle: 0x01 | 0x0100
```

# 自定义键盘工具栏

通过这样的方式自定义键盘工具栏：

```js
$ui.render({
  views: [
    {
      type: "input",
      props: {
        accessoryView: {
          type: "view",
          props: {
            height: 44
          },
          views: [

          ]
        }
      }
    }
  ]
});
```

# 自定义键盘视图

通过这样的方式自定义键盘视图：

```js
$ui.render({
  views: [
    {
      type: "input",
      props: {
        keyboardView: {
          type: "view",
          props: {
            height: 267
          },
          views: [

          ]
        }
      }
    }
  ]
});
```
```

--- 

## 文件内容解读与示例

### 组件用途

`text` 组件是一个功能强大的**多行文本视图**。与只能输入单行文本的 `input` 组件不同，`text` 专为处理大段文本而设计。它的核心用途有两个：

1.  **多行文本编辑器**：作为输入框时（`editable: true`），它是构建笔记应用、代码编辑器或任何长文本输入界面的基础。
2.  **富文本展示器**：作为展示器时（`editable: false`），它能渲染包含多种样式的“富文本”（Attributed String），效果远超 `label` 组件。

此外，`text` 组件继承自 `scroll`，因此它天生支持滚动，能容纳超出其可视范围的大量文本。

### 核心概念：`text` vs. `styledText`

- **`text` 属性**: 用于处理**纯文本**。当你只需要一个简单的、没有特殊格式的多行输入框时，使用此属性。

- **`styledText` 属性**: 用于处理**富文本**，这是 `text` 组件最强大的特性。它有两种使用方式：
  1.  **简单 Markdown 模式**: 直接给 `styledText` 赋一个包含 Markdown 语法的字符串，`text` 组件会自动渲染出粗体、斜体和链接等基本样式。
  2.  **精细样式数组模式**: 给 `styledText` 赋一个包含 `text` 和 `styles` 两个键的对象。`styles` 是一个样式规则数组，每个规则对象都通过 `range`（文字范围）来为文本的特定部分精确地指定 `font`、`color`、`underlineStyle` 等数十种样式。这是实现复杂富文本编辑和展示的关键。

### 示例代码：一个迷你富文本编辑器

下面的示例将创建一个带工具栏的文本编辑器，用户可以选择文本，然后点击按钮将其变为粗体或斜体，以此来演示如何动态地读写 `styledText`。

```javascript
$ui.render({
  props: { title: "Text 组件示例" },
  views: [
    {
      // 工具栏
      type: "stack",
      props: {
        axis: $stackViewAxis.horizontal,
        distribution: $stackViewDistribution.fillEqually,
        spacing: 10
      },
      layout: make => {
        make.top.left.right.inset(10);
        make.height.equalTo(32);
      },
      views: [
        { type: "button", title: "加粗", events: { tapped: () => applyStyle("bold") } },
        { type: "button", title: "斜体", events: { tapped: () => applyStyle("italic") } },
        { type: "button", title: "下划线", events: { tapped: () => applyStyle("underline") } }
      ]
    },
    {
      // 文本编辑区
      type: "text",
      props: {
        id: "editor",
        text: "在这里选择一段文字，然后点击上方按钮应用样式。",
        font: $font(18)
      },
      layout: make => {
        make.top.equalTo(view.prev.bottom).offset(10);
        make.left.right.bottom.inset(10);
      }
    }
  ]
});

function applyStyle(type) {
  const editor = $("editor");
  const { location, length } = editor.selectedRange;

  if (length === 0) {
    $ui.toast("请先选择一段文字");
    return;
  }

  // 获取当前的 styledText 对象，如果不存在则基于当前 text 创建
  let styled = editor.styledText || { text: editor.text, styles: [] };

  // 创建新的样式规则
  let newStyle = { range: $range(location, length) };
  const baseFont = editor.font || $font(18);

  if (type === "bold") {
    newStyle.font = $font("bold", baseFont.pointSize);
  } else if (type === "italic") {
    newStyle.font = $font("italic", baseFont.pointSize);
  } else if (type === "underline") {
    newStyle.underlineStyle = 1; // NSUnderlineStyleSingle
  }

  // 添加新样式并重新应用
  styled.styles.push(newStyle);
  editor.styledText = styled;
  
  // 失去焦点以查看效果
  editor.blur();
}
```

**代码解读**：

1.  我们创建了一个 `text` 视图作为编辑器，以及一个包含三个按钮的工具栏。
2.  核心逻辑在 `applyStyle` 函数中。
3.  `editor.selectedRange` 获取用户当前选中的文本范围（起始位置和长度）。
4.  `editor.styledText || { text: editor.text, styles: [] }` 这是一个关键技巧，用于获取当前的富文本对象，如果之前是纯文本，则基于当前文本创建一个新的富文本对象。
5.  我们根据按钮类型，创建一个新的样式规则对象 `newStyle`，并指定其作用的 `range`。
6.  最后，将新规则 `push` 到 `styles` 数组中，再把整个 `styled` 对象赋回给 `editor.styledText`，视图便会立即更新以反映新的样式。

这个例子展示了 `text` 组件作为富文本编辑器的强大能力，其核心在于对 `styledText` 对象的编程化读写. 