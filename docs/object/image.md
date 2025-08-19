# image

`image` 类型表示图片对象。

属性 | 类型 | 读写 | 说明
---|---|---|---
size | $size | 只读 | 尺寸
orientation | number | 只读 | 方向
info | object | 只读 | metadata
scale | number | 只读 | scale
png | data | 只读 | png 表示的二进制数据

# alwaysTemplate

返回一个使用 `template` 模式渲染的图片，结合 `tintColor` 可以用于对模板图片进行着色：

```js
{
  type: "image",
  props: {
    tintColor: $color("red"),
    image: rawImage.alwaysTemplate
  }
}
```

上述 `rawImage` 是原始的图片。

# alwaysOriginal

与 `alwaysTemplate` 相对于，此属性返回的图片总是渲染自身的颜色，而非 `tintColor`。

# resized($size)

返回调整尺寸的图片：

```js
const resized = image.resized($size(100, 100));
```

# jpg(number)

返回 jpg 表示的二进制数据，number 表示压缩质量(0 ~ 1)：

```js
const jpg = image.jpg(0.8);
```

# colorAtPixel($point)

获得某个点的颜色值：

```js
const color = image.colorAtPixel($point(0, 0));
const hexCode = color.hexCode;
```

# averageColor

获得整张图片的平均颜色：

```js
const avgColor = image.averageColor;
```

# orientationFixedImage

获得旋转方向修正了的图片：

```js
const fixedImage = image.orientationFixedImage;
```

---

## 文件内容解读与示例

### 用途说明

`image` 对象是 JSBox 中用于表示**内存中的图片数据**的核心类型。它由 `$image()` 函数创建，或通过 `$photo` API、`$http.download` 等方式获取。`image` 对象封装了图片的像素数据和元信息，并提供了丰富的属性和方法，用于获取图片信息、进行格式转换以及执行一些基本的图像处理操作。

### 核心概念：图片数据的封装与操作

`image` 对象提供了一种统一的方式来处理各种图片内容，并且提供了多种便捷的属性和方法，用于在不同格式之间进行转换（如转换为 PNG/JPEG `$data`）以及进行基本的图像处理。

### 属性与方法详解

#### 1. 图片信息属性 (只读)

-   **`image.size`**: 图片的尺寸，一个 `$size` 对象，包含 `width` 和 `height`（以点为单位）。
-   **`image.width` / `image.height`**: （补充）直接访问图片的宽度和高度。
-   **`image.orientation`**: 图片的旋转方向（例如，横向、纵向）。
-   **`image.info`**: 包含图片的元数据，如 EXIF 信息等。
-   **`image.scale`**: 图片的缩放比例（例如，`2.0` 代表 @2x 图像）。

#### 2. 格式转换方法

-   **`image.png`**: 将图片转换为 PNG 格式的 `$data` 对象。PNG 是一种无损压缩格式，适合图标和需要透明背景的图片。
-   **`image.jpg(quality)`**: 将图片转换为 JPEG 格式的 `$data` 对象。`quality` 参数是一个 `0.0` 到 `1.0` 之间的浮点数，表示压缩质量（`1.0` 为最高质量，文件最大）。JPEG 是一种有损压缩格式，适合照片。

#### 3. 图像处理与显示相关方法

-   **`image.alwaysTemplate`**: 返回一个图片的“模板”版本。模板图片只保留形状信息，不保留颜色信息。这使得你可以通过 `tintColor` 属性对其进行着色，常用于图标或需要根据主题色变化的图片。
-   **`image.alwaysOriginal`**: 返回图片的原始版本，确保其颜色不会被 `tintColor` 影响。与 `alwaysTemplate` 相对。
-   **`image.resized($size)`**: 返回一个**新**的 `$image` 对象，其尺寸被调整为指定的 `$size`。这是一个简单的缩放操作。
-   **`image.colorAtPixel($point)`**: 获取图片在指定坐标 `$point` 处的像素颜色，返回一个 `color` 对象。
-   **`image.averageColor`**: 获取整张图片的平均颜色，返回一个 `color` 对象。常用于根据图片内容动态调整 UI 颜色。
-   **`image.orientationFixedImage`**: 返回一个修正了方向的图片。有些图片在拍摄时带有方向信息，直接显示可能会旋转，此方法可以修正图片的显示方向。

### 示例代码：图片信息与处理

下面的示例将演示如何从相册选择一张图片，显示其基本信息，并对其进行一些简单的处理（如获取平均颜色、转换为不同格式）。

```javascript
$ui.render({
  props: { title: "Image 对象示例" },
  views: [
    {
      type: "image",
      props: { id: "display-image", bgcolor: $color("lightGray"), contentMode: $contentMode.scaleAspectFit },
      layout: make => make.top.inset(20).centerX.equalTo(view.super).size.equalTo($size(200, 200))
    },
    {
      type: "label",
      props: { id: "info-label", text: "图片信息:", lines: 0 },
      layout: make => make.top.equalTo($("display-image").bottom).offset(20).left.right.inset(20)
    },
    {
      type: "button",
      props: { title: "选择图片" },
      layout: make => make.top.equalTo($("info-label").bottom).offset(20).centerX.equalTo(view.super).width.equalTo(120),
      events: {
        tapped: async () => {
          const resp = await $photo.pick();
          if (resp && resp.image) {
            const pickedImage = resp.image;
            $("display-image").image = pickedImage;

            // 显示图片信息
            let infoText = `图片尺寸: ${pickedImage.width}x${pickedImage.height}\n`;
            infoText += `缩放比例: ${pickedImage.scale}\n`;
            infoText += `平均颜色: ${pickedImage.averageColor.hexCode.toUpperCase()}\n`;
            infoText += `PNG 大小: ${pickedImage.png.info.size} bytes\n`;
            infoText += `JPG (80%) 大小: ${pickedImage.jpg(0.8).info.size} bytes`;
            $("info-label").text = infoText;

            // 演示 resized
            const resizedImage = pickedImage.resized($size(50, 50));
            console.log("调整大小后的图片尺寸:", resizedImage.width, resizedImage.height);

            // 演示 colorAtPixel
            const pixelColor = pickedImage.colorAtPixel($point(10, 10));
            console.log("像素 (10,10) 颜色:", pixelColor.hexCode);

          } else {
            $ui.toast("未选择图片。");
          }
        }
      }
    }
  ]
});
```

**代码解读**：

1.  用户点击“选择图片”按钮后，通过 `$photo.pick()` 获取一个 `image` 对象。
2.  获取到 `image` 对象后，我们直接访问其 `width`, `height`, `scale`, `averageColor` 等属性来显示图片的基本信息。
3.  `pickedImage.png` 和 `pickedImage.jpg(0.8)` 演示了如何将 `image` 对象转换为不同格式的 `$data` 对象，并获取其大小。
4.  `pickedImage.resized()` 和 `pickedImage.colorAtPixel()` 演示了 `image` 对象提供的基本图像处理方法。

`image` 对象是 JSBox 中处理视觉内容的核心。熟练掌握其属性和方法，将使你能够轻松地获取图片信息、进行格式转换和执行基本的图像处理。
