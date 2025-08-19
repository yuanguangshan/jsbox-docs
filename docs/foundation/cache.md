> 持久化，是程序必备的内容，JSBox 提供了多种持久化方式，这里我们介绍对象缓存。

# 提示

JSBox 提供了对象的内存缓存和文件缓存，需注意缓存的对象要遵循 [NSCoding](https://developer.apple.com/documentation/foundation/nscoding)，默认的 JavaScript 对象都支持。

所有的接口均提供同步操作和异步操作（Async 结尾），可以根据需要选择。

# $cache.set(string, object)

写入缓存：

```js
$cache.set("sample", {
  "a": [1, 2, 3],
  "b": "1, 2, 3"
})
```

# $cache.setAsync(object)

异步写入缓存：

```js
$cache.setAsync({
  key: "sample",
  value: {
    "a": [1, 2, 3],
    "b": "1, 2, 3"
  },
  handler: function(object) {

  }
})
```

# $cache.get(string)

读取缓存：

```js
$cache.get("sample")
```

# $cache.getAsync(object)

异步读取缓存：

```js
$cache.getAsync({
  key: "sample",
  handler: function(object) {

  }
})
```

# $cache.remove(string)

移除缓存：

```js
$cache.remove("sample")
```

# $cache.removeAsync(object)

异步移除缓存：

```js
$cache.removeAsync({
  key: "sample",
  handler: function() {
    
  }
})
```

# $cache.clear()

清除该扩展的全部缓存，并不会影响其他扩展产生的任何缓存：

```js
$cache.clear()
```

# $cache.clearAsync(object)

异步清除该扩展的全部缓存：

```js
$cache.clearAsync({
  handler: function() {

  }
})
```

---

## 文件内容解读与示例

### 用途说明

`$cache` API 提供了一种**临时性、高性能的数据存储机制**。它允许你的脚本以键值对（Key-Value）的形式快速存储和检索 JavaScript 对象。`$cache` 适用于那些不需要永久保存、但能显著提升脚本运行效率的数据，例如：

-   网络请求的临时结果（避免重复请求）
-   用户界面状态（如滚动位置、上次选择的 Tab）
-   计算结果的缓存（避免重复计算）

### 核心概念

-   **键值对存储**: 数据以字符串 `key` 和任意 JavaScript 对象 `value` 的形式存储。
-   **临时性**: `cache` 中的数据是临时的。虽然在大多数情况下，它会跨脚本运行甚至应用重启而保留，但系统可能会在内存不足或长时间不使用时清除它。因此，**不要用 `$cache` 存储需要永久保存的关键数据**，对于这类数据，应使用 `$file` 或 `$prefs`。
-   **作用域**: `$cache` 是**每个脚本独立**的。一个脚本的缓存不会影响其他脚本的缓存，保证了数据隔离。
-   **同步与异步**: 所有 `$cache` 方法都提供了同步版本（如 `set`）和异步版本（以 `Async` 结尾，如 `setAsync`）。
    -   **同步方法**: 立即执行并返回结果。适用于小数据量或对实时性要求高的场景。
    -   **异步方法**: 通过回调函数或 Promise 返回结果。适用于大数据量操作，或在 UI 脚本中避免阻塞主线程，保持界面流畅。**在 UI 脚本中，推荐优先使用异步方法。**

### 方法详解与示例

#### 1. 写入缓存

-   **`$cache.set(key, value)`**: 同步写入。
-   **`$cache.setAsync(options)`**: 异步写入，`options` 包含 `key`, `value`, `handler`。

**示例**：

```javascript
$cache.set("last_visited_page", "settings");
$cache.setAsync({
  key: "user_profile",
  value: { name: "JSBoxer", level: 10 },
  handler: (success) => {
    if (success) console.log("用户资料异步保存成功。");
  }
});
```

#### 2. 读取缓存

-   **`$cache.get(key)`**: 同步读取，返回缓存的值或 `null`。
-   **`$cache.getAsync(options)`**: 异步读取，`options` 包含 `key`, `handler`。

**示例**：

```javascript
const lastPage = $cache.get("last_visited_page");
if (lastPage) {
  console.log("上次访问的页面:", lastPage);
}

$cache.getAsync({
  key: "user_profile",
  handler: (profile) => {
    if (profile) console.log("异步获取的用户等级:", profile.level);
  }
});
```

#### 3. 移除缓存

-   **`$cache.remove(key)`**: 同步移除指定键的缓存。
-   **`$cache.removeAsync(options)`**: 异步移除指定键的缓存。
-   **`$cache.clear()`**: 同步清除当前脚本的所有缓存。
-   **`$cache.clearAsync(options)`**: 异步清除当前脚本的所有缓存。

**示例**：

```javascript
$cache.remove("last_visited_page");
$cache.clearAsync({ handler: () => console.log("所有缓存已异步清除。") });
```

### 示例代码：一个简单的计数器应用

下面的示例将创建一个简单的计数器，其数值会通过 `$cache` 进行存储，以便在脚本重新运行时恢复上次的值。

```javascript
const COUNTER_KEY = "my_app_counter";
let currentCount = $cache.get(COUNTER_KEY) || 0; // 尝试从缓存读取，否则初始化为0

$ui.render({
  props: { title: "$cache 计数器" },
  views: [
    {
      type: "label",
      props: {
        id: "counter-label",
        text: currentCount.toString(),
        font: $font("bold", 48),
        align: $align.center
      },
      layout: make => {
        make.center.equalTo(view.super);
        make.size.equalTo($size(200, 100));
      }
    },
    {
      type: "button",
      props: { title: "增加计数" },
      layout: make => {
        make.top.equalTo($("counter-label").bottom).offset(20);
        make.centerX.equalTo(view.super);
        make.width.equalTo(120);
      },
      events: {
        tapped: () => {
          currentCount++;
          $("counter-label").text = currentCount.toString();
          // 异步保存当前计数到缓存
          $cache.setAsync({
            key: COUNTER_KEY,
            value: currentCount,
            handler: (success) => {
              if (!success) console.error("计数器保存失败！");
            }
          });
        }
      }
    }
  ]
});
```

**代码解读**：

1.  脚本启动时，首先尝试从 `$cache.get(COUNTER_KEY)` 读取上次保存的计数。如果不存在（首次运行或缓存被清除），则默认为 0。
2.  每次点击按钮增加计数后，我们更新 UI，并通过 `$cache.setAsync()` 异步地将新值保存到缓存中。这里使用异步方法是为了避免在 UI 线程中进行可能的磁盘 I/O 操作，保持界面流畅。

`$cache` 是一个非常实用的工具，用于管理脚本的临时状态和优化性能。但请记住其临时性，对于重要数据，请使用更持久的存储方式。 
