> 用于与系统日历进行交互，例如读取或写入

# $calendar.fetch(object)

用于读取日历事件（会提示用户授权）：

```js
$calendar.fetch({
  startDate: new Date(),
  hours: 3 * 24,
  handler: function(resp) {
    const events = resp.events;
  }
})
```

表示读取从现在开始往后 3 天的所有日历事件，除了指定 `hours` 以外，也可以自己计算并指定 `endDate`。

在返回的数据里面，events 内含的对象结构如下：

属性 | 类型 | 读写 | 说明
---|---|---|---
title | string | 读写 | 标题
identifier | string | 只读 | id
location | string | 读写 | 位置
notes | string | 读写 | 备注
url | string | 读写 | 网址
modifiedDate | date | 只读 | 修改时间
creationDate | date | 只读 | 创建时间
allDay | boolean | 只读 | 是否全天
startDate | date | 只读 | 开始时间
endDate | date | 只读 | 结束时间
status | number | 只读 | 状态[请参考](https://developer.apple.com/documentation/eventkit/ekeventstatus)

# $calendar.create(object)

用于创建一个日历事项：

```js
$calendar.create({
  title: "Hey!",
  startDate: new Date(),
  hours: 3,
  notes: "Hello, World!",
  handler: function(resp) {

  }
})
```

# $calendar.save(object)

通过 `$calendar.fetch` 取出来的日历项，可以修改一些属性，然后通过 save 接口更新：

```js
$calendar.fetch({
  startDate: new Date(),
  hours: 3 * 24,
  handler: function(resp) {
    const event = resp.events[0];
    event.title = "Modified"
    $calendar.save({
      event
    })
  }
})
```

可以通过 `alarmDate` 和 `alarmDates` 来指定提醒闹钟时间。

# $calendar.delete(object)

在系统日历中删除某个日历项：

```js
$calendar.delete({
  event,
  handler: function(resp) {
    
  }
})
```

---

## 文件内容解读与示例

### 用途说明

`$calendar` API 提供了与 iOS 系统**日历应用**进行交互的能力。它允许你的脚本读取现有日历事件、创建新事件、修改或删除事件。这对于日程管理、会议提醒、或任何需要与用户日程集成的工作流都非常有用。

### 核心概念：权限与 `calendarItem`

-   **权限**: 所有日历操作都需要用户授权。首次调用相关接口时，系统会弹出权限请求。你的脚本应妥善处理用户拒绝授权的情况。
-   **`calendarItem`**: `$calendar` API 返回和操作的都是 `calendarItem` 对象，其结构在 `object/calendar-item.md` 中有详细定义。理解 `calendarItem` 的属性是使用 `$calendar` API 的基础。

### 主要功能与方法详解

#### 1. 读取日历事件：`$calendar.fetch(options)`

-   **用途**: 获取指定时间范围内的日历事件。
-   **参数**: 
    -   `startDate`: `Date` 对象，查询的开始时间。
    -   `hours`: 从 `startDate` 开始的小时数，用于定义查询范围。也可以直接指定 `endDate`。
    -   `calendars`: 可选，指定要查询的日历 ID 数组。
    -   `async`: 布尔值，如果为 `true`，则返回 Promise。
-   **返回**: `resp.events` 数组，包含匹配的 `calendarItem` 对象。

**示例**：获取未来 24 小时内的所有日历事件

```javascript
async function fetchAndDisplayEvents() {
  const now = new Date();
  const resp = await $calendar.fetch({
    startDate: now,
    hours: 24, // 获取从现在开始未来 24 小时内的事件
    async: true // 使用 Promise 方式调用
  });

  if (resp && resp.events) {
    if (resp.events.length > 0) {
      $ui.alert(`未来 24 小时有 ${resp.events.length} 个事件。`);
      resp.events.forEach(event => {
        console.log(`- ${event.title} (${event.startDate.toLocaleString()})`);
      });
    } else {
      $ui.alert("未来 24 小时没有日历事件。");
    }
  } else if (resp && resp.error) {
    $ui.alert(`获取日历事件失败: ${resp.error.localizedDescription}`);
  } else {
    $ui.alert("获取日历事件失败或无权限。");
  }
}
// fetchAndDisplayEvents();
```

#### 2. 创建日历事件：`$calendar.create(options)`

-   **用途**: 在日历中创建一个新的事件。
-   **参数**: 包含 `calendarItem` 属性的对象，如 `title`, `startDate`, `endDate` (或 `hours`), `notes`, `location`, `url` 等。

**示例**：创建一个 30 分钟后的午餐会议

```javascript
async function createLunchMeeting() {
  const now = new Date();
  const startTime = new Date(now.getTime() + 30 * 60 * 1000); // 30分钟后
  const endTime = new Date(startTime.getTime() + 60 * 60 * 1000); // 持续1小时

  const resp = await $calendar.create({
    title: "午餐会议",
    startDate: startTime,
    endDate: endTime,
    location: "公司餐厅",
    notes: "与团队讨论项目进展。",
    async: true
  });

  if (resp && resp.event) {
    $ui.toast("日历事件创建成功！");
  } else if (resp && resp.error) {
    $ui.alert(`创建失败: ${resp.error.localizedDescription}`);
  } else {
    $ui.alert("日历事件创建失败或无权限。");
  }
}
// createLunchMeeting();
```

#### 3. 更新日历事件：`$calendar.save(options)`

-   **用途**: 修改一个已存在的日历事件。你需要先通过 `fetch` 获取到 `calendarItem` 对象，修改其属性，然后通过 `save` 提交更改。
-   **参数**: `event` (已修改的 `calendarItem` 对象)。
-   可以指定 `alarmDate` 或 `alarmDates` 来设置提醒闹钟。

**示例**：更新最近一个事件的标题

```javascript
async function updateLatestEventTitle() {
  const resp = await $calendar.fetch({
    startDate: new Date(Date.now() - 24 * 60 * 60 * 1000), // 过去24小时
    hours: 48, // 未来24小时
    async: true
  });

  if (resp && resp.events && resp.events.length > 0) {
    const latestEvent = resp.events[0]; // 获取最近的一个事件
    latestEvent.title = `[已更新] ${latestEvent.title}`; // 修改标题

    const saveResp = await $calendar.save({
      event: latestEvent,
      async: true
    });

    if (saveResp && saveResp.event) {
      $ui.toast("事件标题已更新！");
    } else if (saveResp && saveResp.error) {
      $ui.alert(`更新失败: ${saveResp.error.localizedDescription}`);
    } else {
      $ui.alert("更新失败或无权限。");
    }
  } else {
    $ui.toast("未找到事件可更新。");
  }
}
// updateLatestEventTitle();
```

#### 4. 删除日历事件：`$calendar.delete(options)`

-   **用途**: 从日历中删除一个事件。
-   **参数**: `event` (要删除的 `calendarItem` 对象)。

**示例**：删除最近一个事件

```javascript
async function deleteLatestEvent() {
  const resp = await $calendar.fetch({
    startDate: new Date(Date.now() - 24 * 60 * 60 * 1000), // 过去24小时
    hours: 48, // 未来24小时
    async: true
  });

  if (resp && resp.events && resp.events.length > 0) {
    const eventToDelete = resp.events[0];
    const deleteResp = await $calendar.delete({
      event: eventToDelete,
      async: true
    });

    if (deleteResp && deleteResp.success) {
      $ui.toast("事件已删除！");
    } else if (deleteResp && deleteResp.error) {
      $ui.alert(`删除失败: ${deleteResp.error.localizedDescription}`);
    } else {
      $ui.alert("删除失败或无权限。");
    }
  } else {
    $ui.toast("未找到事件可删除。");
  }
}
// deleteLatestEvent();
```

### 总结

`$calendar` API 为你的 JSBox 脚本提供了与 iOS 日历应用进行深度交互的全面能力。无论是获取用户日程、创建会议提醒，还是自动化事件管理，它都是不可或缺的工具。 
