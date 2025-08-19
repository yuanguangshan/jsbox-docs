> 剪贴板对于 iOS 的数据分享和交换很重要，JSBox 提供了很多相关接口。

# $clipboard.text

```js
// 获取剪贴板文本
const text = $clipboard.text;
// 设置剪贴板文本
$clipboard.text = "Hello, World!"
```

# $clipboard.image

```js
// 获取剪贴板图片，请注意返回的是二进制数据
const data = $clipboard.image;
// 设置剪贴板图片
$clipboard.image = data
```

# $clipboard.items

```js
// 获取剪贴板中的所有项目
const items = $clipboard.items;
// 设置剪贴板中的所有项目
$clipboard.items = items
```

# $clipboard.phoneNumbers

获取剪贴板中的所有电话号码。

# $clipboard.phoneNumber

获取剪贴板中的第一个电话号码。

# $clipboard.links

获取剪贴板中的所有链接。

# $clipboard.link

获取剪贴板中的第一个链接。

# $clipboard.emails

获取剪贴板中的所有 email。

# $clipboard.email

获取剪贴板中的第一个 email。

# $clipboard.dates

获取剪贴板中的所有日期。

# $clipboard.date

获取剪贴板中的第一个日期。

# $clipboard.setTextLocalOnly(string)

设置剪贴板的文本，但忽略 `Universal Clipboard`：

# $clipboard.set(object)

通过 `type` 和 `value` 设置剪贴板，例如：

```js
$clipboard.set({
  "type": "public.plain-text",
  "value": "Hello, World!"
})
```

# $clipboard.copy(object)

此方法可以设置剪贴板过期时间：

```js
$clipboard.copy({
  text: "Temporary text",
  ttl: 20
})
```

支持参数：

属性 | 类型 | 说明
---|---|---
text | string | 文本
image | image | 图片
data | data | 数据
ttl | number | 几秒之后过期
locally | bool | 本地剪贴板

*关于 `UTTypes` 的介绍：https://developer.apple.com/documentation/mobilecoreservices/uttype*

# $clipboard.clear()

清除剪贴板里面的全部内容。

---

## 文件内容解读与示例

### 用途说明

`$clipboard` API 是你的 JSBox 脚本与 iOS 系统**剪贴板**进行交互的核心接口。它允许你的脚本读取和写入文本、图片、文件等多种类型的数据，是实现应用间数据交换、自动化流程以及快速信息处理的重要工具。

### 核心概念

-   **通用剪贴板 (Universal Clipboard)**: iOS 和 macOS 设备之间可以自动同步剪贴板内容。`$clipboard` 的一些方法允许你控制是否参与这种同步。
-   **数据类型**: 剪贴板可以承载多种数据类型。`$clipboard` 提供了便捷的属性来访问这些不同类型的数据。

### 属性与方法详解

#### 1. 基本读写属性

-   **`$clipboard.text`**: 用于读写剪贴板中的**纯文本**内容。最常用。
    ```javascript
    // 读取
    const clipboardText = $clipboard.text;
    // 写入
    $clipboard.text = "新的剪贴板内容";
    ```
-   **`$clipboard.image`**: 用于读写剪贴板中的**图片**内容。读取时返回 `$data` 对象，写入时接受 `$data` 或 `$image` 对象。
    ```javascript
    // 读取
    const clipboardImageData = $clipboard.image;
    // 写入 (假设 myImageData 是一个 $data 对象)
    // $clipboard.image = myImageData;
    ```
-   **`$clipboard.items`**: 返回剪贴板中所有项目的数组。每个项目是一个对象，包含 `type` (UTI) 和 `value`。适用于处理多类型或复杂剪贴板内容。

#### 2. 智能数据检测属性 (只读)

这些属性会自动从剪贴板内容中识别并提取特定类型的数据，类似于 `$detector` 模块的功能，但直接作用于剪贴板。

-   **`$clipboard.phoneNumbers` / `$clipboard.phoneNumber`**: 识别电话号码（复数返回数组，单数返回第一个）。
-   **`$clipboard.links` / `$clipboard.link`**: 识别 URL 链接。
-   **`$clipboard.emails` / `$clipboard.email`**: 识别电子邮件地址。
-   **`$clipboard.dates` / `$clipboard.date`**: 识别日期。

#### 3. 高级写入方法

