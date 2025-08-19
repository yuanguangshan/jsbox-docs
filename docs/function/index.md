# $l10n

扩展在显示文字的时候支持多种语言，JSBox 提供了简单的本地化支持，`l10n` 是 `Localization` 的缩写，因为刚好有 10 个字母。

当然，你也可以通过 `$device.info` 获取当前设备的语言，进而手动提供多种语言，`$l10n` 是一个简单的封装，让代码可以更简单。

```js
const text = $l10n("MAIN_TITLE");
```

# $delay(number, function)

一种更简单的执行延时任务的方法：

```js
$delay(3, () => {
  $ui.alert("Hey!")
})
```

3 秒钟后弹出一个 alert。

你可以通过这个方式取消执行：

```js
const task = $delay(10, () => {

});

// Cancel it
task.cancel();
```

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
systemTertiaryFill | UIColor.tertiarySystemFillColor
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

# $accessibilityAction(title, handler)

创建一个 [UIAccessibilityCustomAction](https://developer.apple.com/documentation/uikit/uiaccessibilitycustomaction)，例如：

```js
{
  type: "view",
  props: {
    isAccessibilityElement: true,
    accessibilityCustomActions: [
      $accessibilityAction("Hello", () => alert("Hello"))
    ]
  }
}
```

# $objc_retain(object)

在有些时候通过 Runtime 声明的对象会被系统释放掉，如果你想要长期持有一个对象，可以使用这个方法：

```js
const manager = $objc("Manager").invoke("new");
$objc_retain(manager)
```

这将在整个脚本运行期间保持 manger 不被释放。

# $objc_release(object)

与 retain 相对应的函数，目的是手动释放掉对象：

```js
$objc_release(manager)
```

# $get_protocol(name)

获得一个 Objective-C 的 Protocol:

```js
const p = $get_protocol(name);
```

# $objc_clean()

清除所有的 Objective-C 定义：

```js
$objc_clean();
```

---

## 文件内容解读与示例

### 用途说明

`function/index.md` 文件是一个**全局函数（Global Functions）的汇总**。它列举了 JSBox 环境中一些常用的、可以直接调用的全局函数。其中大部分函数在其他更具体的文档（如 `data/method.md` 和 `foundation/thread.md`）中已有详细介绍，本节将重点补充介绍此处新增的、与 Objective-C Runtime 交互相关的函数。

### 已在其他文档中详细介绍的函数

以下函数已在 `data/method.md`（数据类型构造函数）和 `foundation/thread.md`（线程与延时）中进行了详细解读，请查阅相应文档获取更多信息：

-   **`$l10n(key)`**: 本地化文本获取。
-   **`$delay(seconds, handler)`**: 延时执行任务。
-   **`$rect(x, y, width, height)`**: 创建矩形对象。
-   **`$size(width, height)`**: 创建尺寸对象。
-   **`$point(x, y)`**: 创建点对象。
-   **`$insets(top, left, bottom, right)`**: 创建边距对象。
-   **`$color(string)` / `$rgb(...)` / `$rgba(...)`**: 创建颜色对象。
-   **`$font(name, size)`**: 创建字体对象。
-   **`$range(location, length)`**: 创建范围对象。
-   **`$indexPath(section, row)`**: 创建索引路径对象。
-   **`$data(object)`**: 创建二进制数据对象。
-   **`$image(object, scale)`**: 创建图片对象。
-   **`$icon(code, color, size)`**: 获取 JSBox 内置图标。

### 新增函数详解：Objective-C Runtime 交互

以下函数提供了与 iOS 底层 Objective-C Runtime 进行交互的能力，它们是高级功能，通常用于实现 JSBox 未直接封装的原生特性。

#### 1. `$accessibilityAction(title, handler)`

-   **用途**: 创建一个自定义的辅助功能动作（`UIAccessibilityCustomAction`）。这允许你为 UI 元素添加 VoiceOver 用户可以通过手势（如双指轻点两下并按住，或在元素上上下滑动）触发的自定义操作。
-   **参数**: 
    -   `title`: 辅助功能动作的标题，VoiceOver 会朗读这个标题。
    -   `handler`: 当用户触发此动作时执行的回调函数。

**示例**：为列表项添加自定义辅助功能动作

```javascript
$ui.render({
  props: { title: "辅助功能示例" },
  views: [
    {
      type: "list",
      props: {
        data: [
          {
            title: "邮件 1",
            // 为这个列表项添加自定义辅助功能动作
            isAccessibilityElement: true, // 确保视图是可访问的
            accessibilityLabel: "邮件 1", // VoiceOver 朗读的标签
            accessibilityCustomActions: [
              $accessibilityAction("标记为已读", () => {
                $ui.toast("邮件 1 已标记为已读");
              }),
              $accessibilityAction("归档", () => {
                $ui.toast("邮件 1 已归档");
              })
            ]
          },
          { title: "邮件 2" }
        ]
      },
      layout: $layout.fill
    }
  ]
});
```

#### 2. `$objc_retain(object)` 和 `$objc_release(object)`

-   **用途**: 用于手动管理通过 `$objc` 创建的原生 Objective-C 对象的内存引用计数。`retain` 增加引用计数，`release` 减少引用计数。这通常在自动引用计数（ARC）无法正确管理某些原生对象生命周期时使用。
-   **警告**: 这是非常高级且危险的操作。不正确的使用会导致内存泄漏或程序崩溃。除非你非常清楚 Objective-C 的内存管理机制，否则不建议使用。

#### 3. `$get_protocol(name)`

-   **用途**: 获取一个 Objective-C 协议（Protocol）的引用。协议定义了一组方法，类可以声明遵循这些协议。这在需要动态检查对象是否遵循某个协议，或在实现原生代理（Delegate）模式时会用到。

#### 4. `$objc_clean()`

-   **用途**: 清除所有通过 `$objc` 动态创建或加载的 Objective-C 类和协议定义。这主要用于开发和调试阶段，以确保每次运行脚本时都有一个干净的 Objective-C Runtime 环境。
-   **警告**: 不应在生产脚本中随意调用此函数。

### 总结

`function/index.md` 作为一个全局函数的索引，涵盖了 JSBox 提供的多种便捷工具。其中，与 `$objc` 相关的函数为高级开发者提供了与 iOS 原生环境深度交互的能力，但它们的使用需要对 Objective-C Runtime 有深入的理解。