> 用于处理和短信或邮件相关的操作

# $message.sms(object)

调用系统接口发送短信：

```js
$http.download({
  url: "https://images.apple.com/v/iphone/compare/f/images/compare/compare_iphone7_jetblack_large_2x.jpg",
  handler: function(resp) {
    $message.sms({
      recipients: ["18688888888", "10010"],
      body: "Message body",
      subject: "Message subject",
      attachments: [resp.data],
      handler: function(result) {

      }
    })
  }
})
```

参数 | 说明
---|---
recipients | 接受者
body | 内容
subject | 主题
attachments | 附件(图片或文件)
result | 0: 取消 1: 成功 2: 失败

# $message.mail(object)

调用系统接口发送邮件：

```js
$http.download({
  url: "https://images.apple.com/v/iphone/compare/f/images/compare/compare_iphone7_jetblack_large_2x.jpg",
  handler: function(resp) {
    $message.mail({
      subject: "Message subject",
      to: ["18688888888", "10010"],
      cc: [],
      bcc: [],
      body: "Message body",
      attachments: [resp.data],
      handler: function(result) {

      }
    })
  }
})
```

参数 | 说明
---|---
subject | 主题
to | 接受者
cc | 抄送
bcc | 密送
body | 内容
isHTML | 内容是否为 HTML
attachments | 附件(图片或文件)
result | 0: 取消 1: 保存 2: 成功 3: 失败

---

## 文件内容解读与示例

### 用途说明

`$message` API 提供了在 JSBox 脚本中**发送短信和电子邮件**的能力。它会调用 iOS 系统内置的短信和邮件撰写界面，并允许你预填充收件人、主题、内容和附件。这对于构建快速联系、自动化通知或报告发送的脚本非常有用，同时确保了用户对发送内容的最终控制权。

### 核心概念：系统撰写界面与异步操作

-   **系统撰写界面**: `$message` API 不会直接在后台发送消息。相反，它会弹出 iOS 系统提供的短信或邮件撰写界面。用户需要手动点击界面上的“发送”按钮来完成操作。这保证了用户对发送内容的最终控制权和隐私。
-   **异步操作**: 所有 `$message` 操作都是异步的。它们会立即返回，而实际的结果（用户是否发送、取消或保存）会通过 `handler` 回调函数返回。

### 主要功能与方法详解

#### 1. 发送短信：`$message.sms(options)`

-   **用途**: 弹出短信撰写界面，并预填充相关信息。
-   **参数**: 
    -   `recipients`: 字符串数组，收件人电话号码。
    -   `body`: 短信内容。
    -   `subject`: 可选，短信主题（部分设备或应用可能不支持显示）。
    -   `attachments`: 可选，`$data` 对象数组，作为短信附件（如图片）。
    -   `handler`: 回调函数，`result` 为 `0`（取消）, `1`（成功）, `2`（失败）。

**示例**：发送一条预设内容的短信

```javascript
$ui.render({
  props: { title: "发送短信" },
  views: [
    {
      type: "button",
      props: { title: "发送短信给 10086" },
      layout: make => make.center.equalTo(view.super).width.equalTo(200).height.equalTo(40),
      events: {
        tapped: () => {
          $message.sms({
            recipients: ["10086"],
            body: "查询话费",
            handler: (result) => {
              if (result === 1) {
                $ui.toast("短信已发送！");
              } else if (result === 0) {
                $ui.toast("短信发送已取消。");
              } else {
                $ui.toast("短信发送失败。");
              }
            }
          });
        }
      }
    }
  ]
});
```

#### 2. 发送邮件：`$message.mail(options)`

-   **用途**: 弹出邮件撰写界面，并预填充相关信息。
-   **参数**: 
    -   `to`, `cc`, `bcc`: 字符串数组，收件人、抄送、密送邮箱地址。
    -   `subject`: 邮件主题。
    -   `body`: 邮件内容。
    -   `isHTML`: 布尔值，如果为 `true`，则 `body` 内容会被解析为 HTML 格式。
    -   `attachments`: 可选，`$data` 对象数组，作为邮件附件。
    -   `handler`: 回调函数，`result` 为 `0`（取消）, `1`（保存草稿）, `2`（成功）, `3`（失败）。

**示例**：发送一封带附件的邮件

```javascript
async function sendEmailWithAttachment() {
  // 假设你有一个本地的图片文件，例如 "assets/report.png"
  const imageToAttach = $file.read("assets/report.png"); // 确保 assets/report.png 存在

  if (!imageToAttach) {
    $ui.alert("附件文件不存在！");
    return;
  }

  $ui.render({
    props: { title: "发送邮件" },
    views: [
      {
        type: "button",
        props: { title: "发送报告邮件" },
        layout: make => make.center.equalTo(view.super).width.equalTo(200).height.equalTo(40),
        events: {
          tapped: () => {
            $message.mail({
              to: ["report@example.com"],
              subject: "JSBox 自动化报告",
              body: "<p>这是我用 <b>JSBox</b> 脚本生成的报告，请查收。</p>",
              isHTML: true, // 邮件内容是 HTML 格式
              attachments: [imageToAttach], // 添加附件
              handler: (result) => {
                if (result === 2) {
                  $ui.toast("邮件已发送！");
                } else if (result === 0) {
                  $ui.toast("邮件发送已取消。");
                } else {
                  $ui.toast("邮件发送失败或保存草稿。");
                }
              }
            });
          }
        }
      }
    ]
  });
}
// sendEmailWithAttachment();
```

### 总结

`$message` API 提供了一种用户友好的方式，将短信和邮件发送功能集成到你的 JSBox 脚本中。它通过调用系统撰写界面，确保了用户对发送内容的最终控制权，同时提供了预填充内容和附件的能力，极大地提升了自动化效率。 
