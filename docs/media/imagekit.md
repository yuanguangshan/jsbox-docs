# 图像处理

JSBox 2.1.0 增加了图像处理模块 `$imagekit`，通过这个模块你可以很轻松的处理图像，例如：

- 调整大小
- 旋转
- 制作灰度图像
- 图像叠加
...

为了更直观的介绍，我们构建了一个使用了所有接口的样例项目：https://github.com/cyanzhong/jsbox-imagekit

# $imagekit.render(options, handler)

创建一张图片，并可以使用 canvas 进行绘制：

```js
const options = {
  size: $size(100, 100),
  color: $color("#00FF00"),
  // scale: default to screen scale
  // opaque: default to false
}

const image = $imagekit.render(options, ctx => {
  const centerX = 50;
  const centerY = 50;
  const radius = 25;
  ctx.fillColor = $color("#FF0000");
  ctx.moveToPoint(centerX, centerY - radius);
  for (let i = 1; i < 5; ++i) {
    const x = radius * Math.sin(i * Math.PI * 0.8);
    const y = radius * Math.cos(i * Math.PI * 0.8);
    ctx.addLineToPoint(x + centerX, centerY - y);
  }
  ctx.fillPath();
});
```

`ctx` 与 `canvas` 组件中的一样，请参考 [canvas](component/canvas.md) 文档。

# $imagekit.info(image)

获取图片信息：

```js
const info = $imagekit.info(source);
// width, height, orientation, scale, props
```

# $imagekit.grayscale(image)

转换成灰度图像：

```js
const output = $imagekit.grayscale(source);
```

# $imagekit.invert(image)

将图像反色：

```js
const output = $imagekit.invert(source);
```

# $imagekit.sepia(image)

添加棕褐色滤镜：

```js
const output = $imagekit.sepia(source);
```

# $imagekit.adjustEnhance(image)

自动改善图像效果：

```js
const output = $imagekit.adjustEnhance(source);
```

# $imagekit.adjustRedEye(image)

自动红眼消除：

```js
const output = $imagekit.adjustRedEye(source);
```

# $imagekit.adjustBrightness(image, value)

调整图片亮度：

```js
const output = $imagekit.adjustBrightness(source, 100);
// value range: (-255, 255)
```

# $imagekit.adjustContrast(image, value)

调整图片对比度：

```js
const output = $imagekit.adjustContrast(source, 100);
// value range: (-255, 255)
```

# $imagekit.adjustGamma(image, value)

调整图片伽马值：

```js
const output = $imagekit.adjustGamma(source, 4);
// value range: (0.01, 8)
```

# $imagekit.adjustOpacity(image, value)

调整图片不透明度：

```js
const output = $imagekit.adjustOpacity(source, 0.5);
// value range: (0, 1)
```

# $imagekit.blur(image, bias)

高斯模糊效果：

```js
const output = $imagekit.blur(source, 0);
```

# $imagekit.emboss(image, bias)

浮雕效果：

```js
const output = $imagekit.emboss(source, 0);
```

# $imagekit.sharpen(image, bias)

锐化：

```js
const output = $imagekit.sharpen(source, 0);
```

# $imagekit.unsharpen(image, bias)

钝化:

```js
const output = $imagekit.unsharpen(source, 0);
```

# $imagekit.detectEdge(source, bias)

边缘检测：

```js
const output = $imagekit.detectEdge(source, 0);
```

# $imagekit.mask(image, mask)

使用 `mask` 作为蒙版进行切图：

```js
const output = $imagekit.mask(source, mask);
```

# $imagekit.reflect(image, height, fromAlpha, toAlpha)

创建上下镜像的图片，从 `height` 处折叠，透明度从 `fromAlpha` 变化到 `toAlpha`：

```js
const output = $imagekit.reflect(source, 100, 0, 1);
```

# $imagekit.cropTo(image, size, mode)

图片裁剪：

```js
const output = $imagekit.cropTo(source, $size(100, 100), 0);
// mode:
//   - 0: top-left
//   - 1: top-center
//   - 2: top-right
//   - 3: bottom-left
//   - 4: bottom-center
//   - 5: bottom-right
//   - 6: left-center
//   - 7: right-center
//   - 8: center
```

# $imagekit.scaleBy(image, value)

使用比例调整图片大小：

```js
const output = $imagekit.scaleBy(source, 0.5);
```

# $imagekit.scaleTo(image, size, mode)

使用 size 调整图片大小：

```js
const output = $imagekit.scaleTo(source, $size(100, 100), 0);
// mode:
//   - 0: scaleFill
//   - 1: scaleAspectFit
//   - 2: scaleAspectFill
```

