# type: "tab" & type: "menu"

提供两种方式用于显示一个菜单，当项目较少时，使用 `tab`，项目较多可以滚动时，使用 `menu`：

```js
$ui.render({
  views: [
    {
      type: "menu",
      props: {
        items: ["item 1", "item 2", "item 3", "item 4", "item 5", "item 6", "item 7", "item 8", "item 9"],
        dynamicWidth: true, // dynamic item width, default is false
      },
      layout: function(make) {
        make.left.top.right.equalTo(0)
        make.height.equalTo(44)
      },
      events: {
        changed: function(sender) {
          const items = sender.items;
          const index = sender.index;
          $ui.toast(`${index}: ${items[index]}`)
        }
      }
    }
  ]
})
```

上述代码生成一个菜单，并且处理用户点击的动作。

`sender.items` 表示所有项目，`sender.index` 表示被选中的 index（从 0 开始）。

`index` 属性也可以定义初始选中：

```js
props: {
  index: 1
}
```

默认选中第二个。

---

## 文件内容解读与示例

### 组件用途

`menu` 和 `tab` 组件都用于创建**分段控件（Segmented Control）**。这是一种由多个分段组成的水平条，用户每次只能选择其中一个。它通常被用作视图切换器（例如在不同功能页面间导航）或内容过滤器（例如筛选“全部”、“未读”、“已完成”）。

### `menu` 与 `tab` 的区别

选择哪个组件取决于你的选项数量：

- **`tab`**: 用于**少量、固定数量**的选项（通常为 2-5 个）。它的宽度是固定的，每个分段会自动平分总宽度。
- **`menu`**: 用于**较多数量**的选项。当所有选项的总宽度超过屏幕时，`menu` 组件可以**水平滚动**。

### 核心属性与事件

- **`items`**: 一个字符串数组，数组中的每个字符串都会成为菜单中的一个可点击的分段标题。
- **`index`**: 当前被选中的分段的索引（从 0 开始）。这个属性是可读写的。你可以在 `props` 中设置它来指定初始选中的项，也可以在 `changed` 事件中通过 `sender.index` 读取当前选中的项。
- **`changed` 事件**: 这是分段控件的灵魂。当用户点击并切换到一个新的分段时，这个事件就会被触发。所有的响应逻辑都应该写在这里。

### 示例代码：使用 `menu` 切换内容视图

下面的示例将创建一个顶部菜单，用户点击不同的菜单项时，下方的标签会显示对应的内容，这是一个非常典型的视图切换场景。

```javascript
const pages = ["首页", "消息", "发现", "我"];
const pageContent = [
  "这是首页的内容。欢迎使用我们的应用！",
  "你收到了 3 条新消息。",
  "发现页面有很多有趣的东西等你探索。",
  "这里是你的个人中心。"
];

$ui.render({
  props: {
    title: "Menu/Tab 示例"
  },
  views: [
    {
      // 1. 创建顶部的菜单栏
      type: "menu", // 如果选项少，可以换成 "tab"
      props: {
        id: "main-menu",
        items: pages,
        index: 0 // 默认选中第一项
      },
      layout: (make, view) => {
        make.top.left.right.inset(10);
        make.height.equalTo(32);
      },
      events: {
        changed: (sender) => {
          // 当菜单选项改变时，更新下方的标签内容
          const selectedIndex = sender.index;
          $("content-label").text = pageContent[selectedIndex];
        }
      }
    },
    {
      // 2. 用于显示内容的容器
      type: "view",
      props: {
        id: "content-container",
        bgcolor: $color("#F0F0F0"),
        radius: 10
      },
      layout: (make, view) => {
        make.top.equalTo($("main-menu").bottom).offset(10);
        make.left.right.bottom.inset(10);
      },
      views: [
        {
          // 3. 显示内容的标签
          type: "label",
          props: {
            id: "content-label",
            text: pageContent[0], // 初始显示第一页的内容
            font: $font(18),
            align: $align.center,
            lines: 0
          },
          layout: (make, view) => {
            make.center.equalTo(view.super);
            make.left.right.inset(20);
          }
        }
      ]
    }
  ]
});
```

**代码解读**：

1.  我们创建了一个 `menu` 控件，并用 `pages` 数组初始化了它的 `items`。`index: 0` 确保了应用启动时默认选中的是“首页”。
2.  下方是一个 `label`，用于显示与当前选中菜单项对应的内容。它的初始文本被设置为 `pageContent[0]`。
3.  **核心逻辑**在 `menu` 的 `changed` 事件中：
    - `const selectedIndex = sender.index;` 获取到用户新选中的是第几项。
    - `$("content-label").text = pageContent[selectedIndex];` 根据这个索引，我们从 `pageContent` 数组中取出对应的文本，并更新到下方的 `label` 上。

这个例子清晰地展示了 `menu` (或 `tab`) 作为“控制器”，去改变应用其他部分“状态”的基本模式。 
