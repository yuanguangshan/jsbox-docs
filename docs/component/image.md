# type: "image"

`image` 用于显示图片，可以加载远程的图片或本地图片：

```js
{
  type: "image",
  props: {
    src: "https://images.apple.com/v/ios/what-is/b/images/performance_large.jpg"
  },
  layout: function(make, view) {
    make.center.equalTo(view.super)
    make.size.equalTo($size(100, 100))
  }
}
```

从远程加载一张图片，屏幕上尺寸是 100*100。

src 同样也支持 `data:image` 开头的 base64 字符串格式，例如：

```js
{
  type: "image",
  props: {
    src: "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAB4AAAASCAMAAAB7LJ7rAAAANlBMVEUAAABnZ2dmZmZmZmZnZ2dmZmZmZmZmZmZnZ2dnZ2dnZ2dmZmZoaGhnZ2dnZ2dubm5paWlmZmbvpwLOAAAAEXRSTlMA9h6lQ95r4cmLdHNbTzksJ9o8+Y0AAABcSURBVCjPhc1JDoAwFAJQWus8cv/LqkkjMXwjCxa8BfjLWuI9L/nqhmwiLYnpAMjqpuQMDI+bcgNyW921A+Sxyl3NXeWu7lL3WOXS0Ck1N3WXut/HEz6z92l8Lyf1mAh1wPbVFAAAAABJRU5ErkJggg=="
  },
  layout: function(make, view) {
    make.center.equalTo(view.super)
    make.size.equalTo($size(100, 100))
  }
}
```

src 还支持以 `shared://` 或 `drive://` 开头的文件。

另外 image 也支持显示 JSBox 自带的 icon，[请参考](data/method.md?id=iconcode-color-size)。

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
src | string | 只写 | 图片地址
source | object | 读写 | 图片加载信息
symbol | string | 读写 | SF symbols 名称
data | $data | 只写 | 二进制数据
size | $size | 只读 | 图片大小
orientation | number | 只读 | 图片方向
info | object | 只读 | 图片信息
scale | number | 只读 | 图片比例

# props: source

从 v1.55.0 开始，可以通过 `source` 对图片进行更详细的设定，例如：

```js
source: {
  url: url,
  placeholder: image,
  header: {
    "key1": "value1",
    "key2": "value2",
  }
}
```

# 双指缩放

从 `v1.56.0` 开始，可以很轻松地创建支持双指缩放的图片：

```js
$ui.render({
  views: [
    {
      type: "scroll",
      props: {
        zoomEnabled: true,
        maxZoomScale: 3, // Optional, default is 2,
        doubleTapToZoom: false // Optional, default is true
      },
      layout: $layout.fill,
      views: [
        {
          type: "image",
          props: {
            src: "https://..."
          },
          layout: $layout.fill
        }
      ]
    }
  ]
});
```

只需要用 `scroll` 组件作为容器，并设置为 `zoomEnabled` 即可。

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

返回一个重新调整大小后的图片：

```js
const resizedImage = image.resized($size(60, 60));
```

# resizableImage(args)

返回一个可伸缩的图片：

```js
const resizableImage = image.resizableImage($insets(10, 10, 10, 10));
```

也可以指定填充模式为 `tile` (默认为 stretch):

```js
const resizableImage = image.resizableImage({
  insets: $insets(10, 10, 10, 10),
  mode: "tile"
});
```

---

## 文件内容解读与示例

### 组件用途

`image` 组件是用于在界面上显示图片的基础控件。它功能强大，支持从网络、本地文件、Base64 字符串、甚至系统图标库等多种来源加载图片，并能精细地控制图片的显示方式。

### 核心概念与属性

#### 1. 图片来源 (Source)

你有多种方式来指定要显示的图片：

- **`src: "url"`**: 最常用的方式。可以是 `https://` 开头的网址，也可以是 `shared://` 或 `drive://` 开头的本地文件路径，还可以是 `data:image/png;base64,...` 格式的 Base64 字符串。
- **`data: $data`**: 当你通过网络请求或其他方式获取到图片的二进制数据（一个 `$data` 对象）时，可以使用此属性。
- **`symbol: "name"`**: **（推荐）** 用于显示 Apple 的 SF Symbols 图标。这是一种现代、高效的方式，图标是矢量的，能与文本样式很好地配合。
- **`source: { ... }`**: 一个更高级的加载方式，允许你设置占位图、自定义网络请求头等，适用于需要加载大图或需要认证的场景。

#### 2. 内容模式 (`contentMode`) - 【至关重要】

`contentMode` 属性决定了当图片的原始尺寸与 `image` 视图的 frame 尺寸不匹配时，图片应该如何显示。这是正确布局图片的关键。

- **`$contentMode.scaleAspectFit` (默认)**: 保持图片原始宽高比，**完整显示**整张图片。这可能会导致视图内出现空白区域。
- **`$contentMode.scaleAspectFill`**: 保持图片原始宽高比，**填满**整个视图。这可能会导致图片的部分内容被裁剪掉。
- **`$contentMode.scaleToFill`**: 拉伸或压缩图片，使其**恰好填满**视图，不保持宽高比。通常会导致图片变形。
- **`$contentMode.center`**: 不缩放图片，直接将其原始尺寸的中心对齐到视图的中心。

#### 3. 图片着色 (`tintColor`)

你可以将任何图片（尤其是单色图标）变成“模板图片”，然后用 `tintColor` 对其进行着色，以匹配你的 App 主题色。这通过 `image.alwaysTemplate` 实现。

#### 4. 双指缩放

要让图片支持双指缩放，你需要采用一个固定的组合模式：**将 `image` 视图嵌套在一个 `scroll` 视图内部，并设置 `scroll` 视图的 `zoomEnabled: true`**。缩放功能是由父级的 `scroll` 视图提供的，而非 `image` 视图本身。

### 示例代码：对比不同的 `contentMode`

下面的示例将并排显示三张图片，它们加载相同的源图片，但使用不同的 `contentMode`，以便你直观地理解它们的区别。

```javascript
const imageURL = "https://images.unsplash.com/photo-1517694712202-14dd9538aa97"; // 一张长方形的图片

$ui.render({
  props: {
    title: "Image ContentMode 示例"
  },
  views: [
    {
      // 使用一个水平 stack 容器来并排显示
      type: "stack",
      props: {
        axis: $stackViewAxis.horizontal,
        distribution: $stackViewDistribution.fillEqually,
        spacing: 10
      },
      layout: function(make, view) {
        make.left.right.inset(10);
        make.center.equalTo(view.super);
        make.height.equalTo(120);
      },
      views: [
        // 示例1: scaleAspectFit (默认)
        { type: "label", props: { text: "Fit", align: $align.center } },
        {
          type: "image",
          props: {
            src: imageURL,
            contentMode: $contentMode.scaleAspectFit
          },
          layout: $layout.fill
        },
        // 示例2: scaleAspectFill
        { type: "label", props: { text: "Fill", align: $align.center } },
        {
          type: "image",
          props: {
            src: imageURL,
            contentMode: $contentMode.scaleAspectFill,
            clipped: true // 必须设置，否则会超出边界
          },
          layout: $layout.fill
        },
        // 示例3: scaleToFill
        { type: "label", props: { text: "Stretch", align: $align.center } },
        {
          type: "image",
          props: {
            src: imageURL,
            contentMode: $contentMode.scaleToFill
          },
          layout: $layout.fill
        }
      ].map(view => { // 给图片加个边框，方便观察
        if (view.type === 'image') {
          view.props.border = { width: 1, color: $color("gray") };
        }
        return { type: "view", views: [view], layout: $layout.fill };
      })
    }
  ]
});
```

**代码解读**：

1.  我们选择了一张长方形的图片，并将它显示在三个正方形的 `image` 视图中，这样效果对比会非常明显。
2.  **Fit**: 图片被完整地显示在视图内，但上下有留白。
3.  **Fill**: 图片填满了整个视图，但左右两边的内容被裁切掉了。注意 `clipped: true` 在这里很重要。
4.  **Stretch**: 图片被强行拉伸以填满视图，导致图片中的物体（键盘、笔记本）看起来被压扁了，发生了变形。

通过这个例子，你可以清晰地看到，为 `contentMode` 选择正确的值对于保持 UI 的美观和正确性是多么重要。 
