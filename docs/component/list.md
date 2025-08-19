# type: "list"

`list` 用于创建一个列表，是一个稍微有一点点复杂的结构（其实很复杂），一个最简单的 list 可以是这样：

```js
{
  type: "list",
  props: {
    data: ["JavaScript", "Swift"]
  },
  layout: $layout.fill,
}
```

这会创建一个只有一个 `section` 的，每一行都是文字的列表。

如果你需要创建多个 section，只需要把 `data` 字段变成一个这样的结构：

```js
{
  type: "list",
  props: {
    data: [
      {
        title: "Section 0",
        rows: ["0-0", "0-1", "0-2"]
      },
      {
        title: "Section 1",
        rows: ["1-0", "1-1", "1-2"]
      }
    ]
  },
  layout: $layout.fill,
}
```

这会创建两个 section，每个 section 有三行，用什么样的形式取决于你想要的风格。

以上的每一行数据都是简单的一个 string，实际上 list 的每一行都是可以自定义的，我们称为 `template`（在 props 里）：

```js
template: {
  props: {
    bgcolor: $color("clear")
  },
  views: [
    {
      type: "label",
      props: {
        id: "label",
        bgcolor: $color("#474b51"),
        textColor: $color("#abb2bf"),
        align: $align.center,
        font: $font(32)
      },
      layout: $layout.fill
    }
  ]
}
```

你已经发现了， template 的描述和定义 view 没有任何区别，只不过 JSBox 会用它来渲染 list 的每一行视图，当然这样的话 data 的内容也就不仅仅是一个 string 了：

```js
data: [
  {
    label: {
      text: "Hello"
    }
  },
  {
    label: {
      text: "World"
    }
  }
]
```

事情是这样发生的，data 里面的每个结构对应一行数据，使用 `id: object` 的方式进行映射，例如上面的 `label`，映射的是 label 这种控件可以支持的全部 `props`，上面的 text 就是设置他的文本内容。

有了 template 这样的构造和 data 映射机制，基本上我们可以构造任何我们想要的列表项，不要忘了，即便是在 template 的机制下，也还是支持单 section 和多 section 两种构造方式。

# header & footer

`header` 和 `footer` 是放在头部和尾部的自定义 view，是可选项（在 props 里）：

```js
footer: {
  type: "label",
  props: {
    height: 20,
    text: "Write the Code. Change the world.",
    textColor: $color("#AAAAAA"),
    align: $align.center,
    font: $font(12)
  }
}
```

请在 props 里面指定他的高度 `height`。

# 静态 cells

在 list 里面你也许希望有些行是固定的，不希望以 template 来初始化并进行复用，这是可以的，而且很容易做到。

唯一需要做的是，在设置 `data` 的时候，把某一行的数据替换成一个 view 的定义，他可以和其他的数据混合在一起：

```js
data: [
  {
    title: "System (Text)",
    rows: [
      {
        type: "button",
        props: {
          title: "Button",
          selectable: false
        },
        layout: function(make, view) {
          make.center.equalTo(view.super)
          make.width.equalTo(64)
        },
        events: {
          tapped: function(sender) {
            $ui.toast("Tapped")
          }
        }
      }
    ]
  },
  {
    title: "System (Contact Add)",
    rows: [
      {
        type: "button",
        props: {
          type: 5
        },
        layout: $layout.center
      }
    ]
  }
]
```

其中 `selectable` 属性表示该行是否可以被选中，默认为 false。

# props

属性 | 类型 | 读写 | 说明
---|---|---|---
style | number | 只写 | 样式 0 ~ 2
data | object | 读写 | 数据源
separatorInset | $insets | 读写 | 分割线边距
separatorHidden | boolean | 读写 | 隐藏分割线
separatorColor | $color | 读写 | 分割线颜色
header | object | 读写 | header
footer | object | 读写 | footer
rowHeight | number | 只写 | 行高
autoRowHeight | boolean | 只写 | 是否自动行高
estimatedRowHeight | number | 只写 | 估算的行高
sectionTitleHeight | number | 只写 | section 标题高度
selectable | bool | 只写 | 行是否可被选中
stickyHeader | boolean | 只写 | section header 是否固定
reorder | boolean | 读写 | 是否可以长按排序
crossSections | boolean | 读写 | 长按排序时是否可以跨 section
hasActiveAction | boolean | 只读 | 是否正在使用 action

# props: actions

`actions` 用于定义滑动操作的按钮：

