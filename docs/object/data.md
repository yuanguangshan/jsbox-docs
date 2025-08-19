# data

`data` 类型表示一个二进制数据，亦即一个文件。

属性 | 类型 | 读写 | 说明
---|---|---|---
info | object | 只读 | metadata
string | string | 只读 | 转换成 UTF8 字符串
byteArray | [number] | 只读 | 转换成 byte array
image | image | 只读 | 转换成 image
fileName | string | 只读 | 可能的文件名
gzipped | $data | 只读 | 获取 gzip 后的文件
gunzipped | $data | 只读 | 获取 gunzip 后的文件
isGzipped | bool | 只读 | 检测是否是一个 gzip 文件

---

## 文件内容解读与示例

### 用途说明

`$data` 对象是 JSBox 中用于表示**原始二进制数据**（Raw Binary Data）的核心类型。你可以把它想象成内存中的一个文件或一段字节流。在 JSBox 中，所有非文本文件（如图片、音频、视频、PDF）以及网络请求的原始响应体，都以 `$data` 对象的形式进行处理。它在文件读写、网络通信、图像处理、加密解密等场景中无处不在。

### 核心概念：二进制数据的封装与转换

`$data` 对象提供了一种统一的方式来处理各种二进制内容，并且提供了多种便捷的属性和方法，用于在不同格式之间进行转换（如转换为字符串、图片、Base64 等）。

### 属性详解

-   **`data.info`**: 只读，一个包含数据元信息（如 `size` 字节大小）的对象。
    ```javascript
    const fileData = $file.read("assets/example.png");
    console.log("文件大小 (字节):", fileData.info.size);
    ```

-   **`data.string`**: 只读，将二进制数据尝试解码为 UTF-8 字符串。适用于文本文件或已知编码的二进制数据。
    ```javascript
    const textData = $data({ string: "Hello, World!" });
    console.log("字符串内容:", textData.string);
    ```

-   **`data.byteArray`**: 只读，将二进制数据转换为 JavaScript 的 `Uint8Array` 或普通数字数组。每个元素代表一个字节（0-255）。
    ```javascript
    const textData = $data({ string: "ABC" });
    console.log("字节数组:", textData.byteArray); // 输出: [65, 66, 67]
    ```

-   **`data.image`**: 只读，如果 `$data` 对象包含的是图片格式（如 PNG, JPEG, GIF）的二进制数据，则将其转换为 `$image` 对象。如果不是有效的图片格式，则返回 `null`。
    ```javascript
    // 假设 imageData 是从网络下载的图片 $data
    // const imageObject = imageData.image;
    // if (imageObject) $ui.alert({ title: "图片预览", image: imageObject });
    ```

-   **`data.fileName`**: 只读，一个可能的推断文件名。通常在通过 `$http.download` 或 `$context.data` 获取 `$data` 时提供。

-   **`data.base64`**: （补充）只读，将二进制数据转换为 Base64 编码的字符串。Base64 编码常用于在文本环境中传输二进制数据。
    ```javascript
    const textData = $data({ string: "Hello" });
    console.log("Base64 编码:", textData.base64); // 输出: SGVsbG8=
    ```

-   **`data.gzipped` / `data.gunzipped` / `data.isGzipped`**: 用于处理 Gzip 压缩/解压缩。
    -   `data.gzipped`: 返回当前 `$data` 对象的 Gzip 压缩版本，结果也是一个 `$data` 对象。
    -   `data.gunzipped`: 返回当前 `$data` 对象的 Gzip 解压缩版本，结果也是一个 `$data` 对象。
    -   `data.isGzipped`: 布尔值，检测当前 `$data` 对象是否是 Gzip 压缩格式。
    ```javascript
    const originalData = $data({ string: "这是一段需要被压缩的文本。" });
    const compressedData = originalData.gzipped;
    console.log("压缩前大小:", originalData.info.size, "压缩后大小:", compressedData.info.size);
    console.log("是否是 Gzip:", compressedData.isGzipped);
    console.log("解压后内容:", compressedData.gunzipped.string);
    ```

### 示例代码：数据格式转换

下面的示例将演示如何将一个字符串转换为 `$data` 对象，然后进行 Base64 编码，再将 Base64 解码回 `$data`，最后再转换回字符串。

```javascript
$ui.render({
  props: { title: "$data 对象示例" },
  views: [
    {
      type: "label",
      props: { text: "原始字符串:", font: $font("bold", 16) },
      layout: make => make.top.left.inset(10)
    },
    {
      type: "label",
      props: { id: "original-text", text: "JSBox Data Object Example" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(5).left.right.inset(10)
    },
    {
      type: "button",
      props: { title: "开始转换" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(20).centerX.equalTo(view.super).width.equalTo(120),
      events: {
        tapped: () => {
          const originalString = $("original-text").text;

          // 1. 字符串 -> $data
          const dataFromString = $data({ string: originalString, encoding: 4 }); // UTF-8
          $("data-from-string-label").text = `Data (from string): ${dataFromString.info.size} bytes`;

          // 2. $data -> Base64 字符串
          const base64String = dataFromString.base64;
          $("base64-label").text = `Base64: ${base64String.substring(0, 30)}...`;

          // 3. Base64 字符串 -> $data
          const dataFromBase64 = $data({ base64: base64String });
          $("data-from-base64-label").text = `Data (from Base64): ${dataFromBase64.info.size} bytes`;

          // 4. $data -> 字符串
          const stringFromData = dataFromBase64.string;
          $("string-from-data-label").text = `String (from Data): ${stringFromData}`; 

          $ui.toast("转换完成！");
        }
      }
    },
    {
      type: "label", props: { text: "Data (from string):" }, layout: make => make.top.equalTo(view.prev.bottom).offset(20).left.inset(10)
    },
    {
      type: "label", props: { id: "data-from-string-label", lines: 0 }, layout: make => make.top.equalTo(view.prev.bottom).offset(5).left.right.inset(10)
    },
    {
      type: "label", props: { text: "Base64:" }, layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.inset(10)
    },
    {
      type: "label", props: { id: "base64-label", lines: 0 }, layout: make => make.top.equalTo(view.prev.bottom).offset(5).left.right.inset(10)
    },
    {
      type: "label", props: { text: "Data (from Base64):" }, layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.inset(10)
    },
    {
      type: "label", props: { id: "data-from-base64-label", lines: 0 }, layout: make => make.top.equalTo(view.prev.bottom).offset(5).left.right.inset(10)
    },
    {
      type: "label", props: { text: "String (from Data):" }, layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.inset(10)
    },
    {
      type: "label", props: { id: "string-from-data-label", lines: 0 }, layout: make => make.top.equalTo(view.prev.bottom).offset(5).left.right.inset(10)
    }
  ]
});
```

**代码解读**：

这个示例清晰地展示了 `$data` 对象作为二进制数据容器的灵活性，以及它如何通过各种属性在字符串、Base64 和其他 `$data` 形式之间进行无缝转换。这对于处理文件内容、网络响应或任何二进制数据流都非常有用。 
