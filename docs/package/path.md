# 路径

引入一个脚本时，可以使用绝对路径或相对路径：

例如：

```js
const app = require('./scripts/app');
```

`scripts/` 则是应用根目录 -> scripts 目录下的 app.js 文件，其中 `.js` 可以省略。

路径应用可以跨越各个文件夹，但仅限于当前应用根目录下。

# $file

可以这样读取一个文件：

```js
const file = $file.read('strings/zh-Hans.strings');
```

这会打开 strings 目录下的 zh-Hans.strings 文件。

简而言之，在安装包里面，和文件有关的根目录是应用的根目录。

---

## 文件内容解读与示例

### 用途说明

本文档旨在阐明 JSBox 中**文件路径的解析规则**，尤其是在使用 `require()` 引入模块和 `$file` API 进行文件操作时。理解这些规则对于正确组织脚本文件、引用资源以及确保脚本在不同环境中（如作为安装包或独立文件）都能正常运行至关重要。

### 核心概念：应用根目录与相对路径

1.  **应用根目录**: 每个 JSBox 脚本（无论是单个 `.js` 文件还是 `.box` 安装包）都有一个明确的**应用根目录**。对于单个 `.js` 文件，其所在目录就是根目录；对于 `.box` 安装包，解压后的顶层目录就是根目录。

2.  **相对路径**: 在 JSBox 脚本中，当你使用 `require()` 或 `$file` API 时，默认情况下，所有路径都是相对于**当前脚本文件**的。这意味着：
    -   `./`: 指代当前脚本文件所在的目录。
    -   `../`: 指代当前脚本文件所在目录的上一级目录。

3.  **约定俗成的目录**: `assets/`（存放资源）、`scripts/`（存放模块化脚本）和 `strings/`（存放本地化字符串）是 JSBox 安装包中常见的子目录，方便开发者组织项目。

### 路径在 `require()` 中的应用

`require()` 用于引入其他 JavaScript 模块。路径的写法决定了 JSBox 如何查找这些模块。

-   **相对路径引入**: 
    ```javascript
    // main.js
    const myUtility = require('./scripts/my_utility');
    // 查找路径: main.js 所在目录/scripts/my_utility.js

    // scripts/my_utility.js
    const config = require('../config.json');
    // 查找路径: scripts/my_utility.js 所在目录的上一级目录/config.json
    ```
    **推荐使用相对路径**，因为它能确保模块在项目结构发生变化时仍能正确引用，并且通常与编辑器的自动补全功能配合更好。

-   **省略 `.js` 后缀**: 在 `require()` 中，`.js` 后缀可以省略。

### 路径在 `$file` API 中的应用

`$file` API 用于读写文件、创建目录等操作。其路径解析规则与 `require()` 类似，默认也是相对于**当前脚本文件**的。

-   **读写脚本沙盒内的文件**: 
    ```javascript
    // main.js
    // 写入文件到 main.js 所在目录下的 data.txt
    $file.write({ path: "data.txt", data: $data({ string: "Hello" }) });

    // 读取 assets 目录下的图片
    const image = $file.read("assets/icon.png");
    ```

-   **特殊协议**: 除了相对路径，`$file` API 还支持特殊协议来访问脚本沙盒之外的特定位置，例如：
    -   `shared://`: 访问 JSBox 的共享目录。
    -   `drive://`: 访问 iCloud Drive 中 JSBox 的专属目录。
    -   `inbox://`: 访问通过分享导入的文件。

### 示例代码：文件与模块的路径引用

下面的示例将演示一个结构化的 JSBox 项目中，`main.js` 如何引用 `scripts` 目录下的模块，以及如何读写 `assets` 目录下的文件。

```javascript
// 假设项目结构如下：
// MyProject.box/
// ├── assets/
// │   └── config.json
// └── scripts/
//     └── my_module.js
// └── main.js

// --- main.js --- (项目入口文件)

// 1. 引入 scripts 目录下的模块
// 路径 './scripts/my_module' 是相对于 main.js 所在的目录
const myModule = require('./scripts/my_module');

// 2. 读取 assets 目录下的配置文件
// 路径 'assets/config.json' 是相对于 main.js 所在的目录
const configData = $file.read('assets/config.json');
let config = {};
if (configData) {
  try {
    config = JSON.parse(configData.string);
    console.log("配置已加载:", config);
  } catch (e) {
    console.error("解析 config.json 失败:", e);
  }
}

// 调用模块中的函数
myModule.greet(config.name || "访客");

// --- scripts/my_module.js --- (一个模块文件)

module.exports = {
  greet: (name) => {
    console.log(`Hello, ${name}!`);
    // 3. 在模块内部，读取同级目录下的文件 (假设 scripts/data.txt 存在)
    // 路径 './data.txt' 是相对于 my_module.js 所在的目录
    const moduleData = $file.read('./data.txt');
    if (moduleData) {
      console.log("模块内部数据:", moduleData.string);
    }
  }
};
```

**代码解读**：

1.  在 `main.js` 中，`require('./scripts/my_module')` 使用相对路径引入了 `my_module.js`。这里的 `./` 指的是 `main.js` 所在的目录。
2.  `$file.read('assets/config.json')` 同样使用相对路径，从 `main.js` 所在目录下的 `assets` 文件夹中读取文件。
3.  在 `my_module.js` 内部，`$file.read('./data.txt')` 也是相对路径，但这里的 `./` 指的是 `my_module.js` 所在的 `scripts` 目录。

### 总结

理解 JSBox 的路径解析规则对于正确组织脚本文件和资源至关重要。始终使用**相对路径**是最佳实践，它能确保脚本在项目结构发生变化时仍能正确引用，并提升代码的可移植性。 