```js
actions: [
  {
    title: "delete",
    color: $color("gray"), // default to gray
    handler: function(sender, indexPath) {

    }
  },
  {
    title: "share",
    handler: function(sender, indexPath) {

    }
  }
]
```

可以通过 `swipeEnabled` 函数来决定某一行是否能被滑动：

```js
swipeEnabled: function(sender, indexPath) {
  return indexPath.row > 0;
}
```

以上代码决定了第一行不能被滑动。

# object($indexPath)

返回在 indexPath 位置的数据：

```js
const data = tableView.object($indexPath(0, 0));
```

# insert(object)

插入新的数据：

```js
// indexPath 和 index 可选其一，value 要符合 data 元素的定义
tableView.insert({
  indexPath: $indexPath(0, 0),
  value: "Hey!"
})
```

# delete(object)

删除数据：

```js
// object 可以是 indexPath 或者 index
tableView.delete(0)
```

# cell($indexPath)

返回在 indexPath 位置的视图：

```js
const cell = tableView.cell($indexPath(0, 0));
```

# events: rowHeight

`rowHeight` 可以动态地为每一行指定行高，会覆盖 props 里面那个静态值：

```js
rowHeight: function(sender, indexPath) {
  if (indexPath.row == 0) {
    return 88.0
  } else {
    return 44.0
  }
}
```

# events: sectionTitleHeight

`sectionTitleHeight` 可以动态地为每个 section 指定标题高度，会覆盖 props 里面那个静态值：

```js
sectionTitleHeight: (sender, section) => {
  if (section == 0) {
    return 30;
  } else {
    return 40;
  }
}
```

# events: didSelect

`didSelect` 在点击某一行时回调：

```js
didSelect: function(sender, indexPath, data) {

}
```

# events: didLongPress

`didLongPress` 在长按某一行时回调：

```js
didLongPress: function(sender, indexPath, data) {

}
```

# events: forEachItem

按照顺序获得列表的每一项：

```js
forEachItem: function(view, indexPath) {
  
}
```

# 自动高度

很多时候我们需要动态计算行高，例如每行的文本可能不一样，但我们希望全部展示出来。

从 v2.5.0 开始，list view 支持自动高度，只需指定 `autoRowHeight` 和 `estimatedRowHeight`，并设置好相关约束：

```js
const sentences = [
  "Although moreover mistaken kindness me feelings do be marianne.",
  "Effects present letters inquiry no an removed or friends. Desire behind latter me though in. Supposing shameless am he engrossed up additions. My possible peculiar together to. Desire so better am cannot he up before points. Remember mistaken opinions it pleasure of debating. Court front maids forty if aware their at. Chicken use are pressed removed.",
  "He went such dare good mr fact. The small own seven saved man age ﻿no offer. Suspicion did mrs nor furniture smallness. Scale whole downs often leave not eat. An expression reasonably cultivated indulgence mr he surrounded instrument. Gentleman eat and consisted are pronounce distrusts.",
];

$ui.render({
  views: [
    {
      type: "list",
      props: {
        autoRowHeight: true,
        estimatedRowHeight: 44,
        template: [
          {
            type: "image",
            props: {
              id: "icon",
              symbol: "text.bubble"
            },
            layout: (make, view) => {
              make.left.equalTo(10);
              make.size.equalTo($size(24, 24));
              make.centerY.equalTo(view.super);
            }
          },
          {
            type: "label",
            props: {
              id: "label",
              lines: 0
            },
            layout: (make, view) => {
              const insets = $insets(10, 44, 10, 10);
              make.edges.equalTo(view.super).insets(insets);
            }
          }
        ],
        data: sentences.map(text => {
          return {
            "label": {text}
          }
        })
      },
      layout: $layout.fill
    }
  ]
});
```

简单的用来显示文本的列表，也支持自动高度：

```js
const sentences = [
  "Although moreover mistaken kindness me feelings do be marianne.",
  "Effects present letters inquiry no an removed or friends. Desire behind latter me though in. Supposing shameless am he engrossed up additions. My possible peculiar together to. Desire so better am cannot he up before points. Remember mistaken opinions it pleasure of debating. Court front maids forty if aware their at. Chicken use are pressed removed.",
  "He went such dare good mr fact. The small own seven saved man age ﻿no offer. Suspicion did mrs nor furniture smallness. Scale whole downs often leave not eat. An expression reasonably cultivated indulgence mr he surrounded instrument. Gentleman eat and consisted are pronounce distrusts.",
];

$ui.render({
  views: [
    {
      type: "list",
      props: {
        autoRowHeight: true,
        data: sentences
      },
      layout: $layout.fill
    }
  ]
});
```

