> $drive 用于操作 iCloud Drive 下的文件，与 $file 高度类似

# iCloud Drive

JSBox 支持 iCloud Drive 文件的操作，主要操作有三种类型：

- `$drive.open` 让用户选择文件
- `$drive.save` 让用户存储文件
- `$drive.read` 等和 `$file.read` 一致的接口

# $drive.open

打开 iOS 的文件选择器让用户选择一个文件：

```js
$drive.open({
  handler: function(data) {
    
  }
})
```

可以传入 `types` 用于限定文件 UTI:

```js
types: ["public.png"]
```

也可以通过指定 `multi` 来允许多选：

```js
const files = await $drive.open({ multi: true });
```

在这种情况下，返回的 `files` 是一个 data 的数组。

# $drive.save

打开 iOS 文件选择器让用户把文件存储到 iCloud Drive:

```js
$drive.save({
  data: $data({string: "Hello, World!"}),
  name: "File Name",
  handler: function() {

  }
})
```

# $drive.read

此接口效果与 `$file.read` 完全相同：

```js
const file = $drive.read("demo.txt");
```

不同之处是在于这个接口会从 JSBox 的 iCloud Drive 文件夹读取相应的子目录。

另外，此操作等价于：

```js
const file = $file.read("drive://demo.txt");
```

# $drive.xxx

同样的，`$drive` 也提供了类似 `$file` 的其他方法：

- `$drive.read` vs [$file.read](file/method.md?id=filereadpath)
- `$drive.download` vs [$file.download](file/method.md?id=filedownloadpath)
- `$drive.write` vs [$file.write](file/method.md?id=filewriteobject)
- `$drive.delete` vs [$file.delete](file/method.md?id=filedeletepath)
- `$drive.list` vs [$file.list](file/method.md?id=filelistpath)
- `$drive.copy` vs [$file.copy](file/method.md?id=filecopyobject)
- `$drive.move` vs [$file.move](file/method.md?id=filemoveobject)
- `$drive.mkdir` vs [$file.mkdir](file/method.md?id=filemkdirpath)
- `$drive.exists` vs [$file.exists](file/method.md?id=fileexistspath)
- `$drive.isDirectory` vs [$file.isDirectory](file/method.md?id=fileisdirectorypath)
- `$drive.absolutePath` vs [$file.absolutePath](file/method.md?id=fileabsolutepath)

---

## 文件内容解读与示例

### 用途说明

`$drive` API 专门用于**与 iCloud Drive 及其它云存储服务（通过 iOS 文件 App）进行交互**。它允许你的脚本通过调用系统提供的文件选择器（Document Picker）来让用户选择文件，或者将脚本生成的文件保存到 iCloud Drive。这对于实现跨设备数据同步、备份或与云端文件协作的脚本非常有用。

### 核心概念：系统文件选择器与 JSBox 专属目录

`$drive` 的功能可以分为两类：

1.  **通过系统文件选择器与用户交互**：`$drive.open()` 和 `$drive.save()` 会弹出 iOS 系统提供的文件选择界面，用户可以在其中选择 iCloud Drive、本地设备或其他已连接的云服务（如 Dropbox、Google Drive）。这种方式的优点是用户拥有完全的控制权和隐私保障。

2.  **直接读写 JSBox 在 iCloud Drive 中的专属目录**：`$drive.read()`、`$drive.write()` 等方法，它们与 `$file` 模块中的同名方法类似，但操作的路径默认是 `drive://` 协议下的文件，即 JSBox 在 iCloud Drive 中创建的专属文件夹（通常是 `iCloud Drive/JSBox/`）。

### 核心方法详解

#### 1. `$drive.open(options)`: 让用户选择文件

-   **用途**: 弹出系统文件选择器，让用户从 iCloud Drive 或其他云服务中选择一个或多个文件。
-   **参数**: 
    -   `types`: 数组，用于限定用户可以选择的文件类型（使用 UTI，如 `"public.text"`, `"public.image"`）。
    -   `multi`: 布尔值，如果为 `true`，用户可以多选文件，返回结果将是 `$data` 对象的数组。
-   **返回值**: 一个 Promise，成功时解析为选中的文件的 `$data` 对象（或数组）。

**示例**：让用户选择一个文本文件

```javascript
async function openTextFileFromDrive() {
  try {
    const fileData = await $drive.open({
      types: ["public.text"], // 只允许选择文本文件
      multi: false // 只允许选择一个文件
    });

    if (fileData) {
      $ui.alert(`文件内容:\n${fileData.string}`);
    } else {
      $ui.toast("未选择文件。");
    }
  } catch (error) {
    $ui.alert(`打开文件失败: ${error}`);
  }
}
// openTextFileFromDrive();
```

#### 2. `$drive.save(options)`: 让用户保存文件

-   **用途**: 弹出系统文件保存器，让用户选择一个位置来保存脚本生成的文件。
-   **参数**: 
    -   `data`: 要保存的文件的 `$data` 对象。
    -   `name`: 建议的文件名。
-   **返回值**: 一个 Promise，成功时解析为 `true`。

**示例**：保存一个文本文件到 iCloud Drive

```javascript
async function saveTextFileToDrive() {
  const content = "这是要保存到 iCloud Drive 的文本内容。";
  const fileData = $data({ string: content });

  try {
    const success = await $drive.save({
      data: fileData,
      name: "MyJSBoxDocument.txt"
    });

    if (success) {
      $ui.toast("文件已成功保存到 iCloud Drive！");
    } else {
      $ui.toast("文件保存已取消或失败。");
    }
  } catch (error) {
    $ui.alert(`保存文件失败: ${error}`);
  }
}
// saveTextFileToDrive();
```

#### 3. `$drive.read(path)` 及其他 `$drive.xxx` 方法

-   **用途**: 这些方法（如 `read`, `write`, `delete`, `list`, `exists`, `mkdir` 等）与 `$file` 模块中的同名方法功能一致，但它们操作的路径是 JSBox 在 iCloud Drive 中的专属文件夹（即 `drive://` 协议下的路径）。

**示例**：直接读取 JSBox iCloud 文件夹中的文件

```javascript
// 假设在 iCloud Drive/JSBox/ 目录下有一个名为 "config.json" 的文件
const configPath = "config.json"; // 相当于 drive://config.json

if ($drive.exists(configPath)) {
  const configData = $drive.read(configPath);
  if (configData) {
    $ui.alert(`读取到 iCloud 配置:\n${configData.string}`);
  }
} else {
  $ui.alert("iCloud Drive/JSBox/config.json 不存在。");
}
```

### 总结

`$drive` API 是实现 JSBox 脚本云端文件管理的关键。`$drive.open()` 和 `$drive.save()` 提供了与用户交互的系统级文件选择/保存界面，而 `$drive.read()` 等方法则允许脚本直接操作其在 iCloud Drive 中的专属目录。 
