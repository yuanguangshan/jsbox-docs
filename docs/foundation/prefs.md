> 以最简单的配置文件为脚本应用提供配置项

# prefs.json

从 v1.51.0 开始，你只需要在应用根目录配置一个 `prefs.json` 文件，JSBox 就可以自动为你生成配置页面，从而为用户提供更为友好的设置界面。

来看一个例子：

```json
{
  "title": "SETTINGS",
  "groups": [
    {
      "title": "GENERAL",
      "items": [
        {
          "title": "USER_NAME",
          "type": "string",
          "key": "user.name",
          "value": "default user name"
        },
        {
          "title": "AUTO_SAVE",
          "type": "boolean",
          "key": "auto.save",
          "value": true
        },
        {
          "title": "OTHERS",
          "type": "child",
          "value": {
            "title": "OTHERS",
            "groups": [
              
            ]
          }
        }
      ]
    }
  ]
}
```

调用下面的代码即可打开配置界面，以供用户设置：

```js
$prefs.open(() => {
  // Done
});
```

# 配置文件结构

配置文件的根节点是一个 `title` 和 `groups` 的 JSON 对象，`title` 会被设置为页面的标题，`groups` 数组则表示当前的设置页的所有分组。

在 `groups` 下面，是由分组标题 `title` 以及所有选项 `items` 组成的 JSON 对象，`items` 数组里面的每一项就是一个具体的设置项。

如果当前页面只有一个分组，也可以简写成：

```json
{
  "title": "GENERAL",
  "items": [
    {
      "title": "USER_NAME",
      "type": "string",
      "key": "user.name",
      "value": "default user name",
      "inline": true // inline edit, default is false
    }
  ]
}
```

配置项节点由下列内容组成：

- `title`: 标题，显示在左侧
- `type`: 类型，比如字符串或是布尔值，下面会详细介绍
- `key`: 存储以及获取设置用的键，需要保证脚本内全局无冲突
- `value`: 在用户没有设置的时候，提供的缺省值，可以不提供
- `placeholder`: 没有输入时显示的提示
- `insetGrouped`: 是否使用 insetGrouped 样式的列表
- `inline`: 文本框是否行内编辑

# title

为了简化多语言环境下的配置，上述配置内容所有的 title 首先会从 `strings` 文件夹配置的字符串里面寻找本地化的字符串，如果没有，才使用字符串本身。

# type

目前配置文件支持下面的类型：

- `string`: 字符串类型，默认多行
- `password`: 密码类型，输入后显示为圆点
- `number`: 浮点数类型
- `integer`: 整数类型
- `boolean`: 布尔值，在页面中显示为一个开关
- `slider`: 0 ~ 1 的浮点数，在页面中显示为一个滑块
- `list`: 用于在一个列表里面选择一个值
- `info`: 显示一个不可变的信息，比如版本号
- `link`: 显示一个链接，点击后会直接打开
- `script`: 可配置一段脚本，点击后会执行
- `child`: 子设置项，点击之后打开一个新的设置页面

对于 `string`, `number` 以及 `integer` 等较为简单的类型，尝试一下即可了解，下面会介绍比较不同的几个。

# type: "password"

与 `type: "string"` 参数相同，用于输入密码等敏感信息。

# type: "slider"

将会显示一个滑块，用于设置 0 ~ 1 之间的浮点数，所以默认值 `value` 也需要符合 0 ~ 1 的定义。

如果你在程序中需要的值不是 0 ~ 1 的数，你需要做一些简单的变换来映射到你的取值区间，请将这个滑块理解成一个百分比滑块。

# type: "list"

在一个列表里面选择一个值，例如：

```json
{
  "title": "IMAGE_QUALITY",
  "type": "list",
  "key": "image.quality",
  "items": ["LOW", "MEDIUM", "HIGH"],
  "value": 1
}
```

界面会让用户在 "LOW", "MEDIUM" 和 "HIGH" 里面选一个值，而 `value` 则是选择的 `index`。同样的，为了本地化的方便，`items` 里面的内容也会从 `strings` 文件夹里面优先取值。

请注意，`items` 仅仅是显示给用户的内容，并不会被存储，而 `value` 存储的实际上是序号。

# type: "script"

有些时候，你可能希望设置项的某一行点击之后是你自己定义的行为，这种类型就是为了这个目的：