# 长按排序

list 组件支持让用户通过长按来进行排序，需要实现以下内容：

首先需要在 `props` 里面设置 `reorder: true` 来打开这个功能：

```js
props: {
  reorder: true
}
```

其次，list 组件提供了三个方法来对用户排序的结果进行处理：

用户触发了长按排序操作：

```js
reorderBegan: function(indexPath) {

}
```

用户把一个列表项从 `fromIndex` 移动到了 `toIndex`:

```js
reorderMoved: function(fromIndexPath, toIndexPath) {
  // Reorder your data source here
}
```

用户结束了列表项的重新排序：

```js
reorderFinished: function(data) {
  // Save your data source here
}
```

可以通过 `canMoveItem` 来决定是否能被移动：

```js
canMoveItem: function(sender, indexPath) {
  return indexPath.row > 0;
}
```

上述代码将会使得除了第一个以外的内容都可以被长按排序。

简单说，我们通常需要在 `reorderMoved` 和 `reorderFinished` 里面对 `data` 进行一些处理。

请参考这个例子获得更多信息：https://github.com/cyanzhong/xTeko/blob/3ac0d3f5ac552a1c72ea39d0bd1099fd02f5ca70/extension-scripts/xteko-menu.js

# 分页加载

先看一个例子：

```js
let data = ["1", "2", "3", "4", "5", "6", "7", "8", "9", "10", "11", "12", "13", "14", "15"];
$ui.render({
  views: [
    {
      type: "list",
      props: { data },
      layout: $layout.fill,
      events: {
        didReachBottom: function(sender) {
          $ui.toast("fetching...")
          $delay(1.5, () => {
            sender.endFetchingMore()
            data = data.concat(data.map(item => {
              return (parseInt(item) + 15).toString()
            }))
            $("list").data = data
          })
        }
      }
    }
  ]
})
```

当用户将 list 滚动到底部的时候，会触发 `didReachBottom` 方法，在这里加载你的下一页数据。

与此同时，用户会看到界面处于加载中的一个状态，此举是避免用户再次触发一次 `didReachBottom`。

当你数据加载完成之后，先调用 `endFetchingMore` 方法结束这个状态，然后对数据源进行更新。

最后，将数据源更新到 list 的 data 属性，即将新的内容显示到列表上。

# setEditing

结束 list 划出的状态：

```js
$("list").setEditing(false)
```

# scrollTo

滚动到某行：

```js
$("list").scrollTo({
  indexPath: $indexPath(0, 0),
  animated: true // 默认为 true
})
```

# 最后

此外，`list` 也是继承自 `scroll`，也会拥有 scroll 有的全部事件回调。

可见 list 并不简单，可以参考 `uikit-list.js` 和 `uikit-catalog.js` 获得更多信息。

---

## 文件内容解读与示例

### 组件用途

`list`（列表）是 JSBox 中功能最强大、用途最广泛的组件，没有之一。它是所有数据驱动界面的基石，用于展示垂直滚动的、可长可短的数据集合。从简单的设置菜单，到复杂的联系人列表、新闻 Feed 流，其核心都是 `list` 组件。虽然初看复杂，但一旦掌握，便能构建出功能完善的应用级界面。

### 核心概念：模板与数据绑定

要精通 `list`，就必须理解其核心设计思想：**视图模板（template）与数据源（data）的分离与绑定**。

1.  **模板 (`template`)**: 
    - 它是一个**视图蓝图**，定义了列表中**每一行长什么样**。
    - 你可以在 `template` 中像设计普通 `view` 一样，放入 `label`, `image`, `button` 等各种组件，并为它们设置 `id`。
    - 这个模板只会被创建一次，然后被高效地复用（dequeue）来显示所有行，这保证了即使有成千上万行数据，列表也能流畅滚动。

2.  **数据源 (`data`)**: 
    - 它是一个**纯数据数组**，负责为模板提供“填充物”。
    - 数组中的每一个对象都对应列表中的一行。
    - 对象中的 `key` 必须与 `template` 中视图的 `id` 相对应，`value` 则是一个 `props` 对象，用于设置该视图的属性。

**绑定过程解析**：

想象一下流水线作业：`template` 是一个模具，`data` 数组是原材料。列表滚动的过程中，系统不断地从 `data` 中取出一条数据，然后用这条数据去“填充”模具，生成一行最终的视图。

