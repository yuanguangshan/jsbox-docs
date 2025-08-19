# color

`color` 类型表示一个颜色，可以通过 `$color` 函数生成：

```js
const color = $color("#00eeee");
```

# color.hexCode

获得颜色对应的 hex 字符串：

```js
const hexCode = color.hexCode;
// -> "#00eeee"
```

# color.components

获得颜色对应的 RGB 数值：

```js
const components = color.components;
const red = components.red;
const green = components.green;
const blue = components.blue;
const alpha = components.alpha;
```

---

## 文件内容解读与示例

### 用途说明

`color` 对象是 JSBox 中用于表示**单一颜色值**的标准数据类型。它由 `$color()`, `$rgb()`, `$rgba()` 等函数创建。`color` 对象封装了颜色的各种表示形式（如 RGB 分量、十六进制代码），并提供便捷的属性来访问这些信息，使得在 UI 渲染、图像处理或任何需要颜色操作的场景中，颜色的处理变得统一和高效。

### 核心属性详解

`color` 对象提供了多种方式来访问其颜色分量和表示形式：

-   **`color.hexCode`**: 返回颜色的**十六进制字符串**表示。例如，红色会返回 `"#FF0000"`。

-   **`color.components`**: 返回一个包含 `red`, `green`, `blue`, `alpha` 四个分量的对象。这些分量的值都在 `0.0` 到 `1.0` 之间。

-   **`color.red`, `color.green`, `color.blue`, `color.alpha`**: （补充）直接访问颜色的红、绿、蓝、透明度分量。这些属性的值也是在 `0.0` 到 `1.0` 之间。例如，`color.red` 会返回红色分量的值。

-   **`color.rgb`**: （补充）返回颜色的 `rgb(r,g,b)` 字符串表示，其中 `r,g,b` 是 0-255 的整数值。

-   **`color.rgba`**: （补充）返回颜色的 `rgba(r,g,b,a)` 字符串表示，其中 `r,g,b` 是 0-255 的整数值，`a` 是 0.0-1.0 的浮点数。

### 示例代码：一个 RGB 颜色选择器

下面的示例将创建一个简单的 RGB 颜色选择器。通过三个滑块调节红、绿、蓝的值，一个视图会实时显示混合后的颜色，同时标签会显示颜色的十六进制代码和 RGB 分量。

```javascript
$ui.render({
  props: { title: "Color 对象示例" },
  views: [
    {
      type: "view",
      props: {
        id: "color-preview",
        bgcolor: $color("black"), // 初始颜色
        cornerRadius: 10
      },
      layout: make => {
        make.top.inset(20);
        make.centerX.equalTo(view.super);
        make.size.equalTo($size(150, 150));
      }
    },
    {
      type: "label",
      props: {
        id: "hex-label",
        text: "HEX: #000000",
        align: $align.center
      },
      layout: make => {
        make.top.equalTo($("color-preview").bottom).offset(20);
        make.left.right.inset(20);
      }
    },
    {
      type: "label",
      props: {
        id: "rgb-label",
        text: "RGB: (0, 0, 0)",
        align: $align.center
      },
      layout: make => {
        make.top.equalTo($("hex-label").bottom).offset(5);
        make.left.right.inset(20);
      }
    },
    // 红色滑块
    createColorSlider("red-slider", "Red", $color("red"), 0),
    // 绿色滑块
    createColorSlider("green-slider", "Green", $color("green"), 1),
    // 蓝色滑块
    createColorSlider("blue-slider", "Blue", $color("blue"), 2)
  ]
});

function createColorSlider(id, labelText, trackColor, index) {
  return {
    type: "stack",
    props: {
      axis: $stackViewAxis.horizontal,
      alignment: $stackViewAlignment.center,
      spacing: 10
    },
    layout: make => {
      make.top.equalTo(index === 0 ? $("rgb-label").bottom.offset(20) : view.prev.bottom.offset(10));
      make.left.right.inset(20);
      make.height.equalTo(30);
    },
    views: [
      { type: "label", props: { text: labelText, width: 40 } },
      {
        type: "slider",
        props: {
          id: id,
          min: 0,
          max: 255,
          value: 0,
          minColor: trackColor,
          maxColor: $color("lightGray"),
          thumbColor: trackColor,
          continuous: true
        },
        layout: $layout.fill,
        events: { changed: updateColorDisplay }
      }
    }
  ];
}

function updateColorDisplay() {
  const r = $("red-slider").value;
  const g = $("green-slider").value;
  const b = $("blue-slider").value;

  const newColor = $rgb(r, g, b);
  $("color-preview").bgcolor = newColor;

  $("hex-label").text = `HEX: ${newColor.hexCode.toUpperCase()}`;
  $("rgb-label").text = `RGB: (${Math.round(newColor.red * 255)}, ${Math.round(newColor.green * 255)}, ${Math.round(newColor.blue * 255)})`;
}

// 初始化显示
updateColorDisplay();
```

**代码解读**：

1.  我们创建了三个滑块来分别控制红、绿、蓝的 0-255 值。
2.  `updateColorDisplay()` 函数是核心。它从滑块获取值，然后使用 `$rgb()` 函数创建一个新的 `color` 对象。
3.  `$("color-preview").bgcolor = newColor;` 直接将新创建的 `color` 对象赋值给视图的背景色。
4.  `newColor.hexCode` 和 `newColor.red` 等属性被用来获取颜色的不同表示形式，并更新到 `hex-label` 和 `rgb-label` 上。

`color` 对象是 JSBox 中处理颜色的统一接口，它使得颜色的创建、访问和转换变得非常便捷。 
