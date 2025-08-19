# 两种模式

JSBox 提供的异步接口里面，对 Promise 的支持两种模式：

- 直接支持 Promise 相关调用
- 需要传参 `async: true` 表明这是一个 Promise 调用

这两种形式其实很好理解，是为了兼容早期 JSBox 的 handler 风格接口而做的妥协，之后可能会全部迁移至第一种。

简单来说，在有些接口里面，你必须处理回调，例如：

```js
$ui.menu({
  items: ["A", "B", "C"],
  handler: (title, index) => {

  }
})
```

如果你弹出一个菜单，用户选择之后什么都不做，这是没有道理的，所以这种情况我们会认为这是一个“必须要回调”的接口，你可以直接：

```js
var result = await $ui.menu({ items: ["A", "B", "C"] })
var title = result.title
var index = result.index
// Or: var result = await $ui.menu(["A", "B", "C"])
```

但是有些情况则不然，比如你可以删除一个图片，这个时候回调也可以不回调也没事：

```js
$photo.delete({
  count: 3,
  screenshot: true,
  handler: success => {
    // It's OK to remove handler here
  }
})
```

对于这种可选回调的异步接口，你需要添加 async 参数来指明你需要 Promise 调用：

```js
var success = await $photo.delete({
  count: 3,
  screenshot: true,
  async: true
})
```

这样的取舍是为了兼容两种调用方式，如果不指明 `async: true` 的话，这将是一个可选的 callback 回调。

同时，请注意不是所有的接口都能异步，有些接口完全是同步接口，例如：

```js
const success = $file.delete("sample.txt");
```

这样的接口没有必要（也无法）Promise 调用，我们之后将会有一个表来详细说明各个接口的支持情况。

# 一些好用的简写

当你使用 Promise 之后，你可能会发现有时候参数只剩下一个，是没有必要包装成一个 JSON 数据的，所以我们提供了一些简写，例如上面提到过的：

```js
var resp = await $http.get('https://docs.xteko.com')
```

这样一些方便的形式可以少写很多代码，这是一些样例：

```js
// Thread
await $thread.main(3)
await $thread.background(3)

// HTTP
var resp = await $http.get('https://docs.xteko.com')
var result = await $http.shorten("http://docs.xteko.com")

// UIKit
var result = await $ui.menu(["A", "B", "C"])

// Cache
var cache = await $cache.getAsync("key")
```

---

## 文件内容解读与示例

### 用途说明

本文档详细阐述了 JSBox API 中 Promise 支持的**两种模式**（`required` 和 `optional`），并介绍了一些常用的**简写形式**。理解这些模式对于编写高效、简洁且符合 JSBox 规范的异步代码至关重要。

### 核心概念：Promise 支持的两种模式

JSBox 的异步接口对 Promise 的支持分为两种模式，这是为了在引入 Promise 的同时，保持对早期回调风格接口的兼容性。

#### 1. 直接支持 Promise (Required Promise)

-   **含义**: 这些 API 方法在设计上就预期会返回一个结果，因此它们**默认直接返回一个 Promise 对象**。你无需额外参数即可使用 `await` 或 `.then()`。
-   **典型场景**: 那些“必须”有回调的接口，例如 `$ui.menu()`（用户选择后总会返回结果）、`$http.get()`（网络请求总会有成功或失败）。
-   **示例**: 
    ```javascript
    // 弹出一个菜单，并等待用户选择
    const result = await $ui.menu({ items: ["选项 A", "选项 B"] });
    $ui.alert(`你选择了: ${result.title} (索引: ${result.index})`);
    ```

#### 2. 可选支持 Promise (Optional Promise with `async: true`)

