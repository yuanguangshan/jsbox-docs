# type: "code"

用于对代码进行高亮，以及简单的编辑，支持常见的几十种编程语言以及主题：

```js
$ui.render({
  views: [
    {
      type: "code",
      props: {
        text: "const value = 100"
      },
      layout: $layout.fill
    }
  ]
});
```

可以通过下面这些参数来配置代码编辑视图：

属性 | 类型 | 默认值 | 说明
---|---|---|---
language | string | javascript | 编程语言类型
theme | string | nord | 编辑器主题
darkKeyboard | bool | true | 是否使用黑色键盘
adjustInsets | bool | true | 是否根据键盘调整 insets
lineNumbers | bool | false | 是否显示行号
invisibles | bool | false | 是否显示不可见字符
linePadding | number | null | 自定义行高
keys | [string] | null | 自定义键盘工具栏

请注意，这些参数必须在初始化视图时确定，不能通过动态设置的方式覆盖。

与此同时，此组件继承自 Text View，所以 [`type: text`](component/text.md) 支持的功能它都支持。

例如，你可以通过 `editable: false` 来将视图设置为只读，不可编辑。可以通过 `text` 属性来读写代码内容：

```js
const view = $("codeView");
const code = view.text;
view.text = "console.log([1, 2, 3])";
```

当然，也支持全部 text 组件支持的事件：

```js
$ui.render({
  views: [
    {
      type: "code",
      props: {
        text: "const value = 100"
      },
      layout: $layout.fill,
      events: {
        changed: sender => {
          console.log("code changed");
        }
      }
    }
  ]
});
```

# props: language

`code` 组件的实现基于开源项目 [highlightjs](https://highlightjs.org/)，支持的编程语言名字列表可以在这里找到：https://github.com/highlightjs/highlight.js/tree/master/src/languages

例如需要对 Python 进行高亮：

```js
props: {
  language: "python"
}
```

# props: theme

`code` 组件的实现基于开源项目 [highlightjs](https://highlightjs.org/)，支持的主题列表可以在这里找到：https://github.com/highlightjs/highlight.js/tree/master/src/styles

例如需要使用 `atom-one-light` 主题：

```js
props: {
  theme: "atom-one-light"
}
```

# props: adjustInsets

在很多时候，我们需要处理键盘遮挡输入区域的问题，比如全屏的编辑页面。`code` 组件自动为你处理了这件事情，它会根据键盘的高度自动调整自己的 insets，以避免输入区域被遮挡。

自动实现的逻辑很难做到完美，当你不需要这个功能，可以通过 `adjustInsets: false` 关闭。

# props: keys

代码视图默认提供一个键盘工具栏，如果需要根据不同的编程语言自定义，可以这样：

```js
$ui.render({
  views: [
    {
      type: "code",
      props: {
        language: "markdown",
        keys: [
          "#",
          "-",
          "*",
          "`",
          //...
        ]
      },
      layout: $layout.fill
    }
  ]
});
```

如果想要完全去除自定义工具栏，可以通过覆盖 text view 的 `accessoryView` 实现。

---

## 文件内容解读与示例

### 组件用途

`code` 组件是一个为查看和编辑源代码而生的、高度特化的文本视图。它的核心能力是**语法高亮**，能够极大地提升代码的可读性。无论你是想在脚本中展示一段示例代码，还是想构建一个迷你代码编辑器，`code` 组件都是你的不二之选。

### 核心概念：基于 highlight.js

与 `chart` 组件类似，`code` 组件也是构建在一个强大的开源项目之上的，这个项目就是 **[highlight.js](https://highlightjs.org/)**。你需要了解：

- **语言 (`language`) 和主题 (`theme`)**: 支持哪些编程语言和高亮主题，完全由 `highlight.js` 决定。你需要到文档中提供的链接去查找它们的确切名称（例如 `python`, `javascript`, `atom-one-dark`）。
- **继承自 `text`**: `code` 组件拥有标准 `text` 组件的所有功能。这意味着你可以用 `text` 属性来读写内容，用 `editable` 属性来控制是否可编辑，以及监听 `changed` 等事件。

### 关键特性

- **只读模式 vs. 编辑模式**: 通过设置 `props: { editable: false }`，你可以轻松地将组件变为一个只读的代码展示框。这是展示示例代码时的常用做法。
- **编辑器功能增强**: 
  - `lineNumbers: true`: 显示行号，方便定位和讨论代码。
  - `keys: ["{", "}", ":"]`: 自定义键盘上方的快捷输入栏，可以根据不同语言设置不同的常用符号，提升编辑效率。
  - `adjustInsets: true`: 自动处理键盘遮挡问题。当键盘弹出时，组件会自动调整内边距，确保光标所在行始终可见，这是一个非常贴心的功能。

### 示例代码：代码查看器与编辑器

下面的示例将创建一个包含两个 `code` 视图的界面：一个用于只读地展示 Python 代码，另一个作为可交互的 JavaScript 编辑器。

```javascript
const pythonCode = `
import requests

def fetch_data(url):
    """Fetches data from a URL."""
    try:
        response = requests.get(url)
        response.raise_for_status()  # Raises an HTTPError for bad responses
        return response.json()
    except requests.exceptions.RequestException as e:
        print(f"Error fetching data: {e}")
        return None
`;

$ui.render({
  props: {
    title: "Code 组件示例"
  },
  views: [
    {
      // 1. 一个只读的 Python 代码查看器
      type: "code",
      props: {
        text: pythonCode,
        language: "python", // 指定语言为 python
        theme: "atom-one-dark", // 使用 atom-one-dark 主题
        lineNumbers: true, // 显示行号
        editable: false, // 设置为不可编辑
        font: $font("Menlo", 14)
      },
      layout: function(make, view) {
        make.top.left.right.inset(10);
        make.height.equalTo(200);
      }
    },
    {
      // 2. 一个可编辑的 JavaScript 编辑器
      type: "code",
      props: {
        id: "js-editor",
        text: "// 在这里输入 JavaScript 代码\nfunction greet() {\n  console.log(\"Hello, JSBox!\");\n}",
        language: "javascript",
        theme: "atom-one-light", // 使用 atom-one-light 主题
        darkKeyboard: false,
        keys: ["{", "}", "[", "]", "(", ")", ";", ".", "=", "=>"]
      },
      layout: function(make, view) {
        make.top.equalTo(view.prev.bottom).offset(10);
        make.left.right.bottom.inset(10);
      },
      events: {
        changed: function(sender) {
          // 每次内容改变时，在控制台输出当前代码
          console.log(sender.text);
        }
      }
    }
  ]
});
```

**代码解读**：

1.  **Python 查看器**：我们设置了 `language: "python"` 和深色主题 `theme: "atom-one-dark"`。最关键的是 `editable: false`，它使得这个视图变成了一个纯展示的窗口。`lineNumbers: true` 增加了可读性。
2.  **JavaScript 编辑器**：我们使用了浅色主题，并保留了默认的 `editable: true`。通过 `keys` 属性，我们为键盘上方定制了一排 JavaScript 中常用的符号。`changed` 事件则让我们能实时获取并处理用户输入的代码。

`code` 组件是处理代码相关任务的完美工具。请务必参考 `highlight.js` 的文档来发掘所有支持的语言和主题，以满足你的定制化需求。 
