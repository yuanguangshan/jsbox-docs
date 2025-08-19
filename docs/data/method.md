# 方法列表

基于上述的考虑，JSBox 提供了下列这些方法来进行数据转换。

# $rect(x, y, width, height)

生成一个矩形，例如：

```js
const rect = $rect(0, 0, 100, 100);
```

# $size(width, height)

生成一个大小，例如：

```js
const size = $size(100, 100);
```

# $point(x, y)

生成一个位置，例如：

```js
const point = $point(0, 0);
```

# $insets(top, left, bottom, right)

返回一个边距，例如：

```js
const insets = $insets(10, 10, 10, 10);
```

# $color(string)

返回一个颜色，这里支持两种表达式，十六进制表达式：

```js
const color = $color("#00EEEE");
```

常见颜色名称：

```js
const blackColor = $color("black");
```

名称 | 颜色
---|---
tint | JSBox 主题色
black | 黑色
darkGray | 深灰
lightGray | 浅灰
white | 白色
gray | 灰色
red | 红色
green | 绿色
blue | 蓝色
cyan | 青色
yellow | 黄色
magenta | 品红
orange | 橙色
purple | 紫色
brown | 棕色
clear | 透明

以下颜色为语义化颜色，方便用于 Dark Mode 的适配，他们会在 Dark 和 Light 时展示不同的颜色：

名称 | 颜色
---|---
tintColor | 主题色
primarySurface | 一级背景
secondarySurface | 二级背景
tertiarySurface | 三级背景
primaryText | 一级文字
secondaryText | 二级文字
backgroundColor | 背景颜色
separatorColor | 分割线颜色
groupedBackground | grouped 列表背景色
insetGroupedBackground | insetGrouped 列表背景色

