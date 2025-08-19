> JSBox 也提供了一套用于拍照或选择照片的接口

# $photo.take(object)

拍摄一张照片：

```js
$photo.take({
  handler: function(resp) {
    const image = resp.image;
  }
})
```

支持的参数表，常量请参考[常量表](data/constant.md)：

参数 | 类型 | 说明
---|---|---
edit | boolean | 是否编辑
mediaTypes | array | 媒体类型
maxDuration | number | 视频最大时长
quality | number | 质量
showsControls | boolean | 是否显示控制器
device | number | 前后镜头
flashMode | number | 闪光灯模式

# $photo.pick(object)

从系统相册选择一张图片：

```js
$photo.pick({
  handler: function(resp) {
    const image = resp.image;
  }
})
```

所支持的参数与 `$photo.take` 完全一致，你可以认为他们只是数据来源不同。

与 `take` 方法不同的是，`pick` 可以指定返回数据的类型：

参数 | 类型 | 说明
---|---|---
format | string | 可选 "image" 和 "data"，默认 "image"

此外，`$photo.pick` 支持通过 `multi: true` 来设置选择多个结果，通过 `selectionLimit` 来限制最多选择的张数，多选时返回的结果结构如下：

参数 | 类型 | 说明
---|---|---
status | bool | 是否成功
results | array | 所有结果

results 里面的某一项，当 format 为 `image` 时结构为：

参数 | 类型 | 说明
---|---|---
image | image | 图片对象
metadata | object | 元数据
filename | string | 文件名

当 format 为 `data` 时结构为：

参数 | 类型 | 说明
---|---|---
data | data | 图片二进制数据
metadata | object | 元数据
filename | string | 文件名

# $photo.prompt(object)

弹出一个 action sheet 让用户选择是拍照还是从相册选择图片：

```js
$photo.prompt({
  handler: function(resp) {
    const image = resp.image;
  }
})
```

# 获取图片的 metadata

在 `$photo.pick` 和 `$photo.take` 的返回结果中，可以取得图片的元数据，例如地理位置：

