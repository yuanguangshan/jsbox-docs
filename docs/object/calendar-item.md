# calendarItem

`calendarItem` 是使用 `$calendar` 接口时返回的数据对象。

属性 | 类型 | 读写 | 说明
---|---|---|---
title | string | 读写 | 标题
location | string | 读写 | 位置
notes | string | 读写 | 备注
url | string | 读写 | 链接
allDay | bool | 读写 | 是否全天时间
startDate | date | 读写 | 开始时间
endDate | date | 读写 | 结束时间
status | number | 只读 | 状态
eventIdentifier | string | 只读 | 标识符
creationDate | date | 只读 | 创建时间
modifiedDate | date | 只读 | 修改时间

---

## 文件内容解读与示例

### 用途说明

`calendarItem` 对象是 JSBox 中用于表示单个**日历事件**或**提醒事项**的数据结构。当你通过 `$calendar` API（例如，获取日历事件或创建新事件）时，你将与这种类型的对象进行交互。它封装了一个日历事件的所有关键信息，并且大多数属性都是可读写的，方便脚本对日历事件进行查询、创建和修改。

### 核心属性详解

`calendarItem` 包含了日历事件的各种详细信息：

#### 1. 基本信息 (可读写)

-   **`title`**: 事件的标题或名称。
-   **`location`**: 事件发生的地点。
-   **`notes`**: 事件的详细备注或描述。
-   **`url`**: 与事件相关的 URL 链接。

#### 2. 时间信息 (可读写)

-   **`allDay`**: 布尔值，表示事件是否为全天事件。如果为 `true`，`startDate` 和 `endDate` 的时间部分通常会被忽略。
-   **`startDate`**: 事件的开始时间，一个 JavaScript `Date` 对象。
-   **`endDate`**: 事件的结束时间，一个 JavaScript `Date` 对象。

#### 3. 元数据 (只读)

-   **`status`**: 事件的状态（例如，已确认、待定、已取消）。具体数值含义通常与原生 iOS 的 `EKEventStatus` 枚举对应。
-   **`eventIdentifier`**: 事件的唯一标识符。在日历数据库中，每个事件都有一个唯一的 ID。
-   **`creationDate`**: 事件的创建时间，一个 `Date` 对象。
-   **`modifiedDate`**: 事件的最后修改时间，一个 `Date` 对象。

### 示例代码：显示下一个日历事件的详情

下面的示例将模拟从日历中获取一个即将到来的事件，并将其详细信息显示在 UI 界面上。**请注意，实际使用 `$calendar` API 获取日历事件需要用户授权日历访问权限。**

```javascript
// 模拟一个从 $calendar API 获取到的 calendarItem 对象
// 实际应用中，你需要调用 $calendar.fetchEvents() 等方法来获取真实数据
const now = new Date();
const oneHourLater = new Date(now.getTime() + 60 * 60 * 1000);
const twoHoursLater = new Date(now.getTime() + 2 * 60 * 60 * 1000);

const sampleCalendarItem = {
  title: "JSBox 文档更新会议",
  location: "线上会议室 (Zoom)",
  notes: "讨论文档翻译、新功能添加和 Bug 修复。请提前阅读会议纪要。",
  url: "https://zoom.us/j/123456789",
  allDay: false,
  startDate: oneHourLater,
  endDate: twoHoursLater,
  status: 0, // 假设 0 代表已确认
  eventIdentifier: "mock-event-abc-123",
  creationDate: new Date(now.getTime() - 24 * 60 * 60 * 1000), // 昨天创建
  modifiedDate: now
};

$ui.render({
  props: { title: "日历事件详情" },
  views: [
    {
      type: "label",
      props: { text: `标题: ${sampleCalendarItem.title}`, font: $font("bold", 20) },
      layout: make => make.top.left.right.inset(10)
    },
    {
      type: "label",
      props: { text: `地点: ${sampleCalendarItem.location}` },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.right.inset(10)
    },
    {
      type: "label",
      props: { text: `开始时间: ${sampleCalendarItem.startDate.toLocaleString()}` },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.right.inset(10)
    },
    {
      type: "label",
      props: { text: `结束时间: ${sampleCalendarItem.endDate.toLocaleString()}` },
      layout: make => make.top.equalTo(view.prev.bottom).offset(5).left.right.inset(10)
    },
    {
      type: "label",
      props: { text: `全天事件: ${sampleCalendarItem.allDay ? "是" : "否"}` },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.right.inset(10)
    },
    {
      type: "label",
      props: { text: `备注: ${sampleCalendarItem.notes}`, lines: 0 },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.right.inset(10)
    },
    {
      type: "label",
      props: { text: `链接: ${sampleCalendarItem.url}`, textColor: $color("blue"), lines: 0 },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.right.inset(10)
    },
    {
      type: "label",
      props: { text: `创建时间: ${sampleCalendarItem.creationDate.toLocaleString()}` },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.right.inset(10)
    }
  ]
});
```

**代码解读**：

1.  我们首先创建了一个 `sampleCalendarItem` 对象，它模拟了从 `$calendar` API 获取到的一个日历事件数据。
2.  然后，我们使用多个 `label` 组件来显示 `sampleCalendarItem` 对象的各个属性。注意 `startDate` 和 `endDate` 是 `Date` 对象，可以直接使用 `toLocaleString()` 方法进行格式化显示。
3.  `notes` 和 `url` 属性的 `label` 设置了 `lines: 0`，以确保多行内容能够完整显示。

`calendarItem` 对象为 JSBox 脚本提供了与 iOS 日历事件进行交互的标准化数据模型，使得你可以方便地读取、创建和修改日历中的事件。 
