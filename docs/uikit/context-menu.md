# 长按菜单

你可以为任意的 view 提供长按菜单（[Context Menus](https://developer.apple.com/design/human-interface-guidelines/ios/controls/context-menus/)），支持子菜单和 [SF Symbols](https://developer.apple.com/design/human-interface-guidelines/sf-symbols/overview/)。

下面是一个最简单的样例：

```js
$ui.render({
  views: [
    {
      type: "button",
      props: {
        title: "Long Press!",
        menu: {
          title: "Context Menu",
          items: [
            {
              title: "Title",
              handler: sender => {}
            }
          ]
        }
      },
      layout: (make, view) => {
        make.center.equalTo(view.super);
        make.size.equalTo($size(120, 36));
      }
    }
  ]
});
```

# SF Symbols

除了显示标题，也可以用 `symbol` 属性指定图标，例如：

```js
{
  title: "Title",
  symbol: "paperplane",
  handler: sender => {}
}
```

查询 SF Symbols 相关的名字，你可以使用 Apple 官方提供的应用：https://developer.apple.com/design/downloads/SF-Symbols.dmg 或是这个 JSBox 脚本：https://xteko.com/install?id=141&lang=zh-Hans

# destructive

可以将标题显示成红色来表示这是个危险动作，提醒用户注意：

```js
{
  title: "Title",
  destructive: true,
  handler: sender => {}
}
```

# 子菜单

当一个菜单项包含 `items` 时，会被收纳为一个子菜单：

```js
{
  title: "Sub Menu",
  items: [
    {
      title: "Item 1",
      handler: sender => {}
    },
    {
      title: "Item 2",
      handler: sender => {}
    }
  ]
}
```

这个子菜单点击之后会显示两个二级选项，子菜单可以多层嵌套。

# Inline 子菜单

上述子菜单将会把菜单项作为二级菜单显示，也可以将其直接显示到当前菜单，用分割线隔开，只需设置 `inline` 属性：

```js
{
  title: "Sub Menu",
  inline: true,
  items: [
    {
      title: "Item",
      handler: sender => {}
    }
  ]
}
```

# list 和 matrix

可以为 `list` 和 `matrix` 的某一行或某一个元素提供长按菜单，在这种情况下 `handler` 将会携带 `indexPath` 信息：

```js
$ui.render({
  views: [
    {
      type: "list",
      props: {
        data: ["A", "B", "C"],
        menu: {
          title: "Context Menu",
          items: [
            {
              title: "Action 1",
              handler: (sender, indexPath) => {}
            }
          ]
        }
      },
      layout: $layout.fill
    }
  ]
});
```

这种场景类似于其 `event: didSelect`，只是触发方式是通过长按某一个元素。

# Pull-Down 菜单

作为 iOS 14 的主要改进之一，您可以为 `button` 和 `navButtons` 提供更现代化的 [Pull-Down](https://developer.apple.com/design/human-interface-guidelines/ios/controls/buttons/) 菜单选项。`Pull-Down` 菜单不会将背景模糊，并可以作为主要菜单触发，无需长按。

为 `button` 类型支持 Pull-Down 菜单，仅需在上述配置中增加 `pullDown: true` 参数：

```js
$ui.render({
  views: [
    {
      type: "button",
      props: {
        title: "Long Press!",
        menu: {
          title: "Context Menu",
          pullDown: true,
          asPrimary: true,
          items: [
            {
              title: "Title",
              handler: sender => {}
            }
          ]
        }
      },
      layout: (make, view) => {
        make.center.equalTo(view.super);
        make.size.equalTo($size(120, 36));
      }
    }
  ]
});
```

为 `navButtons` 支持 Pull-Down 菜单也类似，仅需使用 `menu` 参数：

```js
$ui.render({
  props: {
    navButtons: [
      {
        title: "Title",
        symbol: "checkmark.seal",
        menu: {
          title: "Context Menu",
          asPrimary: true,
          items: [
            {
              title: "Title",
              handler: sender => {}
            }
          ]
        }
      }
    ]
  }
});
```

上述参数中，`asPrimary` 表示是否作为一级操作，也即短按触发。

---

## 文件内容解读与示例

### 用途说明

本文档介绍了如何在 JSBox 中为视图（View）添加上下文菜单。主要涵盖两种现代 iOS 菜单样式：

1.  **上下文菜单 (Context Menus)**: 通过**长按**一个视图触发，通常会模糊背景并显示一个与其内容相关的操作列表。
2.  **下拉菜单 (Pull-Down Menus)**: 主要用于**按钮**，通过**短按或长按**触发，以更轻量的方式在按钮下方或旁边显示一个菜单，不会模糊背景。

这些菜单是增强用户交互、保持界面整洁的有效方式，可以将多个操作收纳到一个控件中。

### 核心概念：`menu` 属性

无论是哪种菜单，其核心都是通过在视图的 `props` 中定义一个 `menu` 对象来实现的。这个 `menu` 对象的结构决定了菜单的标题、外观和行为。

#### `menu` 对象基本结构

```javascript
menu: {
  title: "菜单标题", // 可选，显示在菜单顶部的标题
  items: [ /* 菜单项数组 */ ]
}
```

#### 菜单项（Item）结构

`items` 数组中的每个对象代表一个可点击的菜单项。

```javascript
{
  title: "操作名称",       // 必须，显示给用户的文本
  symbol: "heart.fill", // 可选，使用 SF Symbol 图标名
  destructive: false,  // 可选，true 会将文本显示为红色，表示危险操作
  handler: (sender) => { /* 点击后执行的代码 */ },
  items: [ /* 子菜单项 */ ], // 可选，用于创建子菜单
  inline: false,       // 可选，用于内联子菜单
}
```

### 功能详解

-   **图标 (`symbol`)**: 使用 Apple 的 [SF Symbols](https://developer.apple.com/design/human-interface-guidelines/sf-symbols/overview/) 图标库，只需提供图标名称即可在菜单项标题前显示一个精美的图标。
-   **危险动作 (`destructive`)**: 将 `destructive: true` 设置给一个菜单项，可以向用户传达该操作具有破坏性（如“删除”），应谨慎使用。
-   **子菜单 (`items`)**: 在一个菜单项中嵌套 `items` 数组，即可创建层级菜单。点击该项会展开一个新的子菜单。
-   **内联子菜单 (`inline`)**: 设置 `inline: true` 的子菜单，其 `items` 不会作为下一级菜单弹出，而是直接在当前菜单中展开，并由分割线隔开，适合组织相关的操作组。
-   **列表/矩阵中的菜单 (`list`/`matrix`)**: 当为 `list` 或 `matrix` 设置 `menu` 时，长按其中的某一项会触发菜单。此时，`handler` 函数会额外接收一个 `indexPath` 参数，让你能够精确地知道用户操作的是哪一行/哪一项。

### Pull-Down 菜单 vs. Context Menu

-   **触发方式**: Context Menu 只能长按；Pull-Down Menu 可通过短按或长按触发。
-   **视觉效果**: Context Menu 会模糊背景，聚焦于菜单；Pull-Down Menu 更像是从按钮上弹出的一个轻量浮层。
-   **配置**: 
    -   要将一个 `button` 的 `menu` 变为 Pull-Down 风格，只需在 `menu` 对象中添加 `pullDown: true`。
    -   `asPrimary: true` 属性让按钮的**短按**行为也变成显示菜单。如果为 `false`，则短按执行按钮的 `tapped` 事件，长按才显示菜单。
    -   导航栏按钮 (`navButtons`) 的 `menu` 默认就是 Pull-Down 风格。

### 示例：一个功能丰富的按钮菜单

```javascript
$ui.render({
  views: [
    {
      type: "button",
      props: {
        title: "操作",
        symbol: "ellipsis.circle",
        menu: {
          title: "文件操作",
          pullDown: true, // 使用 Pull-Down 风格
          asPrimary: true, // 短按即显示菜单
          items: [
            { title: "新建", symbol: "doc.badge.plus", handler: () => $ui.toast("新建") },
            { title: "打开", symbol: "folder", handler: () => $ui.toast("打开") },
            // 一个内联子菜单，用于组织分享选项
            {
              title: "分享",
              symbol: "square.and.arrow.up",
              inline: true,
              items: [
                { title: "分享链接", symbol: "link", handler: () => $ui.toast("分享链接") },
                { title: "分享文件", symbol: "doc", handler: () => $ui.toast("分享文件") }
              ]
            },
            // 一个危险操作
            { title: "删除", symbol: "trash", destructive: true, handler: () => $ui.toast("删除") }
          ]
        }
      },
      layout: (make, view) => {
        make.center.equalTo(view.super);
        make.size.equalTo($size(120, 44));
      }
    }
  ]
});
```

### 总结

通过 `menu` 属性，JSBox 提供了一种非常简洁且强大的方式来实现 iOS 系统原生的上下文菜单和下拉菜单。它支持图标、子菜单、危险状态等丰富的自定义选项，能帮助你构建出符合现代 iOS 设计规范、交互友好且功能强大的用户界面。