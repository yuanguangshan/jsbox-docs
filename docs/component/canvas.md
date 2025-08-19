# type: "canvas"

`canvas` 提供的是画图的功能，在绝大部分的情况下，你可以通过各种控件的组合构造出想要的效果，如果哪些控件仍然不让你满意，你可以自己绘图，比如：

```js
{
  type: "canvas",
  layout: $layout.fill,
  events: {
    draw: function(view, ctx) {
      var centerX = view.frame.width * 0.5
      var centerY = view.frame.height * 0.3
      var radius = 50.0
      ctx.fillColor = $color("red")
      ctx.moveToPoint(centerX, centerY - radius)
      for (var i=1; i<5; ++i) {
        var x = radius * Math.sin(i * Math.PI * 0.8)
        var y = radius * Math.cos(i * Math.PI * 0.8)
        ctx.addLineToPoint(x + centerX, centerY - y)
      }
      ctx.fillPath()
    }
  }
}
```

这将在屏幕上画一个红色的五角星，半径为 50pt。

# ctx

`canvas` 控件其实也是有一点点复杂，目前并未实现完全，请参考 Apple [官方文档](https://developer.apple.com/documentation/coregraphics)获得更多信息。

上述代码的 `ctx` 对象是我们绘制图形的上下文，要理解这个概念首先要阅读 Apple 的官方文档，其次我将介绍目前 JSBox 里面实现了的部分。

# ctx.fillColor

`fillColor` 是用于填充的颜色。

# ctx.strokeColor

`strokeColor` 是用于描边的颜色。

# ctx.font

`font` 是绘制文字时的字体。

# ctx.fontSize

`fontSize` 是绘制文字时的字号大小。

# ctx.allowsAntialiasing

`allowsAntialiasing` 表示是否允许自动抗锯齿，抗锯齿会获得更平滑的效果，但性能有所损失。

# saveGState()

存储状态。

# restoreGState()

恢复状态。

# scaleCTM(sx, sy)

进行缩放操作。

# translateCTM(tx, ty)

进行平移操作。

# rotateCTM(scale)

进行旋转操作。

# setLineWidth(width)

设置描边宽度。

# setLineCap(lineCap)

请参考文档：https://developer.apple.com/documentation/coregraphics/cglinecap

# setLineJoin(lineJoin)

请参考文档：https://developer.apple.com/documentation/coregraphics/cglinejoin

# setMiterLimit(miterLimit)

请参考文档：https://developer.apple.com/documentation/coregraphics/cgcontext/1456499-setmiterlimit

# setAlpha(alpha)

设置透明度。

# beginPath()

路径开始。

# moveToPoint(x, y)

移动到 `(x, y)` 这个点。

# addLineToPoint(x, y)

从当前点画一条线到 `(x, y)` 这个点。

# addCurveToPoint(cp1x, cp1y, cp2x, cp2y, x, y)

绘制一条曲线到 `(x, y)`，通过 `(cp1x, cp1y)` 和 `(cp2x, cp2y)` 进行曲率控制。

# addQuadCurveToPoint(cpx, cpy, x, y)

还是绘制曲线到 `(x, y)` 这个点，但是这次只有一个控制点 `(cpx, cpy)`。

# closePath()

将路径闭合。

# addRect(rect)

添加一个矩形。

# addArc(x, y, radius, startAngle, endAngle, clockwise)

添加一条弧线，以 `(x, y)` 为中点，`radius` 为半径，`startAngle` 为起始弧度，`endAngle` 为终止弧度，`clockwise` 表示是否顺时针。

# addArcToPoint(x1, y1, x2, y2, radius)

在 `(x1, y1)` 和 `(x2, y2)` 之间添加一条弧线。

# fillRect(rect)

填充一个矩形。

# strokeRect(rect)

对矩形进行描边。

# clearRect(rect)

清除一个矩形。

# fillPath()

填充一个闭合的路径。

# strokePath()

对闭合的路径进行描边。

# drawPath(mode)

使用指定模式绘制路径：

```
0: kCGPathFill,
1: kCGPathEOFill,
2: kCGPathStroke,
3: kCGPathFillStroke,
4: kCGPathEOFillStroke,
```

参考：https://developer.apple.com/documentation/coregraphics/1455195-cgcontextdrawpath

# drawImage(rect, image)

将 `image` 绘制到 `rect` 这个矩形上。

# drawText(rect, text, attributes)

将文字绘制到 `rect` 这个矩形上：

```js
ctx.drawText($rect(0, 0, 100, 100), "你好", {
  color: $color("red"),
  font: $font(30)
});
```

---

## 文件内容解读与示例

### 组件用途

`canvas`（画布）是一个功能极其强大的底层绘图组件。当标准的 UI 组件（如 `label`, `image`）无法满足你对视觉和布局的精细要求时，`canvas` 允许你像在画板上一样，通过代码绘制出任何你想要的二维图形，例如自定义图表、游戏画面、复杂背景等。

### 核心概念解析

要使用 `canvas`，必须理解以下几个核心概念：

1.  **`draw` 事件**: 所有的绘图代码都必须写在 `events: { draw: function(view, ctx) { ... } }` 函数中。系统会在需要渲染画布时自动调用这个函数。

2.  **绘图上下文 `ctx`**: 这是最重要的概念。`ctx` 对象可以被理解为你的“画笔和调色盘”。你不能直接操作画布，而是通过调用 `ctx` 的各种方法来告诉它“画什么”以及“怎么画”。例如，`ctx.fillColor = $color("red")` 就是把画笔的填充颜色设置为红色。

3.  **坐标系**: 画布的坐标系以左上角为原点 `(0, 0)`，x 轴向右增长，y 轴向下增长。`view.frame.width` 和 `view.frame.height` 分别是画布的宽度和高度，是计算位置的关键。

4.  **绘图三部曲（路径模式）**: 对于绘制复杂的图形（非矩形），通常遵循以下步骤：
    a. **设置状态**: 设置颜色、线条宽度等，如 `ctx.fillColor`。
    b. **定义路径**: 使用 `ctx.beginPath()` 开始，然后用 `moveToPoint()`（移动画笔）、`addLineToPoint()`（画直线）、`addArc()`（画圆弧）等方法来描述图形的轮廓。
    c. **渲染路径**: 使用 `ctx.fillPath()`（填充路径）或 `ctx.strokePath()`（描边路径）将之前定义的路径实际绘制出来。

5.  **状态的保存与恢复**: `saveGState()` 和 `restoreGState()` 用于保存和恢复 `ctx` 的当前状态（如颜色、旋转、缩放等）。这在绘制多个独立图形时非常有用，可以确保对一个图形的样式设置不会影响到另一个。

### 示例代码：绘制多种图形

下面的示例将在一个画布上绘制一个带边框的圆形、一个矩形、一段文字和一个旋转的线条，以演示 `canvas` 的基本用法。

```javascript
$ui.render({
  props: {
    title: "Canvas 示例"
  },
  views: [
    {
      type: "canvas",
      layout: $layout.fill,
      events: {
        draw: function(view, ctx) {
          // 获取画布尺寸
          const width = view.frame.width;
          const height = view.frame.height;

          // --- 1. 绘制一个带边框的红色圆形 ---
          ctx.setLineWidth(4); // 设置线条宽度为 4
          ctx.strokeColor = $color("black"); // 边框为黑色
          ctx.fillColor = $color("red"); // 填充为红色
          
          ctx.beginPath(); // 开始定义路径
          // 添加一个圆弧路径：圆心(60, 60)，半径50，从0到2*PI弧度（一个完整的圆）
          ctx.addArc(60, 60, 50, 0, Math.PI * 2, true);
          ctx.closePath(); // 闭合路径
          
          // 先填充再描边，这样边框不会被填充覆盖一半
          ctx.fillPath();
          ctx.strokePath();

          // --- 2. 绘制一个半透明的蓝色矩形 ---
          // 对于矩形，有更简单的直接绘制方法
          ctx.fillColor = $color("blue");
          ctx.setAlpha(0.5); // 设置50%的透明度
          ctx.fillRect($rect(150, 20, 100, 80)); // 直接填充矩形
          ctx.setAlpha(1.0); // 恢复透明度，以免影响后续绘图

          // --- 3. 绘制文字 ---
          ctx.drawText(
            $rect(20, 150, 300, 50),
            "Hello, Canvas!",
            {
              font: $font("bold", 24),
              color: $color("darkGray"),
              alignment: $align.center // 居中对齐
            }
          );

          // --- 4. 绘制一条旋转的线条 (使用状态保存) ---
          ctx.saveGState(); // 保存当前状态（颜色、坐标系等）
          
          ctx.strokeColor = $color("green");
          ctx.setLineWidth(5);
          
          // 将坐标系原点平移到线条的旋转中心
          ctx.translateCTM(width / 2, height / 2);
          // 旋转坐标系45度（PI/4弧度）
          ctx.rotateCTM(Math.PI / 4);
          
          // 在新的坐标系下，我们只需画一条简单的水平线
          ctx.moveToPoint(-50, 0);
          ctx.addLineToPoint(50, 0);
          ctx.strokePath();
          
          ctx.restoreGState(); // 恢复到旋转之前的状态
        }
      }
    }
  ]
});
```


当您运行这段代码时，会看到一个模拟夜空的界面。

```
$ui.render({
  props: {
    title: "夜空 Canvas 示例 (最终修复颜色透明度)"
  },
  views: [
    {
      type: "canvas",
      layout: $layout.fill,
      events: {
        draw: function(view, ctx) {
          // 获取画布尺寸
          const width = view.frame.width;
          const height = view.frame.height;

          // =====================================
          // 绘制过程第一步：绘制夜空背景 (模拟径向渐变)
          // 目的：为整个场景创建一个深邃的渐变背景
          // =====================================
          console.log("Canvas 绘制步骤 1: 模拟绘制夜空径向渐变背景..."); // 控制台输出

          // --- 模拟渐变开始 ---
          const steps = 80; // 绘制的同心圆数量，数量越多，渐变越平滑，性能消耗越大
          const center = $point(width / 2, height / 2);
          const maxRadius = Math.max(width, height) / 2; // 确保覆盖整个画布的最大半径

          // 定义渐变的起始和结束颜色 (RGB)
          // #0F2027 (深蓝) => R:15, G:32, B:39
          // #203A43 (中间过渡) => R:32, G:58, B:67
          // #2C5364 (边缘更深) => R:44, G:83, B:100

          const startR = 15, startG = 32, startB = 39; // 最中心颜色
          const midR = 32, midG = 58, midB = 67;       // 中间颜色
          const endR = 44, endG = 83, endB = 100;       // 最边缘颜色

          // 从最外层圆开始向内绘制，以确保颜色覆盖正确
          for (let i = steps; i >= 0; i--) {
            const currentRadius = (i / steps) * maxRadius; // 当前圆的半径
            const t = i / steps; // 颜色插值因子，0表示最中心颜色，1表示最边缘颜色

            let r, g, b;
            if (t <= 0.5) { // 从中心到中间
                const t0_5 = t / 0.5; // 映射到 0-1 范围
                r = startR + (midR - startR) * t0_5;
                g = startG + (midG - startG) * t0_5;
                b = startB + (midB - startB) * t0_5;
            } else { // 从中间到边缘
                const t0_5 = (t - 0.5) / 0.5; // 映射到 0-1 范围
                r = midR + (endR - midR) * t0_5;
                g = midG + (endG - midG) * t0_5;
                b = midB + (endB - midB) * t0_5;
            }

            // --- 修复开始 ---
            // 直接在 $color 构造函数中指定 alpha
            ctx.fillColor = $color(r / 255.0, g / 255.0, b / 255.0, 1.0); // RGB值转换为0-1范围
            // --- 修复结束 ---

            ctx.beginPath();
            ctx.addArc(center.x, center.y, currentRadius, 0, Math.PI * 2, true);
            ctx.closePath();
            ctx.fillPath();
          }
          // --- 模拟渐变结束 ---

          // =====================================
          // 绘制过程第二步：绘制月亮
          // 目的：在背景上添加一个主要的天体
          // =====================================
          console.log("Canvas 绘制步骤 2: 绘制月亮 (带阴影)..."); // 控制台输出
          const moonRadius = 40;
          const moonX = width - moonRadius - 30; // 靠右边
          const moonY = moonRadius + 30; // 靠上边

          // 设置月亮阴影
          // --- 修复开始 ---
          ctx.setShadow(
            $size(5, 5), // 阴影偏移量 (X, Y)
            10, // 阴影模糊半径
            $color(0, 0, 0, 0.6) // 直接使用 $color(r, g, b, a) 来创建带透明度的黑色
          );
          // --- 修复结束 ---

          ctx.fillColor = $color("white"); // 月亮颜色
          ctx.beginPath();
          ctx.addArc(moonX, moonY, moonRadius, 0, Math.PI * 2, true);
          ctx.closePath();
          ctx.fillPath();

          // 清除阴影，以免影响后续绘制
          ctx.setShadow($size(0, 0), 0, $color("clear"));

          // =====================================
          // 绘制过程第三步：绘制随机星星
          // 目的：增加夜空的细节和真实感
          // =====================================
          console.log("Canvas 绘制步骤 3: 绘制随机分布的星星..."); // 控制台输出
          const numberOfStars = 100; // 星星的数量
          const stars = []; // 用于存储星星位置，后续用于绘制星座

          for (let i = 0; i < numberOfStars; i++) {
            const starX = Math.random() * width;
            const starY = Math.random() * height;
            const starRadius = Math.random() * 1.5 + 0.5; // 星星大小 0.5 到 2.0
            const starAlpha = Math.random() * 0.7 + 0.3; // 透明度 0.3 到 1.0

            // --- 修复开始 ---
            ctx.fillColor = $color(1, 1, 1, starAlpha); // 直接使用 $color(r, g, b, a) 创建带透明度的白色
            // --- 修复结束 ---

            ctx.beginPath();
            ctx.addArc(starX, starY, starRadius, 0, Math.PI * 2, true);
            ctx.closePath();
            ctx.fillPath();

            // 存储一些星星的位置，用于后续绘制星座
            // 避免数组过大，仅存储约 10% 的星星
            if (Math.random() < 0.1) {
              stars.push({ x: starX, y: starY });
            }
          }

          // =====================================
          // 绘制过程第四步：绘制一个简单的星座和名称
          // 目的：展示路径连接和文字绘制，并使用状态保存/恢复
          // =====================================
          console.log("Canvas 绘制步骤 4: 绘制一个简单星座和名称..."); // 控制台输出
          // 确保有足够的星星来形成星座，至少需要4颗星来连接
          if (stars.length >= 4) {
            // 定义构成星座的星星，随机选取4颗
            const constellationStars = [];
            while(constellationStars.length < 4) {
                const randomIndex = Math.floor(Math.random() * stars.length);
                const star = stars[randomIndex];
                // 确保选取的星星不重复，并且不在月亮附近（避免重叠）
                const distanceToMoon = Math.sqrt(Math.pow(star.x - moonX, 2) + Math.pow(star.y - moonY, 2));
                if (star && !constellationStars.includes(star) && distanceToMoon > moonRadius + 50) { // 增加 star 存在性检查
                    constellationStars.push(star);
                }
            }
            
            // 如果最终选取的星星不足4颗，则跳过星座绘制
            if (constellationStars.length < 4) {
                console.log("   - 星星数量不足或附近无合适位置，跳过星座绘制。");
                return; // 退出绘制，不强制绘制星座
            }

            // 保存当前绘图状态，以便后续恢复，避免线条颜色、宽度影响全局
            ctx.saveGState();
            console.log("   - 保存当前 Canvas 状态."); // 控制台输出

            ctx.strokeColor = $color(0, 1, 1, 0.7); // 青色，直接指定透明度
            ctx.setLineWidth(1.5); // 连接线宽度

            // 绘制连接线
            ctx.beginPath();
            ctx.moveToPoint(constellationStars[0].x, constellationStars[0].y);
            ctx.addLineToPoint(constellationStars[1].x, constellationStars[1].y);
            ctx.addLineToPoint(constellationStars[2].x, constellationStars[2].y);
            ctx.addLineToPoint(constellationStars[3].x, constellationStars[3].y);
            ctx.strokePath();
            console.log("   - 绘制星座连接线."); // 控制台输出

            // 绘制星座名称
            const constellationName = "小狗座"; // 假设的星座名称
            // 文本位置可以基于第一颗星，稍微调整一下
            const textX = constellationStars[0].x + 10;
            const textY = constellationStars[0].y - 20;

            ctx.drawText(
              $rect(textX, textY, 120, 30), // 文本绘制区域
              constellationName,
              {
                font: $font("italic", 16), // 斜体字体
                color: $color("yellow"), // 文本颜色 (这里黄色可以直接用字符串，因为没有透明度需求)
                alignment: $align.left // 左对齐
              }
            );
            console.log("   - 绘制星座名称: '" + constellationName + "'."); // 控制台输出

            // 恢复之前保存的绘图状态，不受星座绘制的影响
            ctx.restoreGState();
            console.log("   - 恢复 Canvas 状态."); // 控制台输出
          } else {
              console.log("   - 星星数量不足，跳过星座绘制。");
          }

          console.log("Canvas 绘制完成！"); // 控制台输出
        }
      }
    }
  ]
});
```

`canvas` 为实现高度自定义的视觉效果提供了无限可能，但需要你对坐标、路径和状态有清晰的认识。对于更复杂的操作，建议参考 Apple 的 Core Graphics 文档。 