```js
$photo.pick({
  handler: function(resp) {
    const metadata = resp.metadata;
    if (metadata) {
      const gps = metadata["{GPS}"];
      const url = `https://maps.apple.com/?ll=${gps.Latitude},${gps.Longitude}`;
      $app.openURL(url)
    }
  }
})
```

获取图片的 GPS 信息并在 Apple 地图中打开。

# $photo.scan()

使用内置的文档扫描工具（仅 iOS 13）：

```js
const response = await $photo.scan();
// response.status, response.results
```

# $photo.save(object)

将图片存储到相册：

```js
// data
$photo.save({
  data,
  handler: function(success) {

  }
})
```

```js
// image
$photo.save({
  image,
  handler: function(success) {

  }
})
```

# $photo.fetch(object)

在系统相册中读取图片：

```js
$photo.fetch({
  count: 3,
  handler: function(images) {

  }
})
```

支持一些参数来确定搜索的范围，比如图片类型，常量请参考[常量表](data/constant.md)：

参数 | 类型 | 说明
---|---|---
type | number | 媒体类型
subType | number | 媒体子类型
format | string | 可选 "image" 和 "data"，默认 "image"
size | $size | 图片大小，默认原图
count | number | 读取数量

# $photo.delete(object)

在系统相册中删除图片：

```js
$photo.delete({
  count: 3,
  handler: function(success) {

  }
})
```

支持的参数和 `$photo.fetch` 相同。

# image 转换成 data

`image` 类型提供了两个方法用于转换成二进制数据：

```js
// PNG
const png = image.png;
// JPEG
const jpg = image.jpg(0.8);
```

jpg 格式使用 `0.0 ~ 1.0` 来表示质量。

---

## 文件内容解读与示例

### 用途说明

`$photo` API 提供了与 iOS 设备**相册和相机**进行深度交互的能力。它允许你的脚本：

-   **拍摄照片或视频**
-   **从系统相册选择图片或视频**
-   **保存内容到相册**
-   **访问和管理相册中的媒体文件**

这对于任何需要处理视觉媒体的脚本都至关重要，例如图片处理工具、视频编辑辅助、或相册管理脚本。

### 核心概念：权限与异步操作

-   **权限**: 所有相机和相册操作都需要用户授权。首次调用相关接口时，系统会弹出权限请求。你的脚本应妥善处理用户拒绝授权的情况。
-   **异步**: 大多数 `$photo` 操作都是异步的，结果通过 `handler` 回调函数或 `await` Promise 返回。这确保了 UI 在等待用户操作或处理媒体文件时不会卡顿。

### 主要功能与方法详解

#### 1. 获取媒体：`$photo.take()`, `$photo.pick()`, `$photo.prompt()`

-   **`$photo.take(options)`**: 启动相机界面，让用户拍摄照片或录制视频。
-   **`$photo.pick(options)`**: 打开系统相册，让用户选择一张或多张照片/视频。
    -   **共同参数**: `edit` (是否允许编辑), `mediaTypes` (媒体类型，如图片或视频), `quality` (图片质量), `device` (前后摄像头), `flashMode` (闪光灯)。
    -   **`pick` 特有**: `format` (返回 `image` 或 `data`), `multi` (多选), `selectionLimit` (多选数量限制)。
-   **`$photo.prompt(options)`**: 弹出一个 Action Sheet，让用户选择是拍照还是从相册选择。
-   **返回结果**: 这些方法都返回一个包含 `image` (`$image` 对象), `data` (`$data` 对象), `metadata` (元数据，如 GPS 信息), `filename` 的 `resp` 对象。

**示例**：从相册选择图片并显示其元数据

```javascript
async function pickAndShowMetadata() {
  const resp = await $photo.pick(); // 默认选择单张图片
  if (resp && resp.image) {
    $ui.alert({
      title: "图片信息",
      message: `文件名: ${resp.filename}\n宽度: ${resp.image.width}\n高度: ${resp.image.height}\n元数据: ${JSON.stringify(resp.metadata, null, 2)}`
    });
  } else {
    $ui.toast("未选择图片或操作取消。");
  }
}
// pickAndShowMetadata();
```

#### 2. 保存媒体：`$photo.save(options)`

-   **用途**: 将 `$data` 或 `$image` 对象保存到系统相册。

**示例**：保存一张图片到相册

```javascript
async function saveImageToAlbum() {
  const imageToSave = await $photo.pick(); // 让用户选择一张图片
  if (!imageToSave) return;

  $photo.save({
    image: imageToSave.image, // 传入 $image 对象
    handler: (success) => {
      if (success) {
        $ui.toast("图片已保存到相册！");
      } else {
        $ui.toast("图片保存失败。");
      }
    }
  });
}
// saveImageToAlbum();
```

#### 3. 相册管理：`$photo.fetch()`, `$photo.delete()`

-   **`$photo.fetch(options)`**: 从系统相册中读取图片或视频。可以按数量、媒体类型、子类型、格式、尺寸等进行筛选。
-   **`$photo.delete(options)`**: 从系统相册中删除图片或视频。

**示例**：获取相册中最新的 3 张图片

```javascript
$photo.fetch({
  count: 3, // 获取最新3张
  handler: (images) => {
    if (images && images.length > 0) {
      $ui.alert(`成功获取到 ${images.length} 张最新图片。`);
      // images 是一个 $image 对象数组，你可以进一步处理它们
    } else {
      $ui.toast("未获取到图片。");
    }
  }
});
```

#### 4. 文档扫描：`$photo.scan()`

-   **用途**: 启动 iOS 内置的文档扫描工具（仅 iOS 13+）。用户可以扫描纸质文档，系统会自动进行边缘检测和图像优化。
-   **返回值**: Promise，成功时解析为包含扫描结果的对象（通常包含扫描到的图片）。

**示例**：

```javascript
async function scanDocument() {
  const response = await $photo.scan();
  if (response.status === "success" && response.results && response.results.length > 0) {
    $ui.alert(`扫描到 ${response.results.length} 页文档。`);
    // response.results[0].image 是扫描到的第一页图片
    // $quicklook.open({ image: response.results[0].image });
  } else {
    $ui.toast("文档扫描取消或失败。");
  }
}
// scanDocument();
```

#### 5. 图片格式转换

-   **`image.png` / `image.jpg(quality)`**: 这些是 `$image` 对象的方法，用于将图片转换为 PNG 或 JPEG 格式的 `$data` 对象。`jpg` 方法接受一个 `quality` 参数（0.0-1.0）。

**示例**：将选中的图片转换为 JPEG 格式

```javascript
async function convertToJpeg() {
  const pickedImage = await $photo.pick();
  if (pickedImage && pickedImage.image) {
    const jpegData = pickedImage.image.jpg(0.7); // 转换为质量为 70% 的 JPEG
    $ui.alert({
      title: "图片已转换",
      message: `JPEG 数据大小: ${jpegData.info.size} 字节`
    });
    // 你可以进一步保存或分享这个 jpegData
  } else {
    $ui.toast("未选择图片。");
  }
}
// convertToJpeg();
```

### 总结

`$photo` API 为你的 JSBox 脚本提供了与设备相机和相册进行深度交互的全面能力。无论是获取用户媒体、保存处理后的图片，还是进行文档扫描，它都是不可或缺的工具。 
