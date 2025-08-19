# Promise

Promise 是一种为了解决回调地狱而引入的方案，请参考 Mozilla 的文档了解更多：https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise

JSBox 里面当然可以使用 Promise，但是 JSBox 里面提供的接口早期并不支持 Promise，只支持通过 callback 回调，比如说：

```js
$http.get({
  url: 'https://docs.xteko.com',
  handler: function(resp) {
    const data = resp.data;
  }
})
```

这个例子通过 `handler` 进行下一步操作，但从 v1.15.0 开始，你可以有更好的方案：

```js
$http.get({ url: 'https://docs.xteko.com' }).then(resp => {
  const data = resp.data;
})
```

这种写法可以避免你陷入回调地狱，或者更进一步地（iOS 10.3 以上）：

```js
var resp = await $http.get({ url: 'https://docs.xteko.com' })
var data = resp.data
```

如果你为了测试之用，对于 `$http.get` 你还能使用这个极简形式：

```js
var resp = await $http.get('https://docs.xteko.com')
var data = resp.data
```

关于 Promise 的种种细节，本文档不会详细描述，但这里会推荐一篇文档：https://javascript.info/async

总的来说，JSBox 现在已经对于一些 API 提供了 Promise 调用。

---

## 文件内容解读与示例

### 用途说明

本文档介绍了 **Promise** 这一现代 JavaScript 异步编程的核心概念，以及它在 JSBox 中的应用。在移动应用开发中，网络请求、文件读写等操作都是异步的。传统的回调函数方式容易导致代码嵌套过深，形成难以维护的“回调地狱”（Callback Hell）。Promise 的引入正是为了解决这一问题，它提供了一种更清晰、更优雅的方式来处理异步操作。

### 核心概念：从回调到 Promise

1.  **回调地狱**: 想象你需要依次执行多个异步操作，每个操作的结果都依赖于上一个操作。传统的回调函数会导致代码层层嵌套，可读性极差。

    ```javascript
    // 传统回调地狱示例
    fetchUser(userId, (user) => {
      fetchPosts(user.id, (posts) => {
        fetchComments(posts[0].id, (comments) => {
          // ... 更多嵌套
        });
      });
    });
    ```

2.  **Promise 的解决方案**: Promise 将异步操作的结果封装成一个对象，这个对象代表了异步操作的最终完成（`resolved`）或失败（`rejected`）。你可以通过 `.then()` 方法来处理成功的结果，通过 `.catch()` 方法来处理失败的结果，从而将异步操作串联起来，形成链式调用，避免了深层嵌套。

    ```javascript
    // Promise 链式调用示例
    fetchUser(userId)
      .then(user => fetchPosts(user.id))
      .then(posts => fetchComments(posts[0].id))
      .then(comments => { /* ... */ })
      .catch(error => { /* 处理错误 */ });
    ```

3.  **`async/await` 的优雅**: `async/await` 是基于 Promise 的语法糖，它使得异步代码的编写和阅读更像同步代码。在一个 `async` 函数中，你可以使用 `await` 关键字等待一个 Promise 对象的解析完成，而不会阻塞主线程。

    ```javascript
    // async/await 示例
    async function fetchData() {
      try {
        const user = await fetchUser(userId);
        const posts = await fetchPosts(user.id);
        const comments = await await fetchComments(posts[0].id);
        // ...
      } catch (error) {
        // 处理错误
      }
    }
    ```

### JSBox 中 Promise 的应用

JSBox 的许多 API 已经支持 Promise。这意味着你可以选择使用传统的基于回调的写法，也可以采用更现代的 Promise 或 `async/await` 方式。

-   **`$http.get()` 的简化形式**: 文档中提到，对于 `$http.get()` 等方法，你可以直接 `await` 一个 URL 字符串，JSBox 会自动将其转换为 Promise 请求。

### 示例代码：使用 `async/await` 获取随机用户数据

下面的示例将演示如何使用 `async/await` 来从一个公共 API 获取随机用户数据，并显示其头像和姓名。

```javascript
async function fetchRandomUser() {
  $ui.toast("正在获取用户数据...");
  try {
    // 1. 发起网络请求获取用户数据
    const userResp = await $http.get({
      url: "https://randomuser.me/api/",
      timeout: 5 // 设置超时
    });

    if (userResp.error) {
      $ui.alert(`获取用户数据失败: ${userResp.error.localizedDescription}`);
      return;
    }

    const user = userResp.data.results[0];
    const userName = `${user.name.first} ${user.name.last}`;
    const userPhotoUrl = user.picture.large;

    // 2. 下载用户头像图片
    const photoResp = await $http.download({
      url: userPhotoUrl,
      timeout: 5
    });

    if (photoResp.error) {
      $ui.alert(`下载头像失败: ${photoResp.error.localizedDescription}`);
      return;
    }

    const userPhoto = photoResp.data.image; // 获取 $image 对象

    // 3. 更新 UI 显示
    $ui.render({
      props: { title: "随机用户" },
      views: [
        {
          type: "image",
          props: {
            image: userPhoto,
            cornerRadius: 50 // 圆形头像
          },
          layout: make => {
            make.top.inset(50);
            make.centerX.equalTo(view.super);
            make.size.equalTo($size(100, 100));
          }
        },
        {
          type: "label",
          props: {
            text: userName,
            font: $font("bold", 24),
            align: $align.center
          },
          layout: make => {
            make.top.equalTo(view.prev.bottom).offset(20);
            make.centerX.equalTo(view.super);
          }
        }
      ]
    });
    $ui.toast("用户数据已加载！");

  } catch (e) {
    $ui.alert(`发生未知错误: ${e}`);
  }
}

// 调用异步函数
fetchRandomUser();
```

**代码解读**：

1.  整个逻辑被封装在一个 `async` 函数 `fetchRandomUser()` 中。
2.  我们使用 `await $http.get()` 来等待用户数据网络请求的完成。代码会在这里暂停执行，直到 Promise 解决或拒绝。
3.  获取到用户数据后，我们再使用 `await $http.download()` 来下载用户头像。这两个异步操作以顺序化的方式编写，避免了回调嵌套。
4.  `try...catch` 块用于捕获 Promise 链中可能发生的任何错误，确保程序的健壮性。

### 总结

Promise 和 `async/await` 是 JSBox 中处理异步操作的现代、推荐方式。它们使得代码更清晰、更易于维护，是编写复杂脚本的必备技能。通过它们，你可以将复杂的异步流程转化为易于理解的顺序代码。 