以下颜色为系统默认颜色，参考 [UI Element Colors](https://developer.apple.com/documentation/uikit/uicolor/ui_element_colors)

名称 | 颜色
---|---
systemGray2 | UIColor.systemGray2Color
systemGray3 | UIColor.systemGray3Color
systemGray4 | UIColor.systemGray4Color
systemGray5 | UIColor.systemGray5Color
systemGray6 | UIColor.systemGray6Color
systemLabel | UIColor.labelColor
systemSecondaryLabel | UIColor.secondaryLabelColor
systemTertiaryLabel | UIColor.tertiaryLabelColor
systemQuaternaryLabel | UIColor.quaternaryLabelColor
systemLink | UIColor.linkColor
systemPlaceholderText | UIColor.placeholderTextColor
systemSeparator | UIColor.separatorColor
systemOpaqueSeparator | UIColor.opaqueSeparatorColor
systemBackground | UIColor.systemBackgroundColor
systemSecondaryBackground | UIColor.secondarySystemBackgroundColor
systemTertiaryBackground | UIColor.tertiarySystemBackgroundColor
systemGroupedBackground | UIColor.systemGroupedBackgroundColor
systemSecondaryGroupedBackground | UIColor.secondarySystemGroupedBackgroundColor
systemTertiaryGroupedBackground | UIColor.tertiarySystemGroupedBackgroundColor
systemFill | UIColor.systemFillColor
systemSecondaryFill | UIColor.secondarySystemFillColor
systemTertiaryFill | UIColor.systemTertiaryFillColor
systemQuaternaryFill | UIColor.quaternarySystemFillColor

这些颜色在分别在 Light 和 Dark 模式下使用不同的颜色，例如 `$color("tintColor")` 会在 Light 模式下使用主题色，在 Dark 模式下使用比较亮的蓝色。

可以使用 `$color("namedColors")` 来获取颜色盘里面所有可用的颜色，返回一个字典：

```js
const colors = $color("namedColors");
```

同时，`$color(...)` 接口也可用于返回适配 Dark Mode 需要的动态颜色，像是这样：

```js
const dynamicColor = $color({
  light: "#FFFFFF",
  dark: "#000000"
});
```

该颜色在两种模式下分别为黑色和白色，自动切换，也可以简写为：

```js
const dynamicColor = $color("#FFFFFF", "#000000");
```

写法支持嵌套，你可以用 `$rgba(...)` 接口生成颜色后，用 `$color(...)` 接口生成动态颜色：

```js
const dynamicColor = $color($rgba(0, 0, 0, 1), $rgba(255, 255, 255, 1));
```

另外，JSBox 的 Dark Mode 支持深灰或纯黑两种模式，如果需要对三种状态使用不同的颜色，可以使用：

```js
const dynamicColor = $color({
  light: "#FFFFFF",
  dark: "#141414",
  black: "#000000"
});
```

# $rgb(red, green, blue)

同样是生成颜色，但这里用的是十进制 `0 ~ 255` 的数值：

```js
const color = $rgb(100, 100, 100);
```
# $rgba(red, green, blue, alpha)

同样是生成颜色，但这个支持 `alpha` 通道：

```js
const color = $rgba(100, 100, 100, 0.5);
```

# $font(name, size)

返回一个字体，`name` 字段是可选的：

```js
const font1 = $font(15);
const font2 = $font("bold", 15);
```

其中 name 是 `"bold"` 和 `default` 时，分别会使用粗体和正常字体，否则根据 name 查找系统支持的字体。

有关 iOS 自带的字体请参考：http://iosfonts.com/

# $range(location, length)

返回一个范围，例如：

```js
const range = $range(0, 10);
```

# $indexPath(section, row)

返回一个 indexPath，表示区域和位置，这在 list 和 matrix 视图里面会很常用：

```js
const indexPath = $indexPath(0, 10);
```

# $data(object)

返回一个二进制数据，支持以下构造形式：

```js
// string
const data = $data({
  string: "Hello, World!",
  encoding: 4 // default, refer: https://developer.apple.com/documentation/foundation/nsstringencoding
});
```

```js
// path
const data = $data({
  path: "demo.txt"
});
```

```js
// url
const data = $data({
  url: "https://images.apple.com/v/ios/what-is/b/images/performance_large.jpg"
});
```

```js
// base64
const data = $data({
  base64: "data:image/png;base64,..."
});
```

```js
// byte array
const data = $data({
  byteArray: [116, 101, 115, 116]
})
```

# $image(object, scale)

创建一个 image 对象，支持多种参数类型：

```js
// file path
const image = $image("assets/icon.png");
```

```js
// sf symbols
const image = $image("sunrise");
```

```js
// url
const image = $image("https://images.apple.com/v/ios/what-is/b/images/performance_large.jpg");
```

```js
// base64
const image = $image("data:image/png;base64,...");
```

其中 `scale` 为可选参数，用于设置比例，默认为 1，设置成 0 的时候表示屏幕比例。

在最新版里面，可以使用 `$image(...)` 函数来创建适用于 Dark Mode 的动态图片，例如：

```js
const dynamicImage = $image({
  light: "light-image.png",
  dark: "dark-image.png"
});
```

该图片会分别在 Light 模式和 Dark 模式下使用不同的资源，自动完成切换，也可以简写成：

```js
const dynamicImage = $image("light-image.png", "dark-image.png");
```

除此之外，此接口还支持将图片嵌套，像是这样：

```js
const lightImage = $image("light-image.png");
const darkImage = $image("dark-image.png");
const dynamicImage = $image(lightImage, darkImage);
```

# $icon(code, color, size)

获得一个 JSBox 内置的图标，图标编号请参考：https://github.com/cyanzhong/xTeko/tree/master/extension-icons

使用样例：

```js
$ui.render({
  views: [
    {
      type: "button",
      props: {
        icon: $icon("005", $color("red"), $size(20, 20)),
        bgcolor: $color("clear")
      },
      layout: function(make, view) {
        make.center.equalTo(view.super)
        make.size.equalTo($size(24, 24))
      }
    }
  ]
})
```

`color` 是可选参数，不填的话使用默认颜色。

`size` 是可选参数，不填的话使用默认大小。

---

## 文件内容解读与示例

### 用途说明

本文档是 JSBox 中用于创建**原生数据类型**的**核心 API 参考**。这些以 `$` 开头的全局方法（如 `$rect`, `$color`, `$font` 等）充当了 JavaScript 世界与底层 iOS 原生世界之间的“翻译官”。它们将 JavaScript 的基本数据类型或简单对象，转换为 iOS 系统能够直接理解和使用的复杂数据结构（如 `CGRect`, `UIColor`, `UIFont` 等）。

掌握这些方法是编写任何非简单 UI 脚本的基础，因为几乎所有 UI 组件的 `props` 都需要这些原生数据类型作为值。

### 方法详解与示例

#### 1. 几何尺寸与位置

- **`$rect(x, y, width, height)`**: 创建一个矩形对象，常用于定义视图的 `frame` 或绘图区域。
  ```javascript
  const myRect = $rect(10, 20, 100, 50); // x=10, y=20, 宽度=100, 高度=50
  // 示例：设置一个视图的 frame
  // view.frame = myRect;
  ```

- **`$size(width, height)`**: 创建一个尺寸对象，常用于定义视图的 `size` 或图片的大小。
  ```javascript
  const mySize = $size(200, 300); // 宽度=200, 高度=300
  // 示例：设置一个视图的固定大小
  // make.size.equalTo(mySize);
  ```

- **`$point(x, y)`**: 创建一个点对象，常用于定义坐标或偏移量。
  ```javascript
  const myPoint = $point(50, 100); // x=50, y=100
  // 示例：设置滚动视图的偏移量
  // scrollView.contentOffset = myPoint;
  ```

- **`$insets(top, left, bottom, right)`**: 创建一个边距对象，常用于定义视图的内边距或外边距。
  ```javascript
  const myInsets = $insets(10, 20, 10, 20); // 上10, 左20, 下10, 右20
  // 示例：设置文本视图的内边距
  // textView.insets = myInsets;
  ```

#### 2. 颜色

- **`$color(string)`**: 最常用的颜色创建方法，支持多种字符串格式。
  - **十六进制**: `$color("#FF0000")` (红色)
  - **常见颜色名称**: `$color("blue")` (蓝色), `$color("clear")` (透明)
  - **语义化颜色**: `$color("primaryText")` (主文本颜色), `$color("backgroundColor")` (背景色)。这些颜色会根据系统的亮/暗模式自动适配，强烈推荐使用。
  - **动态颜色 (亮/暗模式适配)**: `$color({ light: "#FFFFFF", dark: "#000000" })` 或 `$color("#FFFFFF", "#000000")`。在亮色模式下是白色，暗色模式下是黑色。

- **`$rgb(red, green, blue)`**: 使用 0-255 的 RGB 值创建颜色。
  ```javascript
  const pureRed = $rgb(255, 0, 0);
  ```

- **`$rgba(red, green, blue, alpha)`**: 使用 0-255 的 RGB 值和 0.0-1.0 的透明度（alpha）创建颜色。
  ```javascript
  const semiTransparentBlue = $rgba(0, 0, 255, 0.5);
  ```

#### 3. 字体

- **`$font(name, size)`**: 创建一个字体对象。
  - `size`: 字体大小，必填。
  - `name`: 字体名称，可选。可以是系统字体名称（如 `"Menlo"`），也可以是 `"bold"`（粗体）或 `"default"`（默认字体）。
  ```javascript
  const normalText = $font(16);
  const titleFont = $font("bold", 20);
  const codeFont = $font("Courier New", 14);
  ```

#### 4. 范围与索引

- **`$range(location, length)`**: 创建一个表示文本范围的对象，常用于富文本处理中指定样式应用的区域。
  ```javascript
  const firstFiveChars = $range(0, 5); // 从第0个字符开始，长度为5
  ```

- **`$indexPath(section, row)`**: 创建一个索引路径对象，用于在 `list` 和 `matrix` 组件中精确地引用某个单元格的位置。
  ```javascript
  const firstSectionThirdRow = $indexPath(0, 2); // 第0个 section 的第2行 (索引从0开始)
  ```

#### 5. 二进制数据

- **`$data(object)`**: 创建一个二进制数据对象，可以从多种来源构建。
  - `string`: 从字符串创建，可指定编码。
  - `path`: 从本地文件路径创建。
  - `url`: 从网络 URL 创建。
  - `base64`: 从 Base64 编码的字符串创建。
  - `byteArray`: 从字节数组创建。
  ```javascript
  const helloData = $data({ string: "Hello, World!", encoding: 4 }); // UTF-8
  const imageFileData = $data({ path: "assets/my_image.png" });
  ```

#### 6. 图片对象

- **`$image(object, scale)`**: 创建一个图片对象，这个对象可以被 `image` 组件使用，也可以用于图片处理。
  - `object`: 可以是本地文件路径、SF Symbols 名称、网络 URL 或 Base64 字符串。
  - `scale`: 可选，图片比例。`0` 表示屏幕比例。
  - **动态图片**: 类似 `$color`，也支持根据亮/暗模式自动切换图片资源：`$image({ light: "light_icon.png", dark: "dark_icon.png" })`。
  ```javascript
  const localIcon = $image("assets/app_icon.png");
  const systemSymbol = $image("star.fill");
  const remoteImage = $image("https://example.com/photo.jpg");
  ```

#### 7. 内置图标

- **`$icon(code, color, size)`**: 获取 JSBox 内置的图标。`code` 是图标的编号，`color` 和 `size` 是可选参数。
  ```javascript
  const settingsIcon = $icon("001", $color("blue"), $size(24, 24));
  // 示例：在 button 中使用
  // props: { icon: settingsIcon }
  ```

### 总结

这些 `$` 开头的方法是 JSBox 脚本与 iOS 原生 API 交互的基石。它们确保了数据在 JavaScript 和原生层之间能够正确、高效地传递。熟练掌握这些方法，是编写任何复杂 JSBox 脚本的必经之路。 