# $imagekit.scaleFill(image, size)

使用 `scaleFill` 模式调整大小：

```js
const output = $imagekit.scaleFill(source, $size(100, 100));
```

# $imagekit.scaleAspectFit(image, size)

使用 `scaleAspectFit` 模式调整大小：

```js
const output = $imagekit.scaleAspectFit(source, $size(100, 100));
```

# $imagekit.scaleAspectFill(image, size)

使用 `scaleAspectFill` 模式调整大小：

```js
const output = $imagekit.scaleAspectFill(source, $size(100, 100));
```

# $imagekit.rotate(image, radians)

旋转图片（将会调整图像大小）：

```js
const output = $imagekit.rotate(source, Math.PI * 0.25);
```

# $imagekit.rotatePixels(image, radians)

旋转图片（不会改变图像大小，可能会裁剪）：

```js
const output = $imagekit.rotatePixels(source, Math.PI * 0.25);
```

# $imagekit.flip(image, mode)

获得镜像图片：

```js
const output = $imagekit.flip(source, 0);
// mode:
//   - 0: vertically
//   - 1: horizontally
```

# $imagekit.concatenate(images, space, mode)

拼接多张图片，可以添加间距：

```js
const output = $imagekit.concatenate(images, 10, 0);
// mode:
//   - 0: vertically
//   - 1: horizontally
```

# $imagekit.combine(image, mask, mode)

将两个图片叠加：

```js
const output = $imagekit.combine(image1, image2, mode);
// mode:
//   - 0: top-left
//   - 1: top-center
//   - 2: top-right
//   - 3: bottom-left
//   - 4: bottom-center
//   - 5: bottom-right
//   - 6: left-center
//   - 7: right-center
//   - 8: center (default)
//   - $point(x, y): absolute position
```

# $imagekit.rounded(image, radius)

获取圆角图片：

```js
const output = $imagekit.rounded(source, 10);
```

# $imagekit.circular(image)

获取正圆形图片，如果原图不是正方形则会居中并从来裁剪：

```js
const output = $imagekit.circular(source);
```

# $imagekit.extractGIF(data) -> Promise

将 GIF 文件分解成单帧：

```js
const {images, durations} = await $imagekit.extractGIF(data);
// image: all image frames
// durations: duration for each frame
```

# $imagekit.makeGIF(source, options) -> Promise

将 image 数组或视频文件 data 合成为 GIF 文件：

```js
const images = [image1, image2];
const options = {
  durations: [0.5, 0.5],
  // size: 16, 12, 8, 4, 2
}
const data = await $imagekit.makeGIF(images, options);
```

若使用 `duration` 替代 `durations`，则每张图片时长一致，使用 GIF 时不需要指定时长。

# $imagekit.makeVideo(source, options) -> Promise

将 image 数组或 GIF 文件合成为视频文件：

```js
const images = [image1, image2];
const data = await $imagekit.makeVideo(images, {
  durations: [0.5, 0.5]
});
```

若使用 `duration` 替代 `durations`，则每张图片时长一致，使用 GIF 时不需要指定时长。

---

## 文件内容解读与示例

### 用途说明

`$imagekit` API 是 JSBox 中一个功能极其强大且全面的**图像处理模块**。它提供了从基础的图片信息获取、尺寸调整、裁剪、旋转，到各种滤镜效果、图像合成，甚至 GIF 和视频的创建与提取等一系列功能。它是你在 JSBox 中进行任何复杂图像操作的首选工具，能够让你在脚本中实现专业的图像编辑效果。

### 核心概念：输入 `$image`，输出 `$image`

`$imagekit` 的大多数方法都遵循一个简洁而强大的模式：它们接收一个或多个 `$image` 对象作为输入，执行图像处理操作，然后返回一个新的 `$image` 对象作为结果。这种设计使得你可以轻松地将多个图像处理操作**链式调用**，实现复杂的效果。

### 功能分类与示例

#### 1. 图片创建与信息

-   **`$imagekit.render(options, ctx => { ... })`**: 通过 Canvas 绘制来创建图片。这允许你用编程方式生成完全自定义的图像，`ctx` 的用法与 `canvas` 组件相同。
-   **`$imagekit.info(image)`**: 获取图片的基本信息，如宽度、高度、方向等。

**示例**：创建一个带文字的图片

```javascript
const generatedImage = $imagekit.render({ size: $size(200, 100), color: $color("white") }, ctx => {
  ctx.fillColor = $color("black");
  ctx.font = $font("bold", 30);
  ctx.fillText("JSBox", 50, 60);
  ctx.strokeColor = $color("red");
  ctx.setLineWidth(2);
  ctx.strokeRect($rect(0, 0, 200, 100));
});
$ui.alert({ title: "生成的图片", image: generatedImage });
```

