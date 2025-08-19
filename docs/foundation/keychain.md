> 相较于对象缓存，钥匙串以安全的方式存储密码、登录票据等敏感信息。

# $keychain.set(key, value, domain)

写入到钥匙串：

```js
const succeeded = $keychain.set("key", "value", "my.domain");
```

> 当提供 `domain` 时，`key` 应该在您的脚本内唯一。反之，`key` 应该在所有脚本内唯一。

# $keychain.get(key, domain)

从钥匙串中读取：

```js
const item = $keychain.get("key", "my.domain");
```

> 当提供 `domain` 时，`key` 应该在您的脚本内唯一。反之，`key` 应该在所有脚本内唯一。

# $keychain.remove(key, domain)

删除一个钥匙串项目：

```js
const succeeded = $keychain.remove("key", "my.domain");
```

> 当提供 `domain` 时，`key` 应该在您的脚本内唯一。反之，`key` 应该在所有脚本内唯一。

# $keychain.clear(domain)

删除所有的钥匙串项目：

```js
const succeeded = $keychain.clear("my.domain");
```

> `domain` 为必填项。

# $keychain.keys(domain)

获取所有钥匙串键：

```js
const keys = $keychain.keys("my.domain");
```

> `domain` 为必填项。

---

## 文件内容解读与示例

### 用途说明

`$keychain` API 提供了在 iOS 设备上**安全、持久化地存储敏感信息**的能力。它利用了 iOS 系统底层的**钥匙串服务（Keychain Services）**，这是一个专门为存储密码、API 密钥、登录凭证、加密密钥等敏感数据而设计的安全存储区域。与 `$cache` 或 `$prefs` 不同，钥匙串中的数据是加密的，并且通常在应用被卸载后仍然保留，除非用户手动清除。

### 核心概念：安全与 `domain`

1.  **安全性**: 存储在钥匙串中的数据受到系统级别的加密保护，并且只有授权的应用才能访问。这使得 `$keychain` 成为存储用户凭证等敏感信息的首选。

2.  **持久性**: 钥匙串中的数据通常会比应用本身的沙盒数据更持久。即使应用被卸载后重新安装，之前存储在钥匙串中的数据也可能被恢复。

3.  **`domain` (命名空间)**: 这是 `$keychain` API 中一个非常重要的概念。`domain` 是一个字符串，用于为你的钥匙串项目创建**命名空间**。它的作用是：
    -   **隔离**: 确保你的 `key` 在你的 `domain` 内部是唯一的，而不会与其他脚本或应用使用的相同 `key` 发生冲突。
    -   **组织**: 方便你管理和清除属于特定功能或脚本的所有钥匙串项目。
    -   **推荐**: 强烈建议为你的脚本或特定功能使用一个唯一的 `domain`（例如，使用你的脚本的 Bundle ID 或一个反向域名格式的字符串）。

### 方法详解与示例

#### 1. 写入钥匙串

-   **`$keychain.set(key, value, domain)`**: 将 `value` 存储到钥匙串中，并与 `key` 和可选的 `domain` 关联。`value` 可以是字符串、数字、布尔值、数组或对象。

**示例**：存储用户凭证

```javascript
const MY_KEYCHAIN_DOMAIN = "com.myjsboxapp.credentials";

const username = "jsbox_user";
const password = "my_secret_password";

const successUser = $keychain.set("username", username, MY_KEYCHAIN_DOMAIN);
const successPass = $keychain.set("password", password, MY_KEYCHAIN_DOMAIN);

if (successUser && successPass) {
  $ui.toast("凭证已安全保存！");
} else {
  $ui.alert("凭证保存失败！");
}
```

#### 2. 读取钥匙串

-   **`$keychain.get(key, domain)`**: 从钥匙串中读取与 `key` 和 `domain` 关联的值。如果找不到，返回 `null`。

**示例**：读取用户凭证

```javascript
const MY_KEYCHAIN_DOMAIN = "com.myjsboxapp.credentials";

const storedUsername = $keychain.get("username", MY_KEYCHAIN_DOMAIN);
const storedPassword = $keychain.get("password", MY_KEYCHAIN_DOMAIN);

if (storedUsername && storedPassword) {
  $ui.alert(`读取到凭证:\n用户名: ${storedUsername}\n密码: ${storedPassword}`);
} else {
  $ui.toast("未找到保存的凭证。");
}
```

#### 3. 移除钥匙串项目

-   **`$keychain.remove(key, domain)`**: 删除钥匙串中指定 `key` 和 `domain` 的项目。
-   **`$keychain.clear(domain)`**: **清除指定 `domain` 下的所有钥匙串项目**。注意 `domain` 参数是必填的，你不能清除所有域下的项目。

**示例**：清除凭证

```javascript
const MY_KEYCHAIN_DOMAIN = "com.myjsboxapp.credentials";

const successRemove = $keychain.remove("username", MY_KEYCHAIN_DOMAIN);
const successClear = $keychain.clear(MY_KEYCHAIN_DOMAIN);

if (successRemove && successClear) {
  $ui.toast("凭证已清除！");
} else {
  $ui.alert("凭证清除失败！");
}
```

#### 4. 获取所有键

-   **`$keychain.keys(domain)`**: 返回指定 `domain` 下所有存储的 `key` 的数组。`domain` 参数是必填的。

**示例**：列出所有存储的键

```javascript
const MY_KEYCHAIN_DOMAIN = "com.myjsboxapp.credentials";
const allKeys = $keychain.keys(MY_KEYCHAIN_DOMAIN);
console.log("当前域下的所有键:", allKeys);
```

### 总结

`$keychain` 是 JSBox 中处理敏感数据的首选方式。它提供了系统级的安全存储，并且通过 `domain` 参数实现了良好的数据隔离和管理。在存储任何不希望被轻易泄露或丢失的信息时，都应该优先考虑使用 `$keychain`。 