```json
{
  "title": "TEST",
  "type": "script",
  "value": "require('scripts/test').runTest();"
}
```

# type: "child"

有些时候，你可能希望将一些不那么重要的设置项放到二级甚至是三级设置里面，你可以通过这个类型做到：

```json
{
  "title": "OTHERS",
  "type": "child",
  "value": {
    "title": "OTHERS",
    "groups": []
  }
}
```

上述节点的 `value` 正是设置页面根节点的结构，你可以把这个结构无限嵌套下去来实现多级设置。

# key

`key` 是用来保存和读取一个设置项用的键，需要保证在脚本内全局唯一。

读取设置：

```js
const name = $prefs.get("user.name");
```

在多数情况下，设置项都应该是由用户通过界面配置的，但为了保证灵活性，你仍然可以像这样主动更新一个设置项：

```js
$prefs.set("user.name", "cyan");
```

# $prefs.all()

返回所有的键值对：

```js
const prefs = $prefs.all();
```

`$prefs` 显然并不能够应对任何设置项，但对于大部分的需求来说以及完全能够满足，并且使用极为简单。这里有一个完整的例子：https://github.com/cyanzhong/xTeko/tree/master/extension-demos/prefs

# $prefs.edit(node)

除了为默认的 `prefs.json` 配置文件提供可视化的编辑，您也可以使用 `$prefs.edit` 函数来编辑任意符合上述格式的 JSON 配置：

```js
const edited = await $prefs.edit({
  "title": "SETTINGS",
  "groups": [
    {
      "title": "GENERAL",
      "items": [
        {
          "title": "USER_NAME",
          "type": "string",
          "key": "user.name",
          "value": "default user name"
        }
        // ...
      ]
    }
  ]
});
```

返回的数据即为用户编辑后的配置文件，通过这样的方法可以为脚本提供更灵活的配置。

---

## 文件内容解读与示例

### 用途说明

`$prefs` API 提供了一种**简单、高效且用户友好**的方式来为你的 JSBox 脚本创建**设置界面**和**管理配置项**。通过在脚本根目录放置一个 `prefs.json` 文件，JSBox 能够自动为你生成一个原生的、美观的设置页面，用户无需编写任何 UI 代码即可配置脚本。

### 核心概念：`prefs.json` 驱动的 UI

`prefs.json` 文件是 `$prefs` 模块的核心。它是一个 JSON 格式的配置文件，定义了设置页面的结构、每个设置项的类型、默认值以及如何存储和读取它们。`$prefs.open()` 方法会读取这个 JSON 文件，并将其渲染成一个可交互的 iOS 原生设置界面。

#### `prefs.json` 结构详解

一个典型的 `prefs.json` 文件结构如下：

```json
{
  "title": "设置页面标题", // 整个设置页面的标题
  "groups": [ // 包含多个设置分组的数组
    {
      "title": "分组标题", // 设置分组的标题
      "items": [ // 包含多个设置项的数组
        {
          "title": "设置项显示名称", // 显示在设置界面左侧的标题
          "type": "设置项类型", // 决定了右侧的 UI 控件和数据类型
          "key": "设置项的唯一键", // 用于程序化读写此设置项
          "value": "默认值" // 可选，用户未设置时的默认值
          // ... 其他可选属性如 placeholder, inline 等
        }
      ]
    }
  ]
}
```

-   **`title`**: 所有的 `title` 属性都支持本地化。JSBox 会优先从 `strings` 文件中查找对应的本地化字符串。
-   **`key`**: 每个设置项都必须有一个唯一的 `key`。这是你在脚本中通过 `$prefs.get()` 和 `$prefs.set()` 来访问该设置项的唯一标识符。

### 重要的 `type` 类型

`type` 属性决定了设置项在 UI 中的表现形式和其存储的数据类型：

-   **`string` / `password` / `number` / `integer` / `boolean`**: 对应基本的文本输入、密码输入、数字输入和开关。
-   **`slider`**: 显示一个滑块，用于选择 0 到 1 之间的浮点数。如果你的实际值范围不同，需要在脚本中进行映射。
-   **`list`**: 显示一个列表，用户可以从中选择一个值。`items` 属性定义了列表选项，`value` 存储的是用户选择的**索引**（从 0 开始），而不是选项的文本本身。

-   **`info`**: 显示一个只读的信息行，常用于显示版本号等。
-   **`link`**: 显示一个可点击的链接，点击后会打开 URL。
-   **`script`**: **非常强大**。点击此设置项时，可以执行一段预定义的 JavaScript 代码或调用一个脚本中的函数。这允许你为设置页面添加自定义行为，例如“清除缓存”按钮。
-   **`child`**: **非常强大**。允许你创建嵌套的设置页面。`value` 属性是一个与 `prefs.json` 根结构相同的 JSON 对象，可以无限嵌套，用于组织复杂的设置。

### 程序化访问设置

除了用户通过 UI 界面进行设置外，你也可以在脚本中通过 API 读写这些配置项：

-   **`$prefs.get(key)`**: 读取指定 `key` 的设置值。如果用户未设置，则返回 `prefs.json` 中定义的 `value` 默认值。
-   **`$prefs.set(key, value)`**: 以编程方式更新指定 `key` 的设置值。这会覆盖用户在 UI 中设置的值。
-   **`$prefs.all()`**: 返回所有设置项的键值对对象。

### `$prefs.open(handler)` 和 `$prefs.edit(object)`

-   **`$prefs.open(handler)`**: 打开脚本根目录下的 `prefs.json` 文件所定义的设置界面。当用户关闭设置页面时，`handler` 回调会被触发。
-   **`$prefs.edit(object)`**: 这是一个更灵活的方法。它允许你传入一个**任意符合 `prefs.json` 格式的 JSON 对象**，并为其生成一个临时的设置界面。用户编辑后，该方法会返回修改后的 JSON 对象。这对于动态生成的配置或一次性配置非常有用。

### 示例代码：一个带设置的脚本

假设你的脚本根目录下有一个 `prefs.json` 文件，内容如下：

```json
// prefs.json
{
  "title": "我的脚本设置",
  "groups": [
    {
      "title": "通用设置",
      "items": [
        {
          "title": "用户名",
          "type": "string",
          "key": "user_name",
          "value": "访客"
        },
        {
          "title": "启用夜间模式",
          "type": "boolean",
          "key": "dark_mode_enabled",
          "value": false
        },
        {
          "title": "选择主题",
          "type": "list",
          "key": "app_theme",
          "items": ["默认", "蓝色", "绿色"],
          "value": 0
        },
        {
          "title": "清除缓存",
          "type": "script",
          "value": "clearCache()"
        }
      ]
    }
  ]
}
```

然后在你的主脚本中：

```javascript
// 主脚本文件

// 定义 prefs.json 中 script 类型调用的函数
function clearCache() {
  $cache.clear();
  $ui.toast("缓存已清除！");
}

// 脚本启动时，读取并应用设置
const userName = $prefs.get("user_name");
const darkModeEnabled = $prefs.get("dark_mode_enabled");
const appThemeIndex = $prefs.get("app_theme");
const themes = ["默认", "蓝色", "绿色"];

console.log(`欢迎，${userName}！`);
console.log(`夜间模式: ${darkModeEnabled ? "已启用" : "已禁用"}`);
console.log(`当前主题: ${themes[appThemeIndex]}`);

// 渲染一个简单的 UI，包含一个打开设置的按钮
$ui.render({
  props: {
    title: "脚本主界面"
  },
  views: [
    {
      type: "label",
      props: {
        text: `当前用户: ${userName}\n夜间模式: ${darkModeEnabled ? "开" : "关"}\n主题: ${themes[appThemeIndex]}`, 
        lines: 0
      },
      layout: make => {
        make.center.equalTo(view.super);
        make.left.right.inset(20);
      }
    },
    {
      type: "button",
      props: { title: "打开设置" },
      layout: make => {
        make.top.equalTo(view.prev.bottom).offset(30);
        make.centerX.equalTo(view.super);
        make.width.equalTo(150);
      },
      events: {
        tapped: () => {
          $prefs.open(() => {
            // 设置页面关闭后，重新读取设置并更新 UI
            const updatedUserName = $prefs.get("user_name");
            const updatedDarkMode = $prefs.get("dark_mode_enabled");
            const updatedThemeIndex = $prefs.get("app_theme");
            $("label").text = `当前用户: ${updatedUserName}\n夜间模式: ${updatedDarkMode ? "开" : "关"}\n主题: ${themes[updatedThemeIndex]}`;
            $ui.toast("设置已更新！");
          });
        }
      }
    }
  ]
});
