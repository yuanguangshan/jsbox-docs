> 提供了将图片或文字等信息分享到社交网络的相关方法

# $share.sheet(object)

调用系统的 `share sheet` 分享内容：

```js
$share.sheet(["https://apple.com", "apple"])
```

既可以分享一个数组，也可以分享单独的一个数据。

目前支持的数据类型：`文本`、`链接`、`图片`和`二进制数据`（data）。

当分享二进制数据的时候，可以指定文件名：

```js
$share.sheet([
  {
    "name": "sample.mp4",
    "data": data
  }
])
```

从 Build 80 开始，支持指定回调：

```js
$share.sheet({
  items: [
    {
      "name": "sample.mp4",
      "data": data
    }
  ], // 也支持 item
  handler: function(success) {

  }
})
```

# $share.wechat(object)

分享内容到微信：

```js
$share.wechat(image)
```

将会自动识别分享内容的数据类型，目前支持`文字`、`图片`和`图片二进制数据`。

# $share.qq(object)

分享内容到 QQ:

```js
$share.qq(image)
```

将会自动识别分享内容的数据类型，目前支持`文字`、`图片`和`图片`二进制数据。

# $share.universal(object)

调用 JSBox 内置的分享面板分享内容：

```js
$share.universal(image)
```

将会自动识别分享内容的数据类型，目前支持`图片`和`图片`二进制数据。

---

## 文件内容解读与示例

### 用途说明

`$share` API 模块是你的 JSBox 脚本与 iOS 系统**分享表单（Share Sheet）**交互的桥梁。它允许你的脚本将各种类型的内容（如文本、链接、图片、文件）发送到设备上安装的其他应用程序或直接分享到特定的社交媒体平台。这对于实现脚本与其他应用的无缝协作至关重要。

### 核心方法：`$share.sheet(items)` - 调用系统分享表单

-   **用途**: 这是最常用、最推荐的分享方法。它会弹出 iOS 标准的分享表单，用户可以在其中选择将内容分享到任何支持该内容类型的应用。
-   **`items` 参数**: 可以是一个单独的项目，也可以是一个包含多个项目的数组。支持以下数据类型：
    -   **字符串**: 用于分享纯文本或 URL 链接。
    -   **`$image` 对象**: 用于分享图片。
    -   **`$data` 对象**: 用于分享二进制文件。当分享 `$data` 对象时，你可以提供一个包含 `name`（文件名）和 `data`（`$data` 对象本身）的字典，以便接收方正确识别文件类型。
-   **`handler` 回调**: 可选。分享操作完成后（无论成功或取消），此回调函数会被触发，并接收一个布尔值 `success`。

**示例：分享多种类型的内容**

```javascript
// 分享纯文本
$share.sheet("Hello from JSBox!");

// 分享一个链接
$share.sheet("https://www.jsbox.com");

// 分享一个图片 (假设你有一个 $image 对象)
// const myImage = $image("assets/my_photo.jpg");
// $share.sheet(myImage);

// 分享一个二进制文件 (例如一个 PDF)
// const pdfData = $file.read("assets/document.pdf");
// $share.sheet({
//   name: "MyDocument.pdf",
//   data: pdfData
// });

// 同时分享多种内容
$share.sheet([
  "这是我用 JSBox 脚本生成的内容！",
  "https://www.example.com",
  // myImage // 如果有图片对象，也可以放进来
]);

// 带回调的分享
$share.sheet({
  items: "我正在分享这个！",
  handler: (success) => {
    if (success) {
      $ui.toast("分享成功！");
    } else {
      $ui.toast("分享已取消或失败。");
    }
  }
});
```

### 特定社交媒体分享 (`$share.wechat`, `$share.qq`)

-   **用途**: 这些方法允许你的脚本直接将内容分享到微信或 QQ，而无需通过系统的分享表单。
-   **限制**: 通常只支持分享文本和图片数据。它们的可靠性可能不如系统分享表单，因为它们依赖于目标应用的内部实现。

### `$share.universal(object)` - JSBox 内置分享面板

-   **用途**: 调用 JSBox 自身提供的分享面板。目前主要支持图片和图片二进制数据。
-   **注意**: 对于大多数通用分享需求，`$share.sheet` 是更推荐的选择，因为它能利用 iOS 系统更广泛的分享能力。

### 示例代码：一个分享工具

下面的示例将创建一个简单的 UI，演示如何通过按钮分享不同类型的内容。

```javascript
$ui.render({
  props: {
    title: "分享工具"
  },
  views: [
    {
      type: "button",
      props: { title: "分享文本" },
      layout: make => {
        make.top.left.right.inset(20);
        make.height.equalTo(40);
      },
      events: {
        tapped: () => {
          $share.sheet("你好，世界！这是来自 JSBox 的分享。");
        }
      }
    },
    {
      type: "button",
      props: { title: "分享链接" },
      layout: make => {
        make.top.equalTo(view.prev.bottom).offset(10);
        make.left.right.inset(20);
        make.height.equalTo(40);
      },
      events: {
        tapped: () => {
          $share.sheet("https://www.apple.com");
        }
      }
    },
    {
      type: "button",
      props: { title: "分享二维码图片" },
      layout: make => {
        make.top.equalTo(view.prev.bottom).offset(10);
        make.left.right.inset(20);
        make.height.equalTo(40);
      },
      events: {
        tapped: () => {
          // 假设我们生成一个二维码图片
          const qrImage = $qrcode.encode("https://www.jsbox.com");
          if (qrImage) {
            $share.sheet(qrImage);
          } else {
            $ui.alert("无法生成二维码图片。");
          }
        }
      }
    },
    {
      type: "button",
      props: { title: "分享本地文件" },
      layout: make => {
        make.top.equalTo(view.prev.bottom).offset(10);
        make.left.right.inset(20);
        make.height.equalTo(40);
      },
      events: {
        tapped: () => {
          // 假设我们有一个本地的文本文件
          const fileContent = $data({ string: "这是要分享的本地文件内容。" });
          if (fileContent) {
            $share.sheet({
              name: "my_shared_file.txt",
              data: fileContent
            });
          } else {
            $ui.alert("无法创建文件数据。");
          }
        }
      }
    }
  ]
});
```

**代码解读**：

这个示例展示了如何使用 `$share.sheet` 方法来分享不同类型的数据。无论是简单的字符串、通过 `$qrcode.encode` 生成的图片，还是通过 `$data` 创建的二进制文件，都可以作为 `items` 参数传递给 `$share.sheet`，系统会自动识别并提供相应的分享选项。 
