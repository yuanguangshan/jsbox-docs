> 用于与系统提醒事项进行交互，例如读取或写入

# $reminder.fetch(object)

用于读取提醒事项（会提示用户授权）：

```js
$reminder.fetch({
  startDate: new Date(),
  hours: 2 * 24,
  handler: function(resp) {
    const events = resp.events;
  }
})
```

看起来和 `$calendar` 极为相似，返回的数据是如下结构：

属性 | 类型 | 读写 | 说明
---|---|---|---
title | string | 读写 | 标题
identifier | string | 只读 | id
location | string | 读写 | 位置
notes | string | 读写 | 备注
url | string | 读写 | 网址
modifiedDate | date | 只读 | 修改时间
creationDate | date | 只读 | 创建时间
completed | boolean | 读写 | 是否完成
completionDate | date | 只读 | 完成时间
alarmDate | date | 读写 | 闹钟时间
priority | number | 读写 | 优先级(1 ~ 9) 

# $reminder.create(object)

用于创建一个提醒事项：

```js
$reminder.create({
  title: "Hey!",
  alarmDate: new Date(),
  notes: "Hello, World!",
  url: "https://apple.com",
  handler: function(resp) {

  }
})
```

可以通过 `alarmDate` 和 `alarmDates` 来指定提醒闹钟时间。

# $reminder.save(object)

和 `$calendar.save` 类似，用于修改 event 后保存更新：

```js
$reminder.save({
  event,
  handler: function(resp) {

  }
})
```

# $reminder.delete(object)

删除某个提醒事项：

```js
$reminder.delete({
  event,
  handler: function(resp) {
    
  }
})
```

> 其实 `$reminder` 和 `$calendar` 极为类似，在 iOS 内部也的确如此，他们有类似的接口设计，并且继承同一个数据基类。

---

## 文件内容解读与示例

### 用途说明

`$reminder` API 提供了与 iOS 系统**提醒事项应用**进行交互的能力。它允许你的脚本读取现有提醒事项、创建新提醒、修改或删除提醒。这对于任务管理、待办事项提醒、或任何需要与用户日程集成的工作流都非常有用。

### 核心概念：权限与 `reminderItem`

-   **权限**: 所有提醒事项操作都需要用户授权。首次调用相关接口时，系统会弹出权限请求。你的脚本应妥善处理用户拒绝授权的情况。
-   **`reminderItem`**: `$reminder` API 返回和操作的都是 `reminderItem` 对象，其结构在 `object/reminder-item.md` 中有详细定义。理解 `reminderItem` 的属性是使用 `$reminder` API 的基础。
-   **与 `$calendar` 的相似性**: `$reminder` 与 `$calendar` API 在设计和使用上非常相似，因为它们在 iOS 内部共享底层框架。如果你熟悉 `$calendar`，那么 `$reminder` 会很容易上手。

### 主要功能与方法详解

#### 1. 读取提醒事项：`$reminder.fetch(options)`

-   **用途**: 获取指定时间范围内的提醒事项。
-   **参数**: 
    -   `startDate`: `Date` 对象，查询的开始时间。
    -   `hours`: 从 `startDate` 开始的小时数，用于定义查询范围。也可以直接指定 `endDate`。
    -   `async`: 布尔值，如果为 `true`，则返回 Promise。
-   **返回**: `resp.events` 数组，包含匹配的 `reminderItem` 对象。

**示例**：获取未来 24 小时内的所有未完成提醒

```javascript
async function fetchAndDisplayReminders() {
  const now = new Date();
  const resp = await $reminder.fetch({
    startDate: now,
    hours: 24, // 获取从现在开始未来 24 小时内的提醒
    async: true // 使用 Promise 方式调用
  });

  if (resp && resp.events) {
    const incompleteReminders = resp.events.filter(r => !r.completed);
    if (incompleteReminders.length > 0) {
      $ui.alert(`未来 24 小时有 ${incompleteReminders.length} 个未完成提醒。`);
      incompleteReminders.forEach(reminder => {
        console.log(`- ${reminder.title} (优先级: ${reminder.priority})`);
      });
    } else {
      $ui.alert("未来 24 小时没有未完成提醒事项。");
    }
  } else if (resp && resp.error) {
    $ui.alert(`获取提醒事项失败: ${resp.error.localizedDescription}`);
  } else {
    $ui.alert("获取提醒事项失败或无权限。");
  }
}
// fetchAndDisplayReminders();
```