#### 2. 滤镜与调整

-   **`$imagekit.grayscale(image)`**: 转换为灰度图。
-   **`$imagekit.invert(image)`**: 反色。
-   **`$imagekit.sepia(image)`**: 棕褐色滤镜。
-   **`$imagekit.blur(image, radius)`**: 高斯模糊效果，`radius` 控制模糊程度。
-   **`$imagekit.adjustBrightness(image, value)`**: 调整亮度，`value` 范围通常为 (-255, 255)。
-   **`$imagekit.sharpen(image, bias)`**: 锐化图片。

**示例**：应用滤镜效果

```javascript
async function applyFilters() {
  const originalImage = await $photo.pick(); // 让用户选择一张图片
  if (!originalImage) return;

  const sepiaImage = $imagekit.sepia(originalImage);
  const blurredImage = $imagekit.blur(sepiaImage, 5); // 模糊半径 5
  const finalImage = $imagekit.sharpen(blurredImage, 0.5); // 锐化 0.5

  $ui.alert({ title: "滤镜效果", image: finalImage });
}
// applyFilters();
```

#### 3. 尺寸与裁剪

-   **`$imagekit.scaleBy(image, scale)`**: 按比例缩放图片。
-   **`$imagekit.scaleTo(image, size, mode)`**: 缩放到指定尺寸，`mode` 可选 `scaleFill` (拉伸填充), `scaleAspectFit` (保持比例适应), `scaleAspectFill` (保持比例填充裁剪)。
-   **`$imagekit.cropTo(image, size, mode)`**: 裁剪图片到指定尺寸，`mode` 可选裁剪的起始位置。
-   **`$imagekit.rounded(image, radius)`**: 获取圆角图片。
-   **`$imagekit.circular(image)`**: 获取正圆形图片（如果原图不是正方形会居中裁剪）。

**示例**：裁剪并制作圆形头像

```javascript
async function makeCircularAvatar() {
  const originalImage = await $photo.pick();
  if (!originalImage) return;

  // 裁剪到正方形 (假设从中心裁剪)
  const croppedImage = $imagekit.cropTo(originalImage, $size(200, 200), 8); 
  // 制作成圆形
  const circularImage = $imagekit.circular(croppedImage);

  $ui.alert({ title: "圆形头像", image: circularImage });
}
// makeCircularAvatar();
```

#### 4. 旋转与翻转

-   **`$imagekit.rotate(image, radians)`**: 旋转图片，会调整图片大小以适应旋转后的内容。
-   **`$imagekit.rotatePixels(image, radians)`**: 旋转图片像素，不改变图片大小，可能会裁剪。
-   **`$imagekit.flip(image, mode)`**: 垂直 (`0`) 或水平 (`1`) 翻转图片。

#### 5. 图像合成与拼接

-   **`$imagekit.combine(image1, image2, mode)`**: 将两张图片叠加。`mode` 可选叠加位置或指定 `$point` 绝对位置。
-   **`$imagekit.concatenate(images, space, mode)`**: 拼接多张图片，`mode` 可选垂直 (`0`) 或水平 (`1`) 拼接，`space` 为间距。

#### 6. GIF 与视频处理 (异步)

-   **`$imagekit.extractGIF(data)`**: 将 GIF 文件分解成单帧图片和每帧时长。
-   **`$imagekit.makeGIF(images, options)`**: 将图片数组或视频文件合成为 GIF。支持设置每帧时长。
-   **`$imagekit.makeVideo(images, options)`**: 将图片数组或 GIF 合成为视频文件。

**示例**：将多张图片合成为 GIF

```javascript
async function makeGifFromPhotos() {
  const images = await $photo.pick({ multi: true, count: 3 }); // 选择3张图片
  if (!images || images.length === 0) return;

  $ui.toast("正在生成 GIF...");
  const gifData = await $imagekit.makeGIF(images, { duration: 0.5 }); // 每帧显示0.5秒

  if (gifData) {
    $ui.alert({ title: "GIF 生成成功", image: gifData });
    // 你也可以保存到相册: $photo.save({"data": gifData});
  } else {
    $ui.alert("GIF 生成失败。");
  }
}
// makeGifFromPhotos();
```

### 总结

`$imagekit` 是 JSBox 中进行图像处理的瑞士军刀。它提供了从基础调整到复杂合成、甚至动态媒体创建的全面能力。熟练掌握这些方法，将使你的脚本在视觉表现力上达到新的高度。 
