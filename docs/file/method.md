> 提供安全的接口用于操作沙盒内的文件

# $file.read(path)

读取文件：

```js
const file = $file.read("demo.txt");
```

# $file.download(path) -> Promise

当使用 `drive://` 路径时，这个方法先会保证文件被下载下来，然后返回其数据：

```js
const data = await $file.download("drive://.test.db.icloud");
```

# $file.write(object)

写入文件：

```js
const success = $file.write({
  data: $data({string: "Hello, World!"}),
  path: "demo.txt"
});
```

# $file.delete(path)

删除文件：

```js
const success = $file.delete("demo.txt");
```

# $file.list(path)

获取目录下所有文件名：

```js
const contents = $file.list("download");
```

# $file.copy(object)

复制文件：

```js
const success = $file.copy({
  src: "demo.txt",
  dst: "download/demo.txt"
});
```

# $file.move(object)

移动文件：

```js
const success = $file.move({
  src: "demo.txt",
  dst: "download/demo.txt"
});
```

# $file.mkdir(path)

创建文件夹：

```js
const success = $file.mkdir("download");
```

# $file.exists(path)

判断文件/目录是否存在：

```js
const exists = $file.exists("demo.txt");
```

# $file.isDirectory(path)

判断某个路径是否是目录：

```js
const isDirectory = $file.isDirectory("download");
```

# $file.merge(args)

将多个文件合并成一个：

```js
$file.merge({
  files: ["assets/1.txt", "assets/2.txt"],
  dest: "assets/merged.txt",
  chunkSize: 1024 * 1024, // optional, default is 1024 * 1024
});
```

将一个文件分割成多个：

```js
$file.split({
  file: "assets/merged.txt",
  chunkSize: 1024, // optional, default is 1024
});
```

文件将会被分割成 merged-001.txt, merged-002.txt, ...

# $file.absolutePath(path)

返回一个相对路径对应的绝对路径：

```js
const absolutePath = $file.absolutePath(path);
```

# $file.rootPath

返回文档根目录的文件路径（以绝对路径的形式）：

```js
const rootPath = $file.rootPath;
```

# $file.extensions

返回所有安装的 JavaScript extension 文件名：

```js
const extensions = $file.extensions;
```

# shared://

上述示例操作都是在扩展各自的目录下进行文件读写，如果目录以 `shared://` 开头，则读写操作都发生在共享目录，可以被别的扩展读取和覆盖：

```js
const file = $file.read("shared://demo.txt");
```

# drive://

用于读写在 iCloud Drive 目录下的文件和子目录：

```js
const file = $file.read("drive://demo.txt");
```

---

## 文件内容解读与示例

### 用途说明

`$file` API 是 JSBox 中进行**文件系统操作**的核心模块。它提供了一整套安全、高效的方法，用于在脚本的沙盒内部以及通过特殊协议访问共享目录和 iCloud Drive 中的文件和文件夹。无论是读取配置、保存数据、管理下载，`$file` 都是不可或缺的工具。

### 核心概念

-   **路径**: 大多数 `$file` 方法都接受一个 `path` 参数。默认情况下，这些路径是相对于当前脚本的沙盒根目录的。例如，`"my_data.txt"` 指的是当前脚本目录下的 `my_data.txt` 文件。
-   **`$data` 对象**: 文件内容通常以 `$data` 对象（二进制数据）的形式进行读写。当你读取文件时，会得到一个 `$data` 对象；当你写入文件时，需要提供一个 `$data` 对象。
-   **同步与异步**: 大多数 `$file` 操作是同步的（立即返回结果），但像 `$file.download` 这样的操作是异步的，需要使用 `await` 或回调函数来处理。

### 方法分类与示例

#### 1. 读写文件

-   **`$file.read(path)`**: 读取文件内容，返回一个 `$data` 对象。如果文件不存在，返回 `null`。
-   **`$file.write(options)`**: 将 `$data` 写入文件。`options` 包含 `path` 和 `data`。
-   **`$file.download(path)`**: 专门用于 `drive://` 路径，确保文件从 iCloud 下载到本地后返回 `$data`。

**示例**：写入并读取一个文本文件

```javascript
const filePath = "my_document.txt";
const content = "Hello, JSBox file system!\nThis is a test.";

// 写入文件
const writeSuccess = $file.write({
  path: filePath,
  data: $data({ string: content, encoding: 4 }) // encoding: 4 for UTF-8
});
if (writeSuccess) {
  $ui.toast("文件写入成功！");
}

// 读取文件
const readData = $file.read(filePath);
if (readData) {
  console.log("读取到的内容:", readData.string);
} else {
  console.log("文件读取失败或不存在。");
}
```

#### 2. 目录操作

-   **`$file.list(path)`**: 获取指定目录下所有文件和文件夹的名称列表。
-   **`$file.mkdir(path)`**: 创建一个新的文件夹。如果父目录不存在，也会一并创建。
-   **`$file.exists(path)`**: 判断文件或目录是否存在。
-   **`$file.isDirectory(path)`**: 判断指定路径是否是一个目录。

**示例**：创建文件夹并列出内容

```javascript
const folderPath = "my_new_folder";
if (!$file.exists(folderPath)) {
  $file.mkdir(folderPath);
  $ui.toast(`文件夹 '${folderPath}' 已创建。`);
}

// 在新文件夹中创建一些文件
$file.write({ path: `${folderPath}/file1.txt`, data: $data({ string: "Content of file 1" }) });
$file.write({ path: `${folderPath}/file2.json`, data: $data({ string: JSON.stringify({ key: "value" }) }) });

console.log(`'${folderPath}' 目录内容:`, $file.list(folderPath));
```

#### 3. 文件管理

-   **`$file.delete(path)`**: 删除文件或空目录。
-   **`$file.copy(options)`**: 复制文件或目录。`options` 包含 `src`（源路径）和 `dst`（目标路径）。
-   **`$file.move(options)`**: 移动或重命名文件/目录。`options` 包含 `src` 和 `dst`。

**示例**：复制和删除文件

```javascript
$file.write({ path: "temp_file.txt", data: $data({ string: "临时文件内容" }) });
$file.copy({ src: "temp_file.txt", dst: "copied_file.txt" });
console.log("copied_file.txt 是否存在:", $file.exists("copied_file.txt"));

$file.delete("temp_file.txt");
console.log("temp_file.txt 是否存在:", $file.exists("temp_file.txt"));
```

#### 4. 路径与信息

-   **`$file.absolutePath(path)`**: 将相对路径转换为绝对路径。
-   **`$file.rootPath`**: 返回当前脚本的根目录的绝对路径。
-   **`$file.extensions`**: 返回所有已安装的 JSBox 扩展的名称列表。

#### 5. 特殊协议 (`shared://`, `drive://`)

如 `file/design.md` 中所述，这些协议可以与 `$file` 的大多数方法结合使用，以访问共享目录或 iCloud Drive 中的文件。

**示例**：读写共享目录文件

```javascript
const sharedConfigPath = "shared://app_config.json";
const defaultConfig = { version: 1, last_sync: Date.now() };

// 写入共享配置文件
$file.write({
  path: sharedConfigPath,
  data: $data({ string: JSON.stringify(defaultConfig) })
});

// 读取共享配置文件
const readConfig = $file.read(sharedConfigPath);
if (readConfig) {
  console.log("读取到的共享配置:", JSON.parse(readConfig.string));
}
```

### 总结

`$file` API 是 JSBox 脚本进行本地文件管理的核心。熟练掌握其读写、目录和管理方法，并理解 `shared://` 和 `drive://` 等协议的用法，将使你的脚本能够高效地处理和存储数据。 