#### 2. 创建提醒事项：`$reminder.create(options)`

-   **用途**: 在提醒事项应用中创建一个新的提醒。
-   **参数**: 包含 `reminderItem` 属性的对象，如 `title`, `alarmDate` (提醒时间), `notes`, `url`, `priority` 等。

**示例**：创建一个高优先级的购物提醒

```javascript
async function createShoppingReminder() {
  const now = new Date();
  const alarmTime = new Date(now.getFullYear(), now.getMonth(), now.getDate(), 18, 0, 0); // 今天下午6点

  const resp = await $reminder.create({
    title: "购买牛奶和鸡蛋",
    alarmDate: alarmTime,
    notes: "记得买低脂牛奶和散养鸡蛋。",
    priority: 1, // 高优先级 (1-9, 1最高)
    async: true
  });

  if (resp && resp.event) {
    $ui.toast("购物提醒已创建！");
  } else if (resp && resp.error) {
    $ui.alert(`创建失败: ${resp.error.localizedDescription}`);
  } else {
    $ui.alert("提醒事项创建失败或无权限。");
  }
}
// createShoppingReminder();
```

#### 3. 更新提醒事项：`$reminder.save(options)`

-   **用途**: 修改一个已存在的提醒事项。你需要先通过 `fetch` 获取到 `reminderItem` 对象，修改其属性，然后通过 `save` 提交更改。
-   **参数**: `event` (已修改的 `reminderItem` 对象)。

**示例**：将最近一个提醒标记为完成

```javascript
async function completeLatestReminder() {
  const resp = await $reminder.fetch({
    startDate: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000), // 过去7天
    hours: 7 * 24 * 2, // 未来7天
    async: true
  });

  if (resp && resp.events && resp.events.length > 0) {
    const latestIncompleteReminder = resp.events.find(r => !r.completed); // 找到第一个未完成的
    if (latestIncompleteReminder) {
      latestIncompleteReminder.completed = true; // 标记为完成
      const saveResp = await $reminder.save({ event: latestIncompleteReminder, async: true });

      if (saveResp && saveResp.event) {
        $ui.toast(`提醒事项 "${latestIncompleteReminder.title}" 已标记为完成！`);
      } else if (saveResp && saveResp.error) {
        $ui.alert(`更新失败: ${saveResp.error.localizedDescription}`);
      } else {
        $ui.alert("更新失败或无权限。");
      }
    } else {
      $ui.toast("没有未完成的提醒事项可更新。");
    }
  } else {
    $ui.toast("未找到提醒事项可更新。");
  }
}
// completeLatestReminder();
```

#### 4. 删除提醒事项：`$reminder.delete(options)`

-   **用途**: 从提醒事项应用中删除一个提醒。
-   **参数**: `event` (要删除的 `reminderItem` 对象)。

**示例**：删除最近一个提醒

```javascript
async function deleteLatestReminder() {
  const resp = await $reminder.fetch({
    startDate: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000), // 过去7天
    hours: 7 * 24 * 2, // 未来7天
    async: true
  });

  if (resp && resp.events && resp.events.length > 0) {
    const reminderToDelete = resp.events[0]; // 获取第一个提醒
    const deleteResp = await $reminder.delete({ event: reminderToDelete, async: true });

    if (deleteResp && deleteResp.success) {
      $ui.toast(`提醒事项 "${reminderToDelete.title}" 已删除！`);
    } else if (deleteResp && deleteResp.error) {
      $ui.alert(`删除失败: ${deleteResp.error.localizedDescription}`);
    } else {
      $ui.alert("删除失败或无权限。");
    }
  } else {
    $ui.toast("未找到提醒事项可删除。");
  }
}
// deleteLatestReminder();
```

### 总结

`$reminder` API 为你的 JSBox 脚本提供了与 iOS 提醒事项应用进行深度交互的全面能力。无论是创建待办事项、管理任务状态，还是自动化提醒流程，它都是不可或缺的工具。 