-   **含义**: 这些 API 方法在设计上，其回调函数是可选的。如果你不关心操作结果，可以不提供回调。但如果你想使用 Promise 模式（即 `await` 或 `.then()`），你必须在调用时明确传入 `async: true` 参数。
-   **典型场景**: 那些“可以”有回调的接口，例如 `$photo.save()`（保存图片后，你可能关心是否成功，也可能不关心）。
-   **示例**: 
    ```javascript
    // 假设 pickedImage 是一个 $image 对象
    const saveSuccess = await $photo.save({
      image: pickedImage,
      async: true // 明确指明需要 Promise 返回
    });
    if (saveSuccess) {
      $ui.toast("图片已保存到相册！");
    } else {
      $ui.toast("图片保存失败或操作取消。");
    }
    ```

### 同步接口的例外

需要注意的是，并非所有 API 都支持 Promise。一些操作是**同步的**（例如 `$file.delete("sample.txt")`），它们会立即返回结果，因此没有必要（也无法）使用 Promise。对于这些接口，你直接获取返回值即可。

### 常用简写形式 (Shorthands)

JSBox 为了一些常用 API 提供了简写形式，当这些 API 的 `options` 参数只有一个且是字符串时，可以省略 `options` 对象，直接传入字符串。

-   **`$http.get(url)`**: 
    ```javascript
    const resp = await $http.get("https://api.ipify.org?format=json");
    console.log("IP 地址:", resp.data.ip);
    ```

-   **`$http.shorten(url)`**: 
    ```javascript
    const shortUrl = await $http.shorten("https://www.jsbox.com");
    console.log("短链接:", shortUrl);
    ```

-   **`$ui.menu(items)`**: 
    ```javascript
    const result = await $ui.menu(["选项 A", "选项 B", "选项 C"]);
    console.log("选择结果:", result.title);
    ```

-   **`$thread.main(delay)` / `$thread.background(delay)`**: 
    ```javascript
    await $thread.main(1); // 在主线程延迟 1 秒
    $ui.toast("延迟 1 秒后显示");
    ```

-   **`$cache.getAsync(key)`**: 
    ```javascript
    const cachedData = await $cache.getAsync("my_cached_key");
    console.log("缓存数据:", cachedData);
    ```

### 示例代码：综合运用 Promise 模式和简写

下面的示例将综合运用 Promise 的两种模式和简写形式，实现一个获取 IP 地址、让用户选择图片并保存的流程。

```javascript
async function main() {
  $ui.toast("开始执行异步任务...");

  try {
    // 1. 获取 IP 地址 (required Promise, 使用简写)
    const ipResp = await $http.get("https://api.ipify.org?format=json");
    $ui.alert(`你的 IP 地址是: ${ipResp.data.ip}`);

    // 2. 让用户选择图片 (optional Promise, 需 async: true)
    const photoResp = await $photo.pick({ multi: true, selectionLimit: 2, async: true });
    if (photoResp && photoResp.results && photoResp.results.length > 0) {
      $ui.toast(`选择了 ${photoResp.results.length} 张图片。`);

      // 3. 遍历并保存每张图片 (optional Promise, 需 async: true)
      for (const item of photoResp.results) {
        const saveSuccess = await $photo.save({
          image: item.image,
          async: true
        });
        if (saveSuccess) {
          console.log(`图片 ${item.filename} 保存成功。`);
        } else {
          console.log(`图片 ${item.filename} 保存失败。`);
        }
      }
      $ui.alert("所有图片保存完成！");
    } else {
      $ui.toast("未选择图片或操作取消。");
    }

  } catch (error) {
    console.error("任务执行失败:", error);
    $ui.alert(`任务失败: ${error.localizedDescription || error}`);
  }
}

main();
```

**代码解读**：

-   `$http.get()` 直接返回 Promise，并使用了简写形式。
-   `$photo.pick()` 和 `$photo.save()` 是 `optional` Promise，因此在 `options` 中明确传入了 `async: true`。
-   `await` 关键字使得整个异步流程看起来像同步代码一样清晰，易于理解和维护。

### 总结

理解 Promise 的两种模式和简写形式，能够帮助开发者编写更符合 JSBox 规范、更简洁、更易读的异步代码。查阅 `promise/index.md` 可以获取所有 API 的详细支持情况。 
