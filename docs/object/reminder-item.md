# reminderItem

`reminderItem` 是使用 `$reminder` 接口时返回的数据对象。

属性 | 类型 | 读写 | 说明
---|---|---|---
title | string | 读写 | 标题
location | string | 读写 | 位置
notes | string | 读写 | 备注
url | string | 读写 | 链接
priority | number | 读写 | 优先级
completed | bool | 读写 | 是否完成
completionDate | date | 只读 | 完成时间
alarmDate | date | 读写 | 提醒时间
creationDate | date | 只读 | 创建时间
modifiedDate | date | 只读 | 修改时间

---

## 文件内容解读与示例

### 用途说明

`reminderItem` 对象是 JSBox 中用于表示单个**提醒事项**的数据结构。当你通过 `$reminder` API（例如，获取提醒事项或创建新提醒）时，你将与这种类型的对象进行交互。它封装了提醒的所有详细信息，并且大多数属性都是可读写的，方便脚本对提醒事项进行查询、创建和修改。

### 核心概念：提醒事项的结构化表示

`reminderItem` 是一个标准化的数据模型，它与 iOS 原生的 `EKReminder` 对象相对应。这使得脚本能够以统一的方式处理从提醒事项中读取的提醒，或创建新的提醒。

### 核心属性详解

`reminderItem` 包含了提醒事项的各种详细信息：

#### 1. 基本信息 (可读写)

-   **`title`**: 提醒事项的标题或名称。
-   **`location`**: 与提醒事项相关的地点信息。
-   **`notes`**: 提醒事项的详细备注或描述。
-   **`url`**: 与提醒事项相关的 URL 链接。

#### 2. 状态与优先级 (可读写)

-   **`priority`**: 提醒事项的优先级，通常是一个数字（例如，0-9，数字越小优先级越高）。
-   **`completed`**: 布尔值，表示提醒事项是否已完成。设置为 `true` 会将提醒标记为完成。
-   **`completionDate`**: 只读，提醒事项完成的时间，一个 JavaScript `Date` 对象。只有当 `completed` 为 `true` 时才有意义。

#### 3. 时间信息 (可读写)

-   **`alarmDate`**: 提醒事项触发的时间，一个 JavaScript `Date` 对象。这是提醒实际会弹出的时间。

#### 4. 元数据 (只读)

-   **`creationDate`**: 提醒事项的创建时间，一个 `Date` 对象。
-   **`modifiedDate`**: 提醒事项的最后修改时间，一个 `Date` 对象。

### 示例代码：显示即将到来的提醒事项详情

下面的示例将模拟从提醒事项中获取一个即将到来的提醒，并将其详细信息显示在 UI 界面上。**请注意，实际使用 `$reminder` API 获取提醒事项需要用户授权提醒事项访问权限。**

```javascript
// 模拟一个从 $reminder API 获取到的 reminderItem 对象
// 实际应用中，你需要调用 $reminder.fetchReminders() 等方法来获取真实数据
const now = new Date();
const tomorrowMorning = new Date(now.getFullYear(), now.getMonth(), now.getDate() + 1, 9, 0, 0); // 明天早上9点

const sampleReminderItem = {
  title: "完成 JSBox 文档更新",
  notes: "确保所有文件都已附上详细解读和示例，并提交到 GitHub。",
  url: "https://github.com/your-repo/jsbox-docs",
  priority: 1, // 高优先级
  completed: false,
  alarmDate: tomorrowMorning,
  creationDate: new Date(now.getTime() - 7 * 24 * 60 * 60 * 1000), // 一周前创建
  modifiedDate: now
};

$ui.render({
  props: { title: "提醒事项详情" },
  views: [
    {
      type: "label",
      props: { text: `标题: ${sampleReminderItem.title}`, font: $font("bold", 20) },
      layout: make => make.top.left.right.inset(10)
    },
    {
      type: "label",
      props: { text: `备注: ${sampleReminderItem.notes}`, lines: 0 },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.right.inset(10)
    },
    {
      type: "label",
      props: { text: `提醒时间: ${sampleReminderItem.alarmDate.toLocaleString()}` },
      layout: make => make.top.equalTo(view.prev.bottom).offset(10).left.right.inset(10)
    },
    {
      type: "label",
      props: { text: `优先级: ${sampleReminderItem.priority}` },
      layout: make => make.top.equalTo(view.prev.bottom).offset(5).left.right.inset(10)
    },
    {
      type: "label",
      props: { text: `已完成: ${sampleReminderItem.completed ? "是" : "否"}` },
      layout: make => make.top.equalTo(view.prev.bottom).offset(5).left.right.inset(10)
    },
    {
      type: "label",
      props: { text: `创建时间: ${sampleReminderItem.creationDate.toLocaleString()}` },
      layout: make => make.top.equalTo(view.prev.bottom).offset(5).left.right.inset(10)
    }
  ]
});
```

**代码解读**：

1.  我们创建了一个 `sampleReminderItem` 对象，它模拟了从 `$reminder` API 获取到的一个提醒事项数据。
2.  然后，我们使用多个 `label` 组件来显示 `sampleReminderItem` 对象的各个属性。注意 `alarmDate`、`creationDate` 和 `completionDate` 都是 `Date` 对象，可以直接使用 `toLocaleString()` 方法进行格式化显示。
3.  `notes` 属性的 `label` 设置了 `lines: 0`，以确保多行内容能够完整显示。

`reminderItem` 对象为 JSBox 脚本提供了与 iOS 提醒事项进行交互的标准化数据模型，使得你可以方便地读取、创建和修改用户的提醒事项。 
