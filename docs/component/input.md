# type: "input"

`input` 可以创建一个输入框，用于处理单行的文本输入：

```js
{
  type: "input",
  props: {
    type: $kbType.search,
    darkKeyboard: true,
  },
  layout: function(make, view) {
    make.center.equalTo(view.super)
    make.size.equalTo($size(100, 32))
  }
}
```

在画布中显示一个搜索键盘，同时是黑色的主题。

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
type | $kbType | 读写 | 类型
darkKeyboard | boolean | 读写 | 黑色主题
text | string | 读写 | 文本内容
styledText | object | 读写 | 带格式的文本，[参考](component/text.md?id=styledtext)
textColor | $color | 读写 | 文本颜色
font | $font | 读写 | 字体
align | $align | 读写 | 对齐方式
placeholder | string | 读写 | placeholder
clearsOnBeginEditing | boolean | 读写 | 开始时清除文本
autoFontSize | boolean | 读写 | 是否动态调整字体大小
editing | boolean | 只读 | 是否在编辑
secure | boolean | 读写 | 是否密码框

# focus()

获取焦点，弹出键盘。

# blur()

模糊焦点，收起键盘。

# events: changed

`changed` 事件可以在文本变化时回调：

```js
changed: function(sender) {

}
```

# events: returned

`returned` 事件在回车键按下时调用：

```js
returned: function(sender) {

}
```

# events: didBeginEditing

`didBeginEditing` 事件在开始编辑时调用：

```js
didBeginEditing: function(sender) {

}
```

# events: didEndEditing

`didEndEditing` 事件在结束编辑时调用：

```js
didEndEditing: function(sender) {
  
}
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

# $input.text(object)

文字输入也可以使用这个快捷方法，将会直接弹出键盘以供输入：

```js
$input.text({
  type: $kbType.number,
  placeholder: "Input a number",
  handler: function(text) {

  }
})
```

# $input.speech(object)

使用系统听写接口输入文字（语音转文字）：

```js
$input.speech({
  locale: "en-US", // 可选
  autoFinish: false, // 可选
  handler: function(text) {

  }
})
```

---

## 文件内容解读与示例

### 组件用途

`input` 组件是用于从用户那里获取**单行文本**输入的基础控件。它是构建任何表单（如登录、注册）、搜索框或设置项的核心元素，相当于 HTML 中的 `<input type="text">`。

### 核心属性与功能

- **`type` (键盘类型)**: 这是提升移动端体验的关键属性。通过设置 `type` 为 `$kbType` 枚举中的不同值（如 `$kbType.email`, `$kbType.number`, `$kbType.url`），你可以为用户弹出最适合当前输入内容的键盘，例如带 `@` 符号的邮箱键盘或纯数字键盘。

- **`placeholder`**: 占位符文本。它以灰色文字显示在输入框中，用于提示用户应该输入什么样的内容，当用户开始输入时会自动消失。

- **`secure` (安全输入)**: 一个非常实用的布尔值。设置为 `true` 后，输入框会变为密码模式，用户输入的内容会显示为圆点，用于保护敏感信息。

- **`focus()` 和 `blur()` 方法**: 
  - `focus()`: 命令输入框获取焦点，主动弹起键盘让用户开始输入。常用于页面加载后自动激活某个输入框。
  - `blur()`: 命令输入框失去焦点，收起键盘。

### 核心事件

- **`changed`**: 当输入框中的文本**每次发生变化**（增加或减少一个字符）时，这个事件都会被触发。它适用于需要实时响应的场景，如显示剩余字数、实时搜索建议等。

- **`returned`**: 当用户点击键盘上的**“返回”键**（在不同键盘类型下可能显示为 “Go”, “Search”, “Done” 等）时，这个事件被触发。这是处理表单提交最常用的事件。

### `input` 组件 vs. `$input.text()` 全局方法

需要特别区分这两者：

- **`{ type: "input" }`**: 这是一个 **UI 组件**，是你自己设计的界面的一部分。你可以完全控制它的位置、大小和样式，并将它与其他组件（如 `label`, `button`）组合成复杂的布局。

- **`$input.text()`**: 这是一个**全局辅助函数**。它会弹出一个模态对话框，其中包含一个输入框和一个确定按钮。它用于**快速、临时地**获取一段文本，而无需你费心设计整个 UI。当你的脚本只需要用户输入一个简单的值时，这是最佳选择。

### 示例代码：一个简单的登录表单

下面的示例将创建一个包含用户名、密码输入框和登录按钮的表单，并演示如何处理输入和事件。

```javascript
$ui.render({
  props: {
    title: "用户登录"
  },
  views: [
    {
      type: "label",
      props: { text: "用户名:" },
      layout: (make, view) => {
        make.top.left.inset(20);
      }
    },
    {
      type: "input",
      props: {
        id: "username-input",
        placeholder: "请输入邮箱或手机号",
        type: $kbType.email // 弹出邮箱键盘
      },
      layout: (make, view) => {
        make.top.equalTo(view.prev.bottom).offset(10);
        make.left.right.inset(20);
        make.height.equalTo(36);
      }
    },
    {
      type: "label",
      props: { text: "密码:" },
      layout: (make, view) => {
        make.top.equalTo(view.prev.bottom).offset(20);
        make.left.inset(20);
      }
    },
    {
      type: "input",
      props: {
        id: "password-input",
        placeholder: "请输入密码",
        secure: true // 设置为密码模式
      },
      layout: (make, view) => {
        make.top.equalTo(view.prev.bottom).offset(10);
        make.left.right.inset(20);
        make.height.equalTo(36);
      },
      events: {
        returned: function(sender) {
          // 在密码框按回车键，直接触发登录
          login();
        }
      }
    },
    {
      type: "button",
      props: { title: "登录" },
      layout: (make, view) => {
        make.top.equalTo(view.prev.bottom).offset(30);
        make.centerX.equalTo(view.super);
        make.width.equalTo(100);
      },
      events: {
        tapped: login
      }
    }
  ]
});

function login() {
  const username = $("username-input").text;
  const password = $("password-input").text;

  if (!username || !password) {
    $ui.alert("用户名和密码不能为空！");
    return;
  }

  // 收起键盘
  $("username-input").blur();
  $("password-input").blur();

  $ui.alert(`登录成功！\n用户名: ${username}\n密码: ${password}`);
}
```

**代码解读**：

1.  我们为用户名输入框设置了 `type: $kbType.email` 以优化体验。
2.  为密码框设置了 `secure: true` 来隐藏输入内容。
3.  我们在密码框的 `returned` 事件中调用了 `login` 函数，这样用户输完密码后可以直接按回车登录，非常便捷。
4.  `login` 函数通过 ID (`$("username-input")`) 获取两个输入框的 `text` 值，并进行处理。最后调用 `blur()` 方法来收起键盘。 
