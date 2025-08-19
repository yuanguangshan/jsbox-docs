# Dark Mode

JSBox 最新版已支持 Dark Mode，不仅是应用本身，更提供了一套 API 让脚本也可以很好地支持 Dark Mode。

本章我们将会讲解 Dark Mode 的机制是什么，以及如何对脚本进行适配。

# theme

`theme` 指定了该脚本的外观偏好，可选值是 `light` / `dark` / `auto`，分别表示始终 Light、始终 Dark，以及自动模式。

可以为脚本全局指定这个参数，例如：

```js
$app.theme = "auto";
```

或是在 [config.json](package/intro.md) 文件中指定：

```json
{
  "settings": {
    "theme": "auto"
  }
}
```

除了全局设置，还可以为某个单独的页面指定不同的 `theme`，写在其 `props` 里面即可：

```js
$ui.push({
  props: {
    "theme": "light"
  }
});
```

也可以对某个特定的 view 指定他的外观偏好：

```js
$ui.render({
  views: [
    {
      type: "view",
      props: {
        "theme": "light"
      }
    }
  ]
});
```

> 请注意，为了避免对现有的脚本造成破坏性的改动，目前脚本默认的 `theme` 是 `light`，也既浅色模式。如果需要适配 Dark Mode，请将其设置为 `auto` 后，再进行颜色的适配工作。