-   **`$clipboard.setTextLocalOnly(string)`**: 写入文本到剪贴板，但**阻止其同步到其他设备**（即不参与通用剪贴板）。
-   **`$clipboard.set(object)`**: 允许你明确指定要写入的数据类型（通过 UTI，如 `"public.plain-text"`）和值。适用于写入特定格式的数据。
-   **`$clipboard.copy(object)`**: 最灵活的写入方法，支持：
    -   `text`, `image`, `data`: 要写入的内容。
    -   `ttl`: **过期时间**（Time-To-Live），单位秒。剪贴板内容会在指定时间后自动清除。非常适合临时性数据。
    -   `locally`: 布尔值，如果为 `true`，则不参与通用剪贴板同步。

#### 4. 清除剪贴板

-   **`$clipboard.clear()`**: 清除剪贴板中的所有内容。

### 示例代码：一个剪贴板工具

下面的示例将创建一个简单的 UI，演示如何读写剪贴板，以及使用过期时间功能。

```javascript
$ui.render({
  props: {
    title: "剪贴板工具"
  },
  views: [
    {
      type: "label",
      props: {
        id: "clipboard-content-label",
        text: "剪贴板内容: (空)",
        lines: 0
      },
      layout: make => {
        make.top.left.right.inset(20);
        make.height.equalTo(80);
      }
    },
    {
      type: "button",
      props: { title: "读取剪贴板文本" },
      layout: make => {
        make.top.equalTo(view.prev.bottom).offset(10);
        make.left.right.inset(20);
        make.height.equalTo(40);
      },
      events: {
        tapped: () => {
          const text = $clipboard.text;
          $("clipboard-content-label").text = `剪贴板内容: ${text || "(空)"}`;
        }
      }
    },
    {
      type: "button",
      props: { title: "复制自定义文本" },
      layout: make => {
        make.top.equalTo(view.prev.bottom).offset(10);
        make.left.right.inset(20);
        make.height.equalTo(40);
      },
      events: {
        tapped: () => {
          $clipboard.text = "这是从 JSBox 复制的文本！";
          $ui.toast("文本已复制！");
        }
      }
    },
    {
      type: "button",
      props: { title: "复制临时文本 (5秒后过期)" },
      layout: make => {
        make.top.equalTo(view.prev.bottom).offset(10);
        make.left.right.inset(20);
        make.height.equalTo(40);
      },
      events: {
        tapped: () => {
          $clipboard.copy({
            text: "这条文本将在5秒后自动消失！",
            ttl: 5 // 5秒后过期
          });
          $ui.toast("临时文本已复制！");
        }
      }
    },
    {
      type: "button",
      props: { title: "清除剪贴板" },
      layout: make => {
        make.top.equalTo(view.prev.bottom).offset(10);
        make.left.right.inset(20);
        make.height.equalTo(40);
      },
      events: {
        tapped: () => {
          $clipboard.clear();
          $("clipboard-content-label").text = "剪贴板内容: (已清除)";
          $ui.toast("剪贴板已清除！");
        }
      }
    },
    {
      type: "button",
      props: { title: "检测剪贴板链接" },
      layout: make => {
        make.top.equalTo(view.prev.bottom).offset(10);
        make.left.right.inset(20);
        make.height.equalTo(40);
      },
      events: {
        tapped: () => {
          const links = $clipboard.links;
          if (links && links.length > 0) {
            $ui.alert(`检测到链接:\n${links.join("\n")}`);
          } else {
            $ui.alert("剪贴板中未检测到链接。");
          }
        }
      }
    }
  ]
});
```

**代码解读**：

1.  “读取剪贴板文本”按钮演示了如何通过 `$clipboard.text` 简单地获取文本内容。
2.  “复制自定义文本”按钮演示了如何通过 `$clipboard.text` 写入文本。
3.  “复制临时文本”按钮展示了 `$clipboard.copy` 方法的 `ttl` 参数，使得复制的内容在指定时间后自动消失，非常适合一次性密码或临时信息。
4.  “清除剪贴板”按钮调用 `$clipboard.clear()` 来清空所有内容。
5.  “检测剪贴板链接”按钮则利用了 `$clipboard.links` 属性的智能检测能力。

`$clipboard` API 是实现脚本自动化和与其他应用数据交换的强大工具。
