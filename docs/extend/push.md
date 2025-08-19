> 用于构建本地的推送通知，例如一段时间之后提醒

# $push.schedule(object)

设定一个本地推送消息：

```js
$push.schedule({
  title: "标题",
  body: "内容",
  delay: 5,
  handler: function(result) {
    const id = result.id;
  }
})
```

将会在代码执行后 5 秒接收到一个本地推送消息。

参数 | 类型 | 说明
---|---|---
title | string | 标题
body | string | 内容
id | string | 标识符（可选）
sound | string | 声音
mute | bool | 是否静音
repeats | bool | 是否重复
script | string | 脚本名称
height | number | 3D Touch 页面高度
query | json | 额外参数，会被传递到 $context.query
attachments | array | 媒体文件，例如 ["assets/icon.png"]
renew | bool | 是否重复创建（固定通知）

上述样例代码是通过 delay 来触发一个通知，你也可以通过 date 来触发：

```js
const date = new Date();
date.setSeconds(date.getSeconds() + 10)

$push.schedule({
  title: "标题",
  body: "内容",
  date,
  handler: function(result) {
    const id = result.id;
  }
})
```

除此之外，你还能设置一个基于地理位置触发的通知：

```js
$push.schedule({
  title: "标题",
  body: "内容",
  region: {
    lat: 0, // latitude
    lng: 0, // longitude
    radius: 1000, // meters
    notifyOnEntry: true, // notify on entry
    notifyOnExit: true // notify on exit
  }
})
```

该方法调用后会请求用户授权地理位置，若用户拒绝授权则推送不会成功。

从 v1.10.0 开始，$push 支持了通过 script 字段来设置一个脚本，在用户点击推送之后会执行脚本，也可以通过 3D Touch 来预览。

# $push.cancel(object)

取消一个计划内的推送消息：

```js
$push.cancel({
  title: "标题",
  body: "内容",
})
```

将会取消以 `title` 和 `body` 组成的所有推送消息。

当你有多个标题和内容重复的推送时，你可以使用上述代码得到的 id 来取消：

```js
$push.cancel({id: ""})
```

# $push.clear()

清除当前脚本所有通知（在 Build 462 之前注册的通知会被忽略）：

```js
$push.clear()
```

---

## 文件内容解读与示例

### 用途说明

`$push` API 允许你的 JSBox 脚本创建和管理**本地通知（Local Notifications）**。这些通知直接由 iOS 系统在设备上触发，无需任何服务器端支持。它们是提醒用户、执行定时任务、或基于地理位置触发脚本的强大工具。

### 核心方法：`$push.schedule(options)`

这是创建本地通知的核心方法。`options` 对象定义了通知的内容、触发方式和行为。

#### 1. 通知内容

-   `title`: 通知标题。
-   `body`: 通知内容。
-   `attachments`: 一个文件路径数组，用于在通知中显示图片或视频（富媒体通知）。

#### 2. 触发方式

-   **`delay` (延时触发)**: 在脚本执行后，经过指定秒数触发通知。
    ```javascript
    $push.schedule({ delay: 5, title: "提醒", body: "5秒后提醒" });
    ```
-   **`date` (定时触发)**: 在一个具体的 `Date` 对象指定的时间点触发通知。
    ```javascript
    const futureDate = new Date();
    futureDate.setMinutes(futureDate.getMinutes() + 30); // 30分钟后
    $push.schedule({ date: futureDate, title: "会议", body: "30分钟后开会" });
    ```
-   **`region` (地理位置触发)**: 当用户进入或离开指定的地理区域时触发通知。这需要用户授权位置权限。
    ```javascript
    $push.schedule({
      title: "到家提醒",
      body: "你已到达家附近！",
      region: {
        lat: 39.9042, // 纬度
        lng: 116.4074, // 经度 (北京天安门)
        radius: 500, // 半径500米
        notifyOnEntry: true, // 进入区域时通知
        notifyOnExit: false // 离开区域时不通知
      }
    });
    ```

#### 3. 高级行为