```javascript
// 模板：定义了行内有哪些控件，以及它们的 ID
template: {
  views: [
    { type: "image", props: { id: "avatar" } },
    { type: "label", props: { id: "name" } }
  ]
}

// 数据：为模板中的控件提供具体内容
data: [
  {
    avatar: { src: "url_to_image_1" }, // 填充 id="avatar" 的 image
    name: { text: "张三" } // 填充 id="name" 的 label
  },
  {
    avatar: { src: "url_to_image_2" },
    name: { text: "李四" }
  }
]
```

### 关键特性一览

- **行选择 (`didSelect`)**: 用户点击某一行时触发，是列表最重要的交互事件。回调函数会直接返回该行的 `data` 和 `indexPath`（位置索引），极其方便。
- **自动行高 (`autoRowHeight`)**: 设置为 `true` 并为模板内视图添加好约束后，列表可以自动计算并适配每行的高度，完美支持显示长度不一的文本内容。
- **下拉刷新与上拉加载**: 分别通过 `pulled`（继承自 `scroll`）和 `didReachBottom` 事件实现。这是构建信息流（Feeds）应用的标配功能。
- **左滑操作 (`actions`)**: 通过定义 `actions` 数组，可以轻松为每一行添加“删除”、“分享”等滑动操作按钮。
- **长按排序 (`reorder`)**: 设置 `reorder: true` 并实现相关事件，即可让用户通过长按拖拽来改变列表项的顺序。

### 示例代码：一个经典的联系人列表

这个示例将创建一个包含头像、姓名和邮箱的自定义列表，并响应用户的点击事件。

```javascript
// 1. 定义列表模板
const contactTemplate = {
  props: {
    // 设置行高
    height: 70
  },
  views: [
    // 头像
    {
      type: "image",
      props: {
        id: "avatar",
        radius: 25 // 圆角
      },
      layout: (make, view) => {
        make.left.top.bottom.inset(10);
        make.width.equalTo(view.height);
      }
    },
    // 姓名
    {
      type: "label",
      props: {
        id: "name",
        font: $font("bold", 17)
      },
      layout: (make, view) => {
        make.left.equalTo($("avatar").right).offset(10);
        make.top.inset(12);
      }
    },
    // 邮箱
    {
      type: "label",
      props: {
        id: "email",
        font: $font(14),
        textColor: $color("gray")
      },
      layout: (make, view) => {
        make.left.equalTo($("name"));
        make.bottom.inset(12);
      }
    }
  ]
};

// 2. 准备数据源
const contactData = [
  {
    avatar: { src: "https://randomuser.me/api/portraits/men/32.jpg" },
    name: { text: "John Appleseed" },
    email: { text: "john.appleseed@icloud.com" }
  },
  {
    avatar: { src: "https://randomuser.me/api/portraits/women/44.jpg" },
    name: { text: "Kate Bell" },
    email: { text: "kate.bell@jsbox.com" }
  },
  {
    avatar: { src: "https://randomuser.me/api/portraits/men/42.jpg" },
    name: { text: "Daniel Higgins Jr." },
    email: { text: "daniel.higgins@work.com" }
  }
];

// 3. 渲染列表
$ui.render({
  props: { title: "联系人" },
  views: [
    {
      type: "list",
      props: {
        template: contactTemplate,
        data: contactData
      },
      layout: $layout.fill,
      events: {
        didSelect: (sender, indexPath, data) => {
          // 点击事件，data 参数就是上面 contactData 数组中的一项
          const contactName = data.name.text;
          $ui.alert(`你选择了 ${contactName}`);
        }
      }
    }
  ]
});
```

**代码解读**：

1.  我们首先定义了 `contactTemplate`，它描述了一行联系人的外观：左边是 `id: "avatar"` 的图片，右边是 `id: "name"` 和 `id: "email"` 的两个标签。
2.  然后，我们创建了 `contactData` 数组。数组中每个对象的 `key` (`avatar`, `name`, `email`) 都与模板中视图的 `id` 完美对应。
3.  最后，在 `list` 的 `props` 中，我们将 `template` 和 `data` 传入。JSBox 会自动完成剩下的所有绑定工作。
4.  在 `didSelect` 事件中，我们可以直接使用 `data` 参数，它就是被点击行对应的数据源对象，从中取值非常方便。

`list` 组件是 JSBox UI 的核心，虽然初学时概念较多，但一旦你掌握了“模板-数据”分离绑定的思想，就能用它来构建任何数据驱动的复杂界面。 
