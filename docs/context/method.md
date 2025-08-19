# $context.query

获取通过 URL Scheme 启动扩展时的所有参数：

```js
const query = $context.query;
```

用 `jsbox://runjs?file=demo.js&text=test` 启动时，query 值为：

```js
{
  "file": "demo.js",
  "text": "test"
}
```

若从其他第三方应用跳转至 JSBox，query 内可能包含来源应用的 Bundle ID:

```js
const sourceApp = $context.query.source_app;
```

您可以根据这个信息进行一些判断，但对 iOS 自带的应用无效。

# $context.text

返回一个文本（当用户选择文本进行分享时）：

```js
const text = $context.text;
```

# $context.textItems

返回所有文本。

# $context.link

返回一个链接（当用户选择链接进行分享时）：

```js
const link = $context.link;
```

# $context.linkItems

返回所有链接。

# $context.image

返回一个图片（当用户选择图片进行分享时）：

```js
const image = $context.image;
```

# $context.imageItems

返回所有图片。

# $context.safari.items

当用户在 Safari 上启动扩展时，通过此方法拿到浏览器中的对象：

```js
const items = $context.safari.items;
```

对象结构：

```json
{
  "baseURI": "",
  "source": "",
  "location": "",
  "contentType": "",
  "title": "",
  "selection": {
    "html": "",
    "text": "",
    "style": ""
  }
}
```

# $context.data

返回一个二进制文件（当用户选择文件进行分享时）：

```js
const data = $context.data;
```

# $context.dataItems

返回所有二进制文件。

# $context.allItems

一次返回上述所有的内容在一个 JSON 对象里。

以上返回的对象和在其他地方使用的方法完全相同，也就是说 `$context` 本质上是提供了一种获取输入数据的方式。

就目前而言，`$context` 仅对从 Action Extension 运行这种场景有效。

# $context.clear()

清除 context 内的所有数据，包括传递进来的参数和 Action Extension 的数据：

```js
$context.clear();
```

# $context.close()

在 Action Extension 中运行时，通过这个方法可以关闭 Extension，请注意如果先调用了 `$app.close()`，这条语句将不会生效，因为 $app.close() 之后的代码都不会生效。

---

## 文件内容解读与示例

### 用途说明

本文档是全局变量 `$context` 的 API 参考手册。`$context` 对象是你的脚本与外部世界沟通的“收件箱”，它包含了所有从外部（主要是 iOS 分享菜单，即 Action Extension）传递给脚本的数据。

### 核心属性详解

编写一个健壮的 Action Extension 脚本的关键，就是依次检查 `$context` 的各个属性，判断当前收到了哪种类型的数据。

- **单数 vs. 复数 (`.text` vs. `.textItems`)**: 
  - 单数属性（如 `$context.text`, `$context.image`）是为了方便起见，当你确定用户只会分享**一个**项目时，可以直接使用它来获取该项目。
  - 复数属性（如 `$context.textItems`, `$context.imageItems`）则会返回一个**数组**，即使用户只分享了一个项目。当你需要处理用户一次性分享**多个**项目的场景时，必须使用复数属性。

- **`$context.query`**: 用于通过 URL Scheme 启动脚本时传递参数。例如，`jsbox://runjs?name=MyScript&x=100`，在 `MyScript.js` 中可以通过 `$context.query.x` 获取到值 `100`。

- **`$context.text` / `.link` / `.image` / `.data`**: 分别用于接收分享过来的纯文本、URL 链接、图片和通用二进制文件（如 PDF、ZIP等）。它们是你最常打交道的属性。

- **`$context.safari.items`**: **（特别强大）** 这是一个**仅在 Safari 中运行分享扩展时**才有的特殊属性。它不仅包含当前页面的标题和链接，甚至还能获取到用户在页面上**选中的文字**（`selection.text`）及其 HTML（`selection.html`）。这对于制作网页剪藏、翻译等工具极其有用。

- **`$context.close()` 方法**: 在你的 Action Extension 脚本完成任务后，调用此方法可以**以编程方式关闭分享窗口**，将用户带回之前的 App。这能提供非常流畅的用户体验，否则用户需要手动点击“完成”或“取消”。

### 示例代码：一个万能的“剪藏”脚本

下面的脚本演示了如何通过检查 `$context` 的不同属性，来智能地处理各种类型的分享内容。

```javascript
// 优先检查最具体的数据类型，例如图片
if ($context.image) {
  // 如果分享的是图片
  handleImage($context.image);
} else if ($context.link) {
  // 如果分享的是链接
  handleLink($context.link);
} else if ($context.text) {
  // 如果分享的是纯文本
  handleText($context.text);
} else if ($context.data) {
  // 如果是其他文件
  handleData($context.data);
} else {
  // 如果直接在主应用中运行，没有任何输入
  $ui.alert("请从其他 App 中通过分享菜单来使用本脚本。");
}

function handleImage(image) {
  $ui.toast("图片已收到，正在预览...");
  $quicklook.open({ image });
  // 实际应用中，这里可能是保存到相册或上传到图床
  $context.close();
}

function handleLink(link) {
  $clipboard.text = link;
  $ui.toast(`链接已复制: ${link}`);
  $context.close();
}

function handleText(text) {
  $clipboard.text = text;
  $ui.toast(`文字已复制`);
  $context.close();
}

function handleData(data) {
  const fileName = data.fileName || "file.dat";
  $ui.alert(`收到了文件: ${fileName}\n大小: ${data.info.size} 字节`);
  // 实际应用中，这里可能是保存到本地或上传到云盘
  $context.close();
}
```

**代码解读**：

这个脚本的逻辑非常清晰，它采用了一系列 `if...else if...` 判断：

1.  首先检查 `$context.image` 是否有值。
2.  如果没有，再检查 `$context.link` 是否有值。
3.  以此类推，直到找到一个有值的属性，就调用对应的处理函数（`handleImage`, `handleLink` 等）。
4.  在每个处理函数的末尾，我们都调用了 `$context.close()`，这样脚本在处理完分享内容后就会自动退出，体验非常顺滑。
5.  如果所有 `$context` 属性都为空，说明脚本是在主应用内直接运行的，此时我们给出一个提示，引导用户正确使用。

通过这种模式，你可以编写出功能强大且用途广泛的 Action Extension 工具。