在不同的 theme 下，控件的默认颜色会有所不同，请参考更新后的 [UIKit-Catalog](https://github.com/cyanzhong/xTeko/blob/master/extension-scripts/uikit-catalog.js) 样例以了解更多。

# 动态颜色

为了更好地为 Light 和 Dark 适配不同的颜色，现提供动态颜色接口，例如：

```js
const dynamicColor = $color({
  light: "#FFFFFF",
  dark: "#000000"
});
```

也可以简写为：

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

动态颜色会在 Light 和 Dark 模式下分别呈现不同的颜色，而无需通过监听 Dark Mode 变化之后进行重新设置。

同时，`$color(...)` 接口也支持了几个[语义化颜色](function/index.md?id=colorstring)，让你可以更方便地调用。

# 动态图片

类似的，你可能需要为 Light 或 Dark 提供两套不同的图片资源，像是这样：

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

> 请注意，这套机制并不适用于 SF Symbols 和在线资源。对于 SF Symbols，请用动态颜色设置 `tintColor` 来完成

# events: themeChanged

对于任何一个控件，都能收到一个 Dark Mode 改变时的事件，例如：

```js
$ui.render({
  views: [
    {
      type: "view",
      layout: $layout.fill,
      events: {
        themeChanged: (sender, isDarkMode) => {
          // Update UI if needed
        }
      }
    }
  ]
});
```

这提供了需要动态改变一些元素的机会，例如根据是否 Dark Mode 来决定控件的 `alpha` 值，或者改变其 `borderColor`（borderColor 不支持动态颜色）。

# Blur Effect

在 iOS 13 及以上，[type: "blur"](component/blur.md) 可以使用更多模糊效果。部分模糊效果可以根据 Light 或 Dark 自动切换，请使用 [$blurStyle](data/constant.md?id=blurstyle) 来实现。

请参考 Apple [相关文档](https://developer.apple.com/documentation/uikit/uiblureffectstyle)以了解更多。

# WebView

WebView 对 Dark Mode 的适配属于比较独立的话题，请参考 [WebKit 官方文档](https://webkit.org/blog/8840/dark-mode-support-in-webkit/)以了解如何适配网页内容。

需要指出的是：对于 JSBox 内置的 WebView，你需要将 `props: opaque` 设置为 `false` 以避免初次打开页面时的白屏问题。

# 如何适配 Dark Mode

大体上来说，让脚本支持 Dark Mode 有三个步骤：

- 将 `theme` 设置为 `auto`
- 使用 `$color(light, dark)` 和 `$image(light, dark)` 提供动态资源
- 使用 `themeChanged` 来完善一些需要动态改变的元素

为了更好地展示这套机制如何工作，我们准备了一个样例项目以供参考：https://github.com/cyanzhong/jsbox-dark-mode

随着机制不断完善，之后可能会增加其他接口，让适配工作变得更简单。同时，`theme` 默认为 `light` 是过渡期的一个设置，之后可能会改为默认为 `auto`，方便仅使用默认控件的脚本直接完美支持 Dark Mode。

---

## 文件内容解读与示例

### 用途说明

本文档是为 JSBox 脚本适配 **深色模式（Dark Mode）** 的权威指南。深色模式是现代操作系统的一项重要功能，能减少眼部疲劳，并在 OLED 屏幕上节省电量。JSBox 提供了一套完整的 API，让你的脚本能够优雅地自动适应系统外观（亮色/暗色）的变化。

### 适配三步曲

文档最后总结了适配的核心步骤，这也是理解整个机制的最佳路径：

1.  **开启自动主题 (`theme: "auto"`)**: 这是适配的第一步，也是最关键的一步。默认情况下，为了兼容旧脚本，所有脚本都以 `light` (亮色) 模式运行。你必须显式地将主题设置为 `auto`，脚本才会开始响应系统的外观变化。
2.  **使用动态资源 (`$color`/`$image`)**: 对于颜色和图片，不要使用写死的值，而应该提供亮色和暗色两套资源。系统会根据当前模式自动选择合适的资源进行显示。
3.  **处理动态变化 (`themeChanged` 事件)**: 对于那些无法使用动态资源（如 `borderColor`）或需要更复杂逻辑的 UI 调整，可以通过监听 `themeChanged` 事件来手动更新。

### API 详解

#### 1. `theme` 属性

-   **作用**: 控制脚本的整体外观模式。
-   **可选值**: `light` (强制亮色), `dark` (强制暗色), `auto` (跟随系统)。
-   **设置范围**: 
    -   **全局**: 通过 `$app.theme = "auto"` 或在 `config.json` 中设置，影响整个脚本。
    -   **页面级**: 在 `$ui.push` 的 `props` 中设置，仅影响新推入的页面。
    -   **视图级**: 在某个 view 的 `props` 中设置，仅影响该视图及其子视图。

#### 2. `$color(light, dark)` - 动态颜色

-   **作用**: 创建一个能根据系统外观自动改变的颜色。
-   **用法**: 
    -   `$color("#FFFFFF", "#000000")`: 亮色模式下为白色，暗色模式下为黑色。
    -   `$color({ light: "white", dark: "black", black: "#000000" })`: 支持为纯黑模式（`black`）提供单独的颜色。
    -   JSBox 还提供了一系列**语义化颜色**（如 `$color("primarySurface")`），它们已经内置了动态适配能力，是设置背景、文本颜色的首选。

#### 3. `$image(light, dark)` - 动态图片

-   **作用**: 创建一个能根据系统外观自动改变的图片对象。
-   **用法**: `$image("light-icon.png", "dark-icon.png")`。脚本会根据当前模式加载对应的图片文件。
-   **限制**: 不适用于 SF Symbols，因为 SF Symbols 的颜色应该通过 `tintColor` 配合动态颜色来控制。

#### 4. `events: themeChanged`

-   **作用**: 当系统外观（亮/暗）发生切换时触发的事件。
-   **用法**: 在任何视图的 `events` 中定义 `themeChanged: (sender, isDarkMode) => { ... }`。
-   **`isDarkMode`**: 一个布尔值，`true` 表示当前为暗色模式，`false` 为亮色模式。
-   **适用场景**: 
    -   修改不支持动态颜色的属性，如 `borderColor`。
    -   执行更复杂的 UI 调整，比如根据模式改变视图的 `alpha` 透明度或布局。

### 示例：创建一个完美适配深色模式的卡片

```javascript
// 1. 全局开启自动主题
$app.theme = "auto";

// 2. 定义动态颜色
const cardBackgroundColor = $color("white", "#2C2C2E"); // 卡片背景色
const textColor = $color("black", "white"); // 主要文字颜色
const shadowColor = $color("gray"); // 阴影颜色

$ui.render({
  props: {
    title: "深色模式适配",
    bgcolor: $color("primarySurface") // 使用语义化颜色作为页面背景
  },
  views: [
    {
      type: "view",
      props: {
        bgcolor: cardBackgroundColor,
        cornerRadius: 10,
        shadow: {
          radius: 5,
          opacity: 0.2,
          color: shadowColor,
          offset: $point(0, 2)
        }
      },
      layout: (make, view) => {
        make.center.equalTo(view.super);
        make.size.equalTo($size(200, 100));
      },
      views: [
        {
          type: "label",
          props: {
            text: "动态卡片",
            font: $font("bold", 20),
            textColor: textColor
          },
          layout: (make, view) => {
            make.center.equalTo(view.super);
          }
        }
      ],
      events: {
        // 3. 监听主题变化，手动更新边框
        themeChanged: (sender, isDarkMode) => {
          sender.borderColor = isDarkMode ? $color("darkGray") : $color("lightGray");
          sender.borderWidth = 1;
        }
      }
    }
  ]
});
```

### 总结

为 JSBox 脚本适配深色模式是提升用户体验的重要一环。通过遵循“开启主题-使用动态资源-监听事件”的三步曲，开发者可以轻松构建出在任何外观模式下都表现出色的精美界面。