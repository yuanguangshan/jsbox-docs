# contact

`contact` 是使用 `$contact` 接口时返回的数据对象。

属性 | 类型 | 读写 | 说明
---|---|---|---
identifier | string | 只读 | 标识符
content | string | 只读 | 内容
contactType | number | 只读 | 类型
namePrefix | string | 读写 | name prefix
givenName | string | 读写 | given name
middleName | string | 读写 | middle name
familyName | string | 读写 | family name
nameSuffix | string | 读写 | name suffix
nickname | string | 读写 | nick name
organizationName | string | 读写 | 组织
departmentName | string | 读写 | 部门
jobTitle | string | 读写 | 职位
note | string | 读写 | 备注
imageData | data | 读写 | 头像
phoneNumbers | array | 读写 | 电话
emailAddresses | array | 读写 | 邮件
postalAddresses | array | 读写 | 邮编
urlAddresses | array | 读写 | 链接
instantMessageAddresses | array | 读写 | 即时通讯

# group

`group` 是使用 `$contact.fetchGroup` 接口时返回的对象

属性 | 类型 | 读写 | 说明
---|---|---|---
identifier | string | 只读 | 标识符
name | string | 读写 | 名称

---

## 文件内容解读与示例

### 用途说明

`contact` 对象是 JSBox 中用于表示设备通讯录中**单个联系人**的数据结构。而 `group` 对象则代表一个**联系人分组**。它们是 `$contact` API 返回的核心数据类型，允许你的脚本读取、创建、修改和管理用户的通讯录信息。

### `contact` 对象详解

`contact` 对象封装了一个联系人的所有详细信息，包括姓名、电话、邮箱、地址、组织信息等。大多数属性都是可读写的，这意味着你可以修改这些对象，并通过 `$contact` API 将更改保存回通讯录。

#### 1. 身份标识与类型

-   **`identifier`**: 只读，联系人的唯一标识符。
-   **`contactType`**: 只读，联系人类型（例如，个人或组织）。

#### 2. 姓名信息

-   **`namePrefix`**: 姓名前缀（如“Mr.”）。
-   **`givenName`**: 名字。
-   **`middleName`**: 中间名。
-   **`familyName`**: 姓氏。
-   **`nameSuffix`**: 姓名后缀（如“Jr.”）。
-   **`nickname`**: 昵称。

#### 3. 组织与备注

-   **`organizationName`**: 组织名称。
-   **`departmentName`**: 部门名称。
-   **`jobTitle`**: 职位。
-   **`note`**: 联系人备注。

#### 4. 头像

-   **`imageData`**: 联系人头像的二进制数据，一个 `$data` 对象。可读写。

#### 5. 多值属性 (数组)

这些属性都是数组，因为一个联系人可以有多个电话号码、邮箱地址等。数组中的每个元素通常是一个包含 `value` 和 `label` 的对象（例如 `{ value: "13800138000", label: "mobile" }`）。

-   **`phoneNumbers`**: 电话号码数组。
-   **`emailAddresses`**: 邮箱地址数组。
-   **`postalAddresses`**: 邮政地址数组。
-   **`urlAddresses`**: 网址数组。
-   **`instantMessageAddresses`**: 即时通讯地址数组。

### `group` 对象详解

`group` 对象代表通讯录中的一个联系人分组。

-   **`identifier`**: 只读，分组的唯一标识符。
-   **`name`**: 分组的名称。

### 示例代码：显示选定联系人的信息

下面的示例将创建一个简单的 UI，让用户从通讯录中选择一个联系人，然后显示其姓名、第一个电话号码和第一个邮箱地址。

```javascript
async function displayContactInfo() {
  // 1. 弹出联系人选择器，让用户选择一个联系人
  const contact = await $contact.pick(); // $contact.pick() 返回一个 contact 对象

  if (contact) {
    // 2. 渲染 UI 来显示联系人信息
    $ui.render({
      props: { title: "联系人详情" },
      views: [
        {
          type: "label",
          props: { text: `姓名: ${contact.familyName || ''}${contact.givenName || ''}` },
          layout: make => make.top.left.right.inset(10)
        },
        {
          type: "label",
          props: { text: `昵称: ${contact.nickname || '无'}` },
          layout: make => make.top.equalTo(view.prev.bottom).offset(5).left.right.inset(10)
        },
        {
          type: "label",
          props: { text: `电话: ${contact.phoneNumbers && contact.phoneNumbers.length > 0 ? contact.phoneNumbers[0].value : '无'}` },
          layout: make => make.top.equalTo(view.prev.bottom).offset(5).left.right.inset(10)
        },
        {
          type: "label",
          props: { text: `邮箱: ${contact.emailAddresses && contact.emailAddresses.length > 0 ? contact.emailAddresses[0].value : '无'}` },
          layout: make => make.top.equalTo(view.prev.bottom).offset(5).left.right.inset(10)
        },
        {
          type: "label",
          props: { text: `组织: ${contact.organizationName || '无'}` },
          layout: make => make.top.equalTo(view.prev.bottom).offset(5).left.right.inset(10)
        },
        {
          type: "label",
          props: { text: `备注: ${contact.note || '无'}`, lines: 0 },
          layout: make => make.top.equalTo(view.prev.bottom).offset(5).left.right.inset(10)
        }
      ]
    });
  } else {
    $ui.toast("未选择联系人。");
  }
}

// 启动脚本时调用函数
displayContactInfo();
```

**代码解读**：

1.  脚本首先调用 `await $contact.pick()` 来弹出系统联系人选择器，等待用户选择一个联系人。**请注意，这需要用户授权访问通讯录权限。**
2.  如果用户成功选择了一个联系人，`contact` 变量就会被填充为一个 `contact` 对象。
3.  然后，我们通过访问 `contact` 对象的各种属性（如 `familyName`, `givenName`, `phoneNumbers[0].value` 等）来获取联系人的详细信息，并将其显示在 UI 的 `label` 组件中。
4.  对于 `phoneNumbers` 和 `emailAddresses` 这样的数组属性，我们首先检查数组是否存在且不为空，以避免访问不存在的索引导致错误。

`contact` 和 `group` 对象为 JSBox 脚本提供了与 iOS 通讯录进行深度集成的能力，使得你可以轻松地构建联系人管理、快速拨号或信息发送等自动化工具。 