-   **`id`**: 为通知指定一个唯一的标识符。这对于后续取消特定通知非常重要。
-   **`repeats`**: 布尔值，如果为 `true`，通知会重复触发（例如每天、每周）。
-   **`script` 和 `query` (JSBox 特色)**: 这是 `$push` 最强大的功能之一。当用户点击通知时，你可以指定一个 JSBox 脚本来执行，并通过 `query` 参数向该脚本传递额外的数据。这使得通知成为了触发脚本的强大入口。

### 通知管理

-   **`$push.cancel(options)`**: 取消一个或多个已计划的通知。你可以通过 `id` 来精确取消，也可以通过 `title` 和 `body` 的组合来取消所有匹配的通知。
    ```javascript
    $push.cancel({ id: "my_unique_reminder_id" });
    $push.cancel({ title: "通用提醒" }); // 取消所有标题为“通用提醒”的通知
    ```
-   **`$push.clear()`**: 清除当前脚本所有已计划和已发送的通知。

### 示例代码：一个交互式提醒脚本

下面的脚本将允许用户设置一个提醒消息和延时。当通知触发并被点击时，它会再次运行脚本并显示原始消息。

```javascript
// 检查脚本是否是由通知触发
if ($context.query && $context.query.reminderMessage) {
  $ui.alert({
    title: "提醒事项",
    message: $context.query.reminderMessage,
    actions: [
      { title: "好的" }
    ]
  });
  $app.close(); // 处理完后关闭脚本
} else {
  // 正常启动时显示设置界面
  $ui.render({
    props: { title: "设置提醒" },
    views: [
      {
        type: "label",
        props: { text: "提醒内容:" },
        layout: make => make.top.left.inset(20)
      },
      {
        type: "input",
        props: {
          id: "message-input",
          placeholder: "输入提醒内容"
        },
        layout: make => {
          make.top.equalTo(view.prev.bottom).offset(10);
          make.left.right.inset(20);
          make.height.equalTo(36);
        }
      },
      {
        type: "label",
        props: { text: "延时 (秒):" },
        layout: make => make.top.equalTo(view.prev.bottom).offset(20).left.inset(20)
      },
      {
        type: "input",
        props: {
          id: "delay-input",
          placeholder: "例如: 5",
          type: $kbType.number
        },
        layout: make => {
          make.top.equalTo(view.prev.bottom).offset(10);
          make.left.right.inset(20);
          make.height.equalTo(36);
        }
      },
      {
        type: "button",
        props: { title: "设置提醒" },
        layout: make => {
          make.top.equalTo(view.prev.bottom).offset(30);
          make.centerX.equalTo(view.super);
          make.width.equalTo(150);
        },
        events: {
          tapped: () => {
            const message = $("message-input").text;
            const delay = parseInt($("delay-input").text);

            if (!message || isNaN(delay) || delay <= 0) {
              $ui.alert("请输入有效的提醒内容和延时！");
              return;
            }

            $push.schedule({
              title: "JSBox 提醒",
              body: message,
              delay: delay,
              id: `reminder_${Date.now()}`, // 确保 ID 唯一
              // 关键：当通知被点击时，再次运行当前脚本，并传递消息
              script: $addin.current.name, // 当前脚本的名称
              query: { reminderMessage: message }
            });

            $ui.toast(`提醒已设置，${delay}秒后触发。`);
            $app.close(); // 设置完成后关闭脚本
          }
        }
      }
    ]
  });
}
```

**代码解读**：

1.  脚本开头通过检查 `$context.query.reminderMessage` 来判断自己是否是由通知触发的。如果是，则显示提醒内容并关闭脚本。
2.  如果不是由通知触发，则显示一个 UI 界面，让用户输入提醒内容和延时。
3.  在“设置提醒”按钮的 `tapped` 事件中，我们调用 `$push.schedule()` 来计划通知。
4.  **最关键的是 `script: $addin.current.name` 和 `query: { reminderMessage: message }`**。这告诉 iOS，当用户点击这个通知时，请再次运行当前脚本，并将 `reminderMessage` 作为参数传递给它。这样，脚本就能在被通知唤醒时，知道要显示什么内容。

`$push` API 为你的 JSBox 脚本提供了强大的定时和事件触发能力，是构建自动化和提醒类工具的基石。 
