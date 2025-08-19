> 通过脚本可以控制 JSBox 的脚本，包括安装新脚本、删除脚本等操作

# $addin.list

获得已安装的全部脚本：

```js
const addins = $addin.list;
```

数据结构：

属性 | 类型 | 读写 | 说明
---|---|---|---
name | string | 读写 | 名称
category | string | 读写 | 分类
url | string | 读写 | url
data | $data | 读写 | 文件
id | string | 读写 | id
version | string | 读写 | 版本号
icon | string | 读写 | 图标
iconImage | image | 只读 | 获得图标
module | bool | 读写 | 是否保存为模块
author | string | 读写 | 作者
website | string | 读写 | 网站

以上字段 `name` 和 `data` 为必须，其余字段可选。

你可以对取出的 `addins` 进行更改，例如重新排序，然后这样保存：

```js
$addin.list = addins;
```

# $addin.categories

获取当前设备所有的脚本分类，返回 string 数组：

```js
const categories = $addin.categories;
```

你可以进行修改，例如重新排序或增加分类，然后这样保存：

```js
$addin.categories = categories;
```

# $addin.current

获取当前正在运行的脚本，结构如上述所示：

```js
const current = $addin.current;
```

# $addin.save(object)

安装一个新的脚本：

```js
$addin.save({
  name: "New Script",
  data: $data({string: "$ui.alert('Hey!')"}),
  handler: function(success) {
    
  }
})
```

JSBox 内将会安装一个叫做 New Script 的脚本，save 的所有字段遵循上述数据结构。

# $addin.delete(name)

通过脚本的名字删除一个脚本：

```js
$addin.delete("New Script")
```

将会删除前一个例子里面创建的脚本。

# $addin.run(name)

通过脚本的名字运行一个脚本：

```js
$addin.run("New Script")
```

将会运行名为 New Script 的脚本，当然前提是你没有删掉它。

从 v1.15.0 开始，你也可以传递而外的参数：

```js
$addin.run({
  name: "New Script",
  query: {
    "a": "b"
  }
})
```

参数将会被传递到 `$context.query` 以便获取到。

# $addin.restart()

重新运行当前的脚本。

# $addin.replay()

重新渲染当前有界面的脚本。

# $addin.compile(script)

将 script 转换成 JSBox 可用的语法：

```js
const result = $addin.compile("$ui.alert('Hey')");

// result => JSBox.ui.alert('Hey')
```

# $addin.eval(script)

与 `eval()` 类似，但先会将 script 转换成 JSBox 语法：

```js
$addin.eval("$ui.alert('Hey')");
```

---

## 文件内容解读

### 文件用途

本文档介绍了 JSBox 中一个非常强大和高级的功能：`$addin` API。这个 API 的核心作用是**让一个脚本能够以编程方式管理其他脚本**。你可以把 JSBox 环境本身看作一个可操作的对象，通过代码来实现对已安装脚本的增、删、改、查、运行等所有操作。这属于“元编程”的范畴，对于构建复杂的、自动化的脚本管理工具非常有价值。

### 重点功能展开说明与实例

下面对该 API 的核心方法进行深入解读，并提供实际应用场景。

#### `$addin.list`：脚本的“数据库”

这个属性不仅可以读取，还可以写入，是整个管理体系的基础。

- **读取**：`const scripts = $addin.list;` 会返回一个包含所有已安装脚本对象的数组。你可以获取到每个脚本的名称、分类、作者等所有信息。
- **写入**：`$addin.list = scripts;` 则会将你修改后的数组保存回去。你可以用它来批量修改脚本属性。

**场景举例：制作一个脚本整理工具**

假设你安装了很多脚本，分类很乱。你可以写一个脚本来自动整理它们。

```javascript
// 脚本整理工具
let scripts = $addin.list;

scripts.forEach(script => {
  if (script.name.includes("剪贴板")) {
    script.category = "剪贴板工具";
  } else if (script.name.includes("图片")) {
    script.category = "图片处理";
  }
});

// 保存更改
$addin.list = scripts;
$ui.alert("脚本已重新分类！");
```

#### `$addin.save(object)`：动态安装脚本

这是 `$addin` API 最强大的功能之一，它允许你的脚本创建并安装一个全新的脚本。

**难点解读**：核心在于 `data` 字段。它必须是一个 `$data` 对象，通常由包含代码的字符串生成，即 `$data({string: "..."})`。

**场景举例：创建自己的“脚本商店”**

你可以写一个脚本，从你的个人网站或某个代码仓库（如 Gist）获取脚本列表。当用户在你的脚本界面上选择安装时，你的代码会执行以下操作：

```javascript
// 从服务器获取的脚本代码
const remoteCode = "$ui.alert('这是从网络安装的脚本！');";

$addin.save({
  name: "网络安装器",
  data: $data({ string: remoteCode }),
  icon: "icon_100.png", // 设置一个图标
  handler: success => {
    if (success) {
      $ui.alert("安装成功！");
    }
  }
});
```

#### `$addin.run(name)`：脚本间的“传动轴”

这个方法让脚本之间可以互相调用，实现功能的模块化和复用。

**难点解读**：如何在一个脚本中向另一个脚本传递数据？答案是使用 `query` 参数。被调用的脚本可以通过 `$context.query` 来获取这些数据。

**场景举例：构建多任务流水线**

假设你有一个通用的“发送通知”脚本，现在你想写一个“天气预警”脚本。

```javascript
// 天气预警脚本
const city = "北京";
const weather = "暴雨"; // 此处应为通过 API 获取的真实天气

// 调用通知脚本，并把天气信息传过去
$addin.run({
  name: "发送通知", // 假设你已经有这个脚本
  query: {
    title: `${city}天气预警`,
    body: `今天将有${weather}，请注意防范。`
  }
});

// 在“发送通知”脚本内部，你可以这样获取数据：
// const info = $context.query;
// const title = info.title;
// const body = info.body;
// $push.schedule(...);
```

#### `$addin.restart()` vs. `$addin.replay()`

这两个方法用于重新加载脚本，但应用场景不同。

- **`$addin.restart()`**: 彻底从头开始重新执行当前脚本。所有变量和状态都会被重置。适用于需要重置逻辑的场景。
- **`$addin.replay()`**: **仅**重新渲染当前脚本的 UI 界面 (`$ui.render`)。脚本的业务逻辑和变量状态会保留。这对于在不中断脚本运行的情况下动态更新界面非常有用，例如在开发 UI 时进行热重载预览。

总而言之，`$addin` API 为开发者提供了一套“元能力”，让你可以超越编写单个工具的局限，转而构建能够管理和调度其他工具的“系统级”脚本。 
