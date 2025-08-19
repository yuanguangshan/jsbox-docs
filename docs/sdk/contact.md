> 用于与系统通讯录进行交互，例如读取或写入

# $contact.pick(object)

从通讯录中选择一个或多个联系人：

```js
$contact.pick({
  multi: false,
  handler: function(contact) {
    
  }
})
```

其中 `multi` 为 `true` 时表示选择多个联系人。

# $contact.fetch(object)

通过关键字查找某些联系人：

```js
$contact.fetch({
  key: "Ying",
  handler: function(contacts) {

  }
})
```

获得的是一个数组，每个元素支持很多属性，请参考：https://developer.apple.com/documentation/contacts/cncontact

也可以查找某个分组下面的所有联系人：

```js
$contact.fetch({
  group,
  handler: function(contacts) {

  }
})
```

# $contact.create(object)

创建联系人：

```js
$contact.create({
  givenName: "Ying",
  familyName: "Zhong",
  phoneNumbers: {
    "Home": "18000000000",
    "Office": "88888888"
  },
  emails: {
    "Home": "log.e@qq.com"
  },
  handler: function(resp) {

  }
})
```

# $contact.save(object)

将更新后的 contact 对象存储到系统通讯录：

```js
$contact.save({
  contact,
  handler: function(resp) {

  }
})
})
```

# $contact.delete(object)

删除一些联系人：

```js
$contact.delete({
  contacts: contacts
  handler: function(resp) {
    
  }
})
```

# $contact.fetchGroups(object)

获取联系人分组：

```js
var groups = await $contact.fetchGroups();

console.log("name: " + groups[0].name);
```

# $contact.addGroup(object)

添加一个新的分组：

```js
var group = await $contact.addGroup({"name": "Group Name"});
```

# $contact.deleteGroup(object)

删除一个分组：

```js
var groups = await $contact.fetchGroups();

$contact.deleteGroup(groups[0]);
```

# $contact.updateGroup(object)

保存更新后的分组：

```js
var group = await $contact.fetchGroups()[0];
group.name = "New Name";

$contact.updateGroup(group);
```

# $contact.addToGroup(object)

将联系人添加到分组：

```js
$contact.addToGroup({
  contact,
  group
});
```

# $contact.removeFromGroup(object)

将联系人从分组中移除：

```js
$contact.removeFromGroup({
  contact,
  group
});
```

---

## 文件内容解读与示例

### 用途说明

`$contact` API 提供了与 iOS 系统**通讯录应用**进行深度交互的能力。它允许你的脚本读取、创建、修改和删除联系人，以及管理联系人分组。这对于构建快速拨号、联系人管理、或任何需要与用户通讯录集成的自动化工具都非常有用。

### 核心概念：权限与 `contact`/`group` 对象

-   **权限**: 所有通讯录操作都需要用户授权。首次调用相关接口时，系统会弹出权限请求。你的脚本应妥善处理用户拒绝授权的情况。
-   **`contact` 对象**: 代表通讯录中的单个联系人。其结构在 `object/contact.md` 中有详细定义，包含姓名、电话、邮箱、地址等信息。
-   **`group` 对象**: 代表通讯录中的一个联系人分组。其结构也在 `object/contact.md` 中有详细定义，包含 `identifier` 和 `name`。

### 主要功能与方法详解

#### 1. 联系人获取与选择

-   **`$contact.pick(options)`**: 从通讯录中选择一个或多个联系人。`options` 可包含 `multi: true`（允许多选）。
-   **`$contact.fetch(options)`**: 通过关键字 (`key`) 或分组 (`group`) 查找联系人。返回一个 `contact` 对象数组。

**示例**：选择联系人并显示其姓名和电话

```javascript
async function pickAndDisplayContact() {
  const contact = await $contact.pick(); // 默认单选
  if (contact) {
    const fullName = `${contact.familyName || ''}${contact.givenName || ''}`;
    const phoneNumber = contact.phoneNumbers && contact.phoneNumbers.length > 0 
                        ? contact.phoneNumbers[0].value : '无电话';
    $ui.alert(`你选择了: ${fullName}\n电话: ${phoneNumber}`);
  } else {
    $ui.toast("未选择联系人或操作取消。");
  }
}
// pickAndDisplayContact();
```

#### 2. 联系人增删改

-   **`$contact.create(options)`**: 创建新联系人。`options` 包含 `contact` 对象的属性。
-   **`$contact.save(options)`**: 更新现有联系人。传入一个已修改的 `contact` 对象。
-   **`$contact.delete(options)`**: 删除联系人。传入一个或多个 `contact` 对象。

**示例**：创建一个新联系人

```javascript
async function createNewContact() {
  const resp = await $contact.create({
    givenName: "JSBox",
    familyName: "测试",
    phoneNumbers: [{ value: "13812345678", label: "mobile" }],
    emails: [{ value: "test@jsbox.com", label: "work" }],
    organizationName: "JSBox Team",
    jobTitle: "Developer"
  });
  if (resp && resp.contact) {
    $ui.toast("联系人创建成功！");
  } else if (resp && resp.error) {
    $ui.alert(`创建失败: ${resp.error.localizedDescription}`);
  } else {
    $ui.alert("联系人创建失败或无权限。");
  }
}
// createNewContact();
```

#### 3. 联系人分组管理

-   **`$contact.fetchGroups()`**: 获取所有联系人分组。返回 `group` 对象数组。
-   **`$contact.addGroup(options)`**: 创建新分组。`options` 包含 `name`。
-   **`$contact.deleteGroup(options)`**: 删除分组。传入一个 `group` 对象。
-   **`$contact.updateGroup(options)`**: 更新分组。传入一个已修改的 `group` 对象。
-   **`$contact.addToGroup(options)`**: 将联系人添加到分组。`options` 包含 `contact` 和 `group`。
-   **`$contact.removeFromGroup(options)`**: 将联系人从分组中移除。

**示例**：创建分组并添加联系人

```javascript
async function createGroupAndAddContact() {
  const newGroup = await $contact.addGroup({ name: "JSBox Friends" });
  if (newGroup) {
    $ui.toast(`分组 "${newGroup.name}" 已创建！`);
    const contact = await $contact.pick(); // 让用户选择一个联系人
    if (contact) {
      const resp = await $contact.addToGroup({ contact: contact, group: newGroup });
      if (resp && resp.success) {
        $ui.toast(`联系人 "${contact.givenName}" 已添加到分组。`);
      } else {
        $ui.alert("添加联系人到分组失败。");
      }
    }
  } else {
    $ui.alert("创建分组失败或无权限。");
  }
}
// createGroupAndAddContact();
```

### 总结

`$contact` API 为你的 JSBox 脚本提供了与 iOS 通讯录进行深度集成的能力。无论是构建快速拨号、联系人管理、或任何需要与用户通讯录集成的自动化工具，它都是不可或缺的。 
