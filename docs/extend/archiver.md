> JSBox 自带的压缩/解压缩模块

# $archiver.zip(object)

将多个文件压缩成一个 ZIP 文件：

```js
var files = $context.dataItems
var dest = "Archive.zip"

if (files.length == 0) {
  return
}

$archiver.zip({
  files: files,
  dest: dest,
  handler: function(success) {
    if (success) {
      $share.sheet([dest, $file.read(dest)])
    }
  }
})
```

`files` 是一个 data 数组，也可以通过 `paths` 指定一组文件。

你也可以通过 directory 参数来对一个文件夹进行压缩：

```js
$archiver.zip({
  directory: "",
  dest: "",
  handler: function(success) {
    
  }
})
```

# $archiver.unzip(object)

将一个 ZIP 文件解压到某个路径下：

```js
$archiver.unzip({
  file,
  dest: "folder",
  handler: function(success) {

  }
})
```

file 是一个 zip 的 data 对象，dest 所指向的目录需要存在，否则将会失败。

从 2.0 开始，你也可以使用 `path` 来指定需要解压的 zip 文件位置，这种方式使用更少的内存：

```js
const success = await $archiver.unzip({
  path: "archive.zip",
  dest: "folder"
});
```

---

## 文件内容解读与示例

### 用途说明

`$archiver` API 提供了在 JSBox 脚本中进行文件**压缩（zip）**和**解压缩（unzip）**的功能。这对于需要打包多个文件、备份数据、或者处理下载的压缩包等场景非常有用。例如，你可以用它来导出你的脚本项目，或者解压一个包含图片、HTML 文件的资源包。

### `$archiver.zip(options)` - 压缩文件或文件夹

- **功能**: 将指定的文件或整个文件夹打包成一个 `.zip` 格式的压缩文件。
- **参数**: `options` 是一个对象，至少需要包含 `dest`（目标 ZIP 文件路径）和一个源（`files`, `paths` 或 `directory`）。
  - `files`: 一个 `$data` 对象的数组，每个 `$data` 对象代表一个文件的内容。
  - `paths`: 一个字符串数组，每个字符串是文件的完整路径。
  - `directory`: 一个字符串，表示要压缩的**文件夹路径**。这会压缩整个文件夹及其内容。
  - `dest`: 压缩后生成的 `.zip` 文件的保存路径。
  - `handler`: 压缩操作完成后的回调函数，接收一个布尔值 `success`。

**示例：压缩一个文件夹**

```javascript
// 假设你的脚本目录下有一个名为 "my_folder" 的文件夹
// 确保这个文件夹存在，并且里面有一些文件

$archiver.zip({
  directory: "my_folder", // 要压缩的文件夹路径
  dest: "my_archive.zip", // 压缩后保存为 my_archive.zip
  handler: (success) => {
    if (success) {
      $ui.alert("文件夹压缩成功！文件已保存到 my_archive.zip");
      // 压缩成功后，你可以选择分享这个文件
      // $share.sheet($file.read("my_archive.zip"));
    } else {
      $ui.alert("文件夹压缩失败。");
    }
  }
});
```

### `$archiver.unzip(options)` - 解压缩文件

- **功能**: 将一个 `.zip` 压缩文件解压到指定的目录下。
- **参数**: `options` 是一个对象，至少需要包含 `dest`（解压目标路径）和一个源（`file` 或 `path`）。
  - `file`: 一个 `$data` 对象，代表要解压的 `.zip` 文件内容。
  - `path`: 一个字符串，代表要解压的 `.zip` 文件在本地的路径。**推荐使用此方式，因为它通常更节省内存。**
  - `dest`: 解压目标文件夹的路径。**注意：这个目标文件夹必须提前存在，否则解压会失败。**
  - `handler`: 解压操作完成后的回调函数，接收一个布尔值 `success`。

**示例：解压一个本地的 ZIP 文件**

```javascript
// 假设你已经有一个名为 "downloaded_archive.zip" 的文件在脚本目录下
// 并且你希望解压到一个名为 "extracted_content" 的文件夹中

// 1. 确保目标文件夹存在
const destFolder = "extracted_content";
if (!$file.exists(destFolder)) {
  $file.mkdir(destFolder);
}

// 2. 执行解压操作
$archiver.unzip({
  path: "downloaded_archive.zip", // 要解压的 ZIP 文件路径
  dest: destFolder, // 解压到这个文件夹
  handler: (success) => {
    if (success) {
      $ui.alert(`文件已成功解压到 ${destFolder}/`);
      // 解压成功后，你可以列出解压后的文件
      // console.log($file.list(destFolder));
    } else {
      $ui.alert("文件解压失败。请检查目标文件夹是否存在或 ZIP 文件是否损坏。");
    }
  }
});
```

### 总结

`$archiver` API 为 JSBox 脚本提供了强大的文件压缩和解压缩能力。在使用时，请注意以下几点：

-   **异步操作**: 压缩和解压缩都是耗时操作，它们是异步的，因此你需要使用 `handler` 回调函数来处理操作完成后的逻辑。
-   **目标路径**: 解压时，请务必确保 `dest` 目标文件夹已经存在，否则操作会失败。

通过这些方法，你可以轻松地在 JSBox 中实现各种文件打包和解包的自动化任务。 
