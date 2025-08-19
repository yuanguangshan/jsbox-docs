# 接口列表

这里是一个表格，用于列举我们在前面文章中提到过的 Promise 接口问题，我们用 `required` 表示这是一个必须要回调的接口，也即你可以直接使用 Promise，用 `optional` 来表示该接口的回调可以省略，当你需要 Promise 调用的时候，需要 `async: true` 参数来指明。

# $thread

接口 | 类型
---|---
main | required
background | required

# $http

接口 | 类型
---|---
request | required
get | required
post | required
download | required
upload | required
shorten | required
lengthen | required
startServer | required

# $push

接口 | 类型
---|---
schedule | optional

# $drive

接口 | 类型
---|---
save | required
open | required

# $cache

接口 | 类型
---|---
setAsync | required
getAsync | required
removeAsync | required
clearAsync | required

# $photo

接口 | 类型
---|---
take | required
pick | required
save | optional
fetch | required
delete | optional

# $input

接口 | 类型
---|---
text | required
speech | required

# $ui

接口 | 类型
---|---
animate | optional
alert | required
action | required
menu | required

# $message

接口 | 类型
---|---
sms | optional
mail | optional

# $calendar

接口 | 类型
---|---
fetch | required
create | optional
save | optional
delete | optional

# $reminder

接口 | 类型
---|---
fetch | required
create | optional
save | optional
delete | optional

# $contact

接口 | 类型
---|---
pick | required
fetch | required
create | optional
save | optional
delete | optional
fetchGroups | required
addGroup | optional
deleteGroup | optional
updateGroup | optional
addToGroup | optional
removeFromGroup | optional

# $location

接口 | 类型
---|---
select | required

# $ssh

接口 | 类型
---|---
connect | required

# $text

接口 | 类型
---|---
analysis | required
tokenize | required
htmlToMarkdown | required

# $qrcode

接口 | 类型
---|---
scan | required

# $pdf

接口 | 类型
---|---
make | required

# $quicklook

接口 | 类型
---|---
open | optional

# $safari

接口 | 类型
---|---
open | optional

# $archiver

接口 | 类型
---|---
zip | required
unzip | required

# $browser

接口 | 类型
---|---
exec | required

# $picker

接口 | 类型
---|---
date | required
data | required
color | required

# $keyboard

接口 | 类型
---|---
getAllText | required

---

## 文件内容解读与示例

### 用途说明

本文档是 JSBox 中**支持 Promise 异步模式的 API 接口索引**。它列出了所有可以返回 Promise 对象的 JSBox API 方法，并区分了哪些方法默认返回 Promise（`required`），以及哪些方法需要额外参数（如 `async: true`）才能返回 Promise。这对于编写现代、可读性高且易于维护的异步 JSBox 脚本至关重要。

### 核心概念：Promise 与 `async/await`

-   **Promise**: Promise 是 JavaScript 中处理异步操作的标准方式。它代表一个异步操作的最终完成（或失败）及其结果值。Promise 使得异步代码的组织更加清晰，避免了传统回调函数嵌套过深导致的“回调地狱”。
-   **`async/await`**: `async/await` 是基于 Promise 的语法糖，它使得异步代码的编写和阅读更像同步代码。在一个 `async` 函数中，你可以使用 `await` 关键字等待一个 Promise 对象的解析完成，而不会阻塞主线程。

### 接口类型区分：`required` vs. `optional`

-   **`required` (默认支持 Promise)**:
    -   **含义**: 这些 API 方法在调用时会**直接返回一个 Promise 对象**。你可以直接对它们使用 `await` 关键字，或者使用 `.then().catch()` 链式调用来处理异步结果。
    -   **示例**: `$http.get()`, `$thread.background()`, `$photo.pick()`。

-   **`optional` (可选支持 Promise)**:
    -   **含义**: 这些 API 方法默认可能使用回调函数来处理异步结果。但如果你在调用时传入 `async: true` 参数，它们就会**返回一个 Promise 对象**，从而允许你使用 `await` 语法。
    -   **示例**: `$push.schedule()`, `$ui.animate()`, `$photo.save()`。
    -   **重要提示**: 对于 `optional` 类型的接口，如果你想使用 `async/await`，就必须在 `options` 参数中明确传入 `async: true`。

### 如何使用 Promise 接口 (通用模式)

#### 1. 使用 `async/await` (推荐)

```javascript
async function performComplexTask() {
  try {
    $ui.toast("开始获取数据...");
    // 1. 发起网络请求 (required Promise)
    const resp = await $http.get("https://api.ipify.org?format=json");
    const ipAddress = resp.data.ip;
    $ui.toast(`获取到 IP: ${ipAddress}`);

    // 2. 让用户选择一张图片 (optional Promise, 需要 async: true)
    const photoResp = await $photo.pick({ async: true });
    if (photoResp && photoResp.image) {
      $ui.toast("图片已选择！");
      // 3. 保存图片到相册 (optional Promise, 需要 async: true)
      const saveSuccess = await $photo.save({
        image: photoResp.image,
        async: true
      });
      if (saveSuccess) {
        $ui.toast("图片已保存到相册！");
      } else {
        $ui.toast("图片保存失败。");
      }
    } else {
      $ui.toast("未选择图片或操作取消。");
    }

    $ui.alert("所有任务完成！");
  } catch (error) {
    console.error("任务执行失败:", error);
    $ui.alert(`任务失败: ${error.localizedDescription || error}`);
  }
}

// 启动任务
performComplexTask();
```

#### 2. 使用 `.then().catch()` 链式调用

```javascript
$http.get("https://api.ipify.org?format=json").then(resp => {
  const ipAddress = resp.data.ip;
  $ui.toast(`获取到 IP: ${ipAddress}`);

  return $photo.pick({ async: true }); // 返回 Promise，继续链式调用
}).then(photoResp => {
  if (photoResp && photoResp.image) {
    $ui.toast("图片已选择！");
    return $photo.save({ image: photoResp.image, async: true });
  } else {
    $ui.toast("未选择图片或操作取消。");
    return Promise.resolve(false); // 返回一个已解决的 Promise，避免中断链式调用
  }
}).then(saveSuccess => {
  if (saveSuccess) {
    $ui.toast("图片已保存到相册！");
  } else {
    // 可能是因为未选择图片导致 saveSuccess 为 false
  }
  $ui.alert("所有任务完成！");
}).catch(error => {
  console.error("任务执行失败:", error);
  $ui.alert(`任务失败: ${error.localizedDescription || error}`);
});
```

**代码解读**：

-   `async/await` 使得异步操作的流程看起来更加线性，易于理解和调试。
-   `try...catch` 块用于捕获 Promise 链中的任何错误。
-   对于 `optional` 类型的接口（如 `$photo.pick` 和 `$photo.save`），我们明确传入了 `async: true`，以便它们返回 Promise，从而可以使用 `await`。

### 总结

Promise 和 `async/await` 是 JSBox 中处理异步操作的现代、推荐方式。查阅此文档可以帮助开发者快速识别哪些 API 支持 Promise，从而编写出更优雅、更易于维护的异步代码。 
