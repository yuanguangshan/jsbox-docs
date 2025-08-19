> JSBox 自带的二维码模块，可以进行编解码和扫描

# $qrcode.encode(string)

将字符串转换成一个二维码图片：

```js
const image = $qrcode.encode("https://apple.com");
```

# $qrcode.decode(image)

将二维码图片解码成字符串：

```js
const text = $qrcode.decode(image);
```

# $qrcode.scan(function)

开始扫描二维码，用户完成扫描之后返回字符串：

```js
$qrcode.scan(text => {
  
})
```

当用户取消扫描时，也支持回调，例如：

```js
$qrcode.scan({
  useFrontCamera: false, // Optional
  turnOnFlash: false, // Optional
  handler(string) {
    $ui.toast(string)
  },
  cancelled() {
    $ui.toast("Cancelled")
  }
})
```

---

## 文件内容解读与示例

### 用途说明

`$qrcode` API 是 JSBox 中用于处理**二维码**的专用模块。它提供了从文本**生成**二维码图片、从二维码图片**解析**文本内容，以及通过摄像头**扫描**现实世界中二维码的完整功能。这使得在你的脚本中集成二维码相关的任务变得非常简单。

### 核心方法详解

#### 1. `$qrcode.encode(string)`: 生成二维码

-   **用途**: 将任意字符串（如网址、文本、Wi-Fi 信息等）编码成一个二维码图片。
-   **参数**: 一个字符串，即你想要编码到二维码中的内容。
-   **返回值**: 一个 `$image` 对象，代表生成的二维码图片。你可以将这个图片显示在 `image` 组件中，或者保存到文件。

**示例**：

```javascript
const qrImage = $qrcode.encode("Hello, JSBox!");
// qrImage 现在是一个 $image 对象，可以用于显示
```

#### 2. `$qrcode.decode(image)`: 解析二维码

-   **用途**: 从一个包含二维码的 `$image` 对象中，解析出其中编码的字符串内容。
-   **参数**: 一个 `$image` 对象，这个图片中必须包含一个可识别的二维码。
-   **返回值**: 如果成功解析，返回二维码中包含的字符串；如果图片中没有二维码或无法识别，则返回 `null`。

**示例**：

```javascript
// 假设 myImage 是一个包含二维码的 $image 对象
// const myImage = $image("assets/my_qrcode.png");
const decodedText = $qrcode.decode(myImage);
if (decodedText) {
  $ui.alert(`二维码内容: ${decodedText}`);
} else {
  $ui.alert("未识别到二维码");
}
```

#### 3. `$qrcode.scan(options)`: 扫描二维码

-   **用途**: 激活设备的摄像头，让用户扫描现实世界中的二维码。这是最常用的二维码功能之一。
-   **参数**: 一个对象，包含 `handler` 回调函数和可选配置。
  - `handler(string)`: 扫描成功后调用的回调函数，参数 `string` 即为扫描到的二维码内容。
  - `cancelled()`: 用户取消扫描时调用的回调函数。
  - `useFrontCamera`: 布尔值，是否使用前置摄像头（默认为 `false`，使用后置）。
  - `turnOnFlash`: 布尔值，是否打开闪光灯（默认为 `false`）。
- **异步操作**: 这是一个异步操作，扫描界面会覆盖当前 UI，扫描结果通过回调函数返回。

**示例**：

```javascript
$qrcode.scan({
  handler: (text) => {
    $ui.alert(`扫描成功: ${text}`);
  },
  cancelled: () => {
    $ui.toast("扫描已取消");
  },
  useFrontCamera: false, // 默认使用后置摄像头
  turnOnFlash: false // 默认不打开闪光灯
});
```

### 示例代码：一个多功能的二维码工具

下面的示例将创建一个简单的 UI，包含生成、解析和扫描二维码的功能。

```javascript
$ui.render({
  props: {
    title: "二维码工具"
  },
  views: [
    {
      type: "input",
      props: {
        id: "encode-input",
        placeholder: "输入要生成二维码的文本或链接"
      },
      layout: make => {
        make.top.left.right.inset(10);
        make.height.equalTo(36);
      }
    },
    {
      type: "button",
      props: { title: "生成二维码" },
      layout: make => {
        make.top.equalTo($("encode-input").bottom).offset(10);
        make.left.right.inset(10);
        make.height.equalTo(36);
      },
      events: {
        tapped: () => {
          const textToEncode = $("encode-input").text;
          if (textToEncode) {
            const qrImage = $qrcode.encode(textToEncode);
            $("qr-image-display").image = qrImage;
          } else {
            $ui.alert("请输入内容！");
          }
        }
      }
    },
    {
      type: "image",
      props: {
        id: "qr-image-display",
        bgcolor: $color("clear"),
        contentMode: $contentMode.scaleAspectFit
      },
      layout: make => {
        make.top.equalTo(view.prev.bottom).offset(10);
        make.centerX.equalTo(view.super);
        make.size.equalTo($size(150, 150));
      }
    },
    {
      type: "button",
      props: { title: "从相册解析二维码" },
      layout: make => {
        make.top.equalTo($("qr-image-display").bottom).offset(20);
        make.left.right.inset(10);
        make.height.equalTo(36);
      },
      events: {
        tapped: async () => {
          const image = await $photo.pick(); // 从相册选择图片
          if (image) {
            const decodedText = $qrcode.decode(image);
            if (decodedText) {
              $ui.alert(`解析结果: ${decodedText}`);
            } else {
              $ui.alert("图片中未识别到二维码");
            }
          }
        }
      }
    },
    {
      type: "button",
      props: { title: "扫描二维码" },
      layout: make => {
        make.top.equalTo(view.prev.bottom).offset(10);
        make.left.right.inset(10);
        make.height.equalTo(36);
      },
      events: {
        tapped: () => {
          $qrcode.scan({
            handler: (text) => {
              $ui.alert(`扫描结果: ${text}`);
            },
            cancelled: () => {
              $ui.toast("扫描已取消");
            }
          });
        }
      }
    }
  ]
});
```

**代码解读**：

1.  **生成**: 用户在输入框输入文本，点击“生成二维码”按钮后，调用 `$qrcode.encode()` 生成图片，并显示在 `qr-image-display` 中。
2.  **解析**: 点击“从相册解析二维码”按钮，通过 `$photo.pick()` 让用户选择一张图片，然后调用 `$qrcode.decode()` 尝试解析其中的二维码。
3.  **扫描**: 点击“扫描二维码”按钮，直接调用 `$qrcode.scan()` 激活摄像头进行扫描。

`$qrcode` 模块为你的脚本提供了完整的二维码处理能力，无论是生成、解析还是扫描，都非常便捷。 
