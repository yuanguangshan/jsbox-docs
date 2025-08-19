# type: "gallery"

`gallery` 用于创建一个横向滚动的分页器，可以滚动显示：

```js
{
  type: "gallery",
  props: {
    items: [
      {
        type: "image",
        props: {
          src: "https://images.apple.com/v/iphone/home/v/images/home/limited_edition/iphone_7_product_red_large_2x.jpg"
        }
      },
      {
        type: "image",
        props: {
          src: "https://images.apple.com/v/iphone/home/v/images/home/airpods_large_2x.jpg"
        }
      },
      {
        type: "image",
        props: {
          src: "https://images.apple.com/v/iphone/home/v/images/home/apple_pay_large_2x.jpg"
        }
      }
    ],
    interval: 3,
    radius: 5.0
  },
  layout: function(make, view) {
    make.left.right.inset(10)
    make.centerY.equalTo(view.super)
    make.height.equalTo(320)
  }
}
```

创建一个有三个分页的视图，显示三张图片。

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
items | object | 只写 | 每个分页项
page | number | 读写 | 当前页数
interval | number | 读写 | 自动播放间隔，为 0 表示不播放
pageControl | $view | 只读 | 页面控制组件

# events

页面改变时将会调用 changed:

```js
changed: function(sender) {
  
}
```

# 获取子 view

在 gallery 里面获取子 view 请使用以下方法：

```js
const views = $("gallery").itemViews; // All views
const view = $("gallery").viewWithIndex(0); // The first view
```

# 滚动到某一页

设置 `page` 将直接切换到某一页，如果需要带着动画滚动过去，请使用：

```js
$("gallery").scrollToPage(index);
```

---

## 文件内容解读与示例

### 组件用途

`gallery` 是一个专门用于展示分页内容的容器视图，它以水平方向滚动，并自带页面指示器（小圆点）。它非常适合用于创建图片轮播、应用的首次启动引导页、或者任何需要在多个页面之间进行切换展示的场景。

### 核心概念

1.  **`items` 数组**: 这是 `gallery` 最核心的属性。它是一个数组，数组中的**每一个元素都是一个完整的视图定义对象**。`gallery` 会为数组中的每个元素创建一个独立的、可滑动的页面。这极其强大，意味着每个页面不仅可以是一张图片，更可以是一个包含文字、按钮等多种组件的复杂布局。

2.  **自动轮播 (`interval`)**: 通过设置 `interval` 属性为一个大于 0 的数字（例如 `3`），`gallery` 就会变身为一个自动轮播的广告牌，每隔 3 秒自动滚动到下一页。如果 `interval` 为 0 或不设置，则需要用户手动滑动。

3.  **页面控制 (`page` 和 `pageControl`)**: 
    - `page` 属性可读可写，你可以通过它获取当前是第几页（从0开始），也可以通过设置它来立即跳转到指定页面。
    - `pageControl` 属性是只读的，它指向底部的页面指示器（小圆点）视图。你可以通过它来修改指示器的颜色，例如 `$("galleryId").pageControl.tintColor = $color("gray")`。

### 交互与控制

- **`changed` 事件**: 当页面因为用户滑动或自动轮播而发生改变时，此事件会被触发。你可以通过 `sender.page` 来获取新的页面索引。
- **`scrollToPage(index)` 方法**: 与直接设置 `page` 属性的瞬时跳转不同，这个方法会以平滑的动画效果滚动到指定页面，用户体验更好。

### 示例代码：产品特性引导页

下面的示例将创建一个包含图片、标题和描述的引导页，并演示如何通过外部按钮来控制其滚动。

```javascript
// 准备我们的数据
const features = [
  {
    title: "快如闪电",
    description: "全新的 A15 仿生芯片带来极致性能。",
    image: "https://www.apple.com.cn/v/iphone-13/g/images/overview/cpu/cpu_hero__b6bllj2klt2y_large.jpg"
  },
  {
    title: "超长续航",
    description: "电池续航迎来巨大飞跃，满足全天所需。",
    image: "https://www.apple.com.cn/v/iphone-13/g/images/overview/battery/battery_hero__t2b4jwisaq6y_large.jpg"
  },
  {
    title: "电影效果模式",
    description: "轻松拍摄具有景深效果的电影感视频。",
    image: "https://www.apple.com.cn/v/iphone-13/g/images/overview/cinematic_mode/cinematic_hero__d5w4z0njbeqa_large.jpg"
  }
];

$ui.render({
  props: {
    title: "Gallery 组件示例"
  },
  views: [
    {
      type: "gallery",
      props: {
        id: "feature-gallery",
        // 使用 map 将数据转换为 gallery 需要的 items 格式
        items: features.map(feature => {
          // 每个 item 是一个完整的视图定义
          return {
            type: "view", // 使用一个 view 作为容器
            views: [
              { type: "image", props: { src: feature.image, contentMode: $contentMode.scaleAspectFill, clipped: true } },
              { type: "label", props: { text: feature.title, font: $font("bold", 24), align: $align.center } },
              { type: "label", props: { text: feature.description, font: $font(16), textColor: $color("gray"), align: $align.center, lines: 0 } }
            ]
          };
        }),
        interval: 0, // 关闭自动轮播
        radius: 10
      },
      layout: function(make, view) {
        make.top.left.right.inset(15);
        make.height.equalTo(400);
      },
      events: {
        changed: function(sender) {
          console.log(`当前页面: ${sender.page}`);
        }
      }
    },
    {
      // 控制按钮
      type: "button",
      props: { title: "下一页" },
      layout: function(make, view) {
        make.top.equalTo($("feature-gallery").bottom).offset(20);
        make.centerX.equalTo(view.super);
      },
      events: {
        tapped: function(sender) {
          const gallery = $("feature-gallery");
          const nextPage = (gallery.page + 1) % gallery.itemViews.length;
          gallery.scrollToPage(nextPage);
        }
      }
    }
  ]
});
```

**代码解读**：

1.  **数据与视图分离**：我们先定义了一个 `features` 数组来存放数据，然后使用 `map` 函数动态生成 `gallery` 的 `items`。这是一个非常好的实践，让代码更清晰、更易于维护。
2.  **复杂的页面内容**：`items` 数组的每个元素不再是简单的 `image`，而是一个包含图片和两个标签的 `view` 容器，展示了 `gallery` 的灵活性。
3.  **外部控制**：我们添加了一个“下一页”按钮。在它的 `tapped` 事件中，我们获取了 `gallery` 的当前页码，计算出下一页的索引，并调用 `scrollToPage` 方法来实现平滑滚动，演示了如何从外部对 `gallery` 进行控制。
