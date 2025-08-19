# JSBox 安装包格式

从 `v1.9.0` 开始，JSBox 不仅支持运行单个 JavaScript 文件，还设计了自己的安装包格式，可以带来如下好处：

- 可以将代码模块化，不需要将所有逻辑都放在同一个文件
- 可以将配置文件化，这样更容易维护
- 可以内置图片等资源，从而不需要通过 Base64 等手段引入图片

# 格式说明

JSBox 安装包格式是一个 zip 文件，只要文件目录包含以下结构则会被认为是一个 JSBox 安装包：

- **assets/** 用于存放资源
- **scripts/** 用于存放脚本
- **strings/** 用于存放本地化字符串
- **config.json** 安装包配置文件
- **main.js** 脚本主要入口，在这里加载其他脚本

这是一个样例：https://github.com/cyanzhong/xTeko/tree/master/extension-demos/package

你也可以通过 JSBox 应用直接创建一个应用模板。

# assets

将应用将用到的资源文件放置于这里，然后可以通过 `assets/filename` 样式的路径引用文件。

这些文件可以被用在 `$file` 相关接口，也可以作为 `src` 被设置为 image 或 button 等组件。

该文件夹默认包含一个 `icon.png`，这将作为应用的图标，在设置里面切换图标将会替换这个文件，你也可以手动替换这个文件以实现自定义图标。

图标制作标准请参考：https://github.com/cyanzhong/xTeko/tree/master/extension-icons

# scripts

将会用到的脚本放到这个文件夹下，可以通过 `require('scripts/utility')` 样式的代码模块化的加载文件，我们稍后会介绍如何模块化。

你可以在这个文件夹下面建立子文件夹，将脚本放到合适的位置，从而实现整个应用的架构划分。

# strings

这个文件夹提供的功能与 `$app.strings` 类似，即为 `$l10n` 提供本地化的字符串，默认包含：

- **en.strings** 英文
- **zh-Hans.strings** 中文简体

strings 文件是一个如下格式的文本文件（键值对）：

```
"OK" = "好的";
"DONE" = "完成";
"HELLO_WORLD" = "你好，世界！";
```

PS: 如果你同时使用了 strings 文件和 $app.strings，将以后者为准。

# config.json

此文件目前包含两个部分，一个是安装文件的元信息，位于 `info` 节点下面，请参考：[$addin](addin/method.md?id=addinlist)。

另一部分是 `settings` 节点，这部分设置与 `$app` 接口相关的一些设置相同，请参考：[$app](foundation/app.md?id=appminsdkver)。

使用 config.json 能更好的组织你的设置项，以后新增的设置项也会被放到这里。

# main.js

当一个应用被执行的时候，main.js 将是入口，在这里我们可以加载 scripts 文件夹里面的脚本，例如：

```js
const app = require('./scripts/app');

app.sayHello();
```

这将会以模块的形式引入 `scripts/app.js` 这个文件，然后调用该文件的 `sayHello` 函数，我们将稍后介绍这个机制。

---

## 文件内容解读与示例

### 用途说明

本文档详细介绍了 JSBox 的**自定义安装包格式**。从 `v1.9.0` 开始，JSBox 不再仅仅支持运行单个 JavaScript 文件，而是允许开发者将复杂的脚本项目组织成一个结构化的包（本质上是一个 `.zip` 文件，但通常以 `.box` 结尾）。这种格式带来了诸多好处，是开发大型、功能丰富且易于维护的 JSBox 脚本的关键。

### 核心概念：结构化打包的优势

JSBox 的安装包格式鼓励良好的项目组织和模块化，从而实现：

-   **代码模块化**: 将复杂的逻辑拆分成多个独立的 JavaScript 文件，提高代码的可读性、可维护性和复用性。
-   **资源管理**: 方便地将图片、音频、字体等非代码资源内置到脚本包中，无需通过 Base64 等方式嵌入代码，使资源管理更清晰。
-   **统一配置**: 通过 `config.json` 文件统一管理脚本的元信息和用户设置，简化了配置流程。
-   **多语言支持**: 通过 `strings/` 文件夹提供多语言文本，方便脚本的本地化。

### 安装包目录结构详解

一个标准的 JSBox 安装包（`.box` 或 `.zip` 文件解压后）通常包含以下核心目录和文件：

```
MyAwesomeScript.box/ (或 MyAwesomeScript.zip 解压后)
├── assets/         # 存放所有非代码资源
├── scripts/        # 存放所有模块化的 JavaScript 代码文件
├── strings/        # 存放本地化字符串文件
├── config.json     # 脚本的元信息和应用级设置
└── main.js         # 脚本的主要入口文件
```

#### 1. `assets/` (资源文件)

-   **用途**: 存放脚本中使用的所有非代码资源，如图片（`icon.png`）、音频、视频、字体等。
-   **引用方式**: 在脚本中通过 `assets/filename.ext` 这样的相对路径引用。例如，`$image("assets/background.jpg")`。
-   **`icon.png`**: 默认的应用图标文件。你可以在设置中切换图标，或手动替换此文件来自定义脚本图标。

#### 2. `scripts/` (脚本文件)

-   **用途**: 存放所有模块化的 JavaScript 代码文件。你可以根据功能将代码拆分成多个文件，并在 `scripts/` 目录下创建子文件夹进行更细致的组织。
-   **引用方式**: 使用 CommonJS 规范的 `require()` 语法来加载其他脚本文件。例如，`const utils = require('scripts/utils');`。

#### 3. `strings/` (本地化字符串)

-   **用途**: 存放多语言文本文件，用于 `$l10n` API 实现脚本的本地化。每个语言对应一个 `.strings` 文件（如 `en.strings`, `zh-Hans.strings`）。
-   **格式**: `.strings` 文件是简单的键值对文本文件，例如 `"HELLO_MESSAGE" = "Hello, World!";`。
-   **优先级**: 如果同时在 `strings/` 文件夹和 `$app.strings` 中定义了相同的键，`$app.strings` 中的定义将优先。

#### 4. `config.json` (配置文件)

-   **用途**: 脚本的元信息和应用级设置的统一管理文件。
-   **结构**: 包含 `info` 节点（脚本名称、版本、作者等元信息，参考 `$addin` 文档）和 `settings` 节点（与 `$app` 接口相关的设置，参考 `$prefs` 文档）。

#### 5. `main.js` (主入口文件)

-   **用途**: 脚本执行的起始点。当用户运行脚本时，JSBox 会首先执行 `main.js`。你通常会在这里引入 `scripts/` 目录下的其他模块，并启动脚本的主要逻辑。

### 示例：一个结构化的 JSBox 脚本项目

假设你有一个名为 `MyProject.box` 的脚本包，其内部结构和 `main.js` 的内容可能如下：

```
MyProject.box/
├── assets/
│   ├── icon.png
│   └── background.jpg
├── scripts/
│   ├── utils.js
│   └── api.js
├── strings/
│   ├── en.strings
│   └── zh-Hans.strings
├── config.json
└── main.js
```

**`main.js` 内容示例**：

```javascript
// main.js

// 引入 scripts 目录下的模块
const utils = require('./scripts/utils'); // 注意相对路径
const api = require('./scripts/api');

// 从 config.json 或 $prefs 读取设置
const scriptName = $addin.current.name; // 获取当前脚本名称
const userName = $prefs.get("user_name") || "访客"; // 从 prefs.json 读取用户设置

// 使用本地化字符串
const greetingMessage = $l10n("HELLO_MESSAGE"); // 从 strings/ 文件夹获取本地化文本

$ui.render({
  props: {
    title: scriptName
  },
  views: [
    {
      type: "image",
      props: {
        image: $image("assets/icon.png"), // 引用 assets 目录下的图片
        contentMode: $contentMode.scaleAspectFit
      },
      layout: make => {
        make.centerX.equalTo(view.super);
        make.top.inset(50);
        make.size.equalTo($size(100, 100));
      }
    },
    {
      type: "label",
      props: {
        text: `${greetingMessage}, ${userName}!`, // 使用本地化问候语和用户设置
        font: $font("bold", 24),
        align: $align.center
      },
      layout: make => {
        make.centerX.equalTo(view.super);
        make.top.equalTo(view.prev.bottom).offset(30);
      }
    }
  ]
});

// 调用其他模块的功能
utils.logCurrentTime();
api.fetchSomeData();
```

**`config.json` 内容示例**：

```json
{
  "info": {
    "name": "我的项目",
    "version": "1.0.0",
    "author": "你的名字",
    "description": "一个演示 JSBox 包结构的脚本。"
  },
  "settings": {
    "groups": [
      {
        "title": "通用",
        "items": [
          {
            "title": "用户名",
            "type": "string",
            "key": "user_name",
            "value": "访客"
          }
        ]
      }
    ]
  }
}
```

**`strings/en.strings` 内容示例**：

```
"HELLO_MESSAGE" = "Hello";
```

**`strings/zh-Hans.strings` 内容示例**：

```
"HELLO_MESSAGE" = "你好";
```

### 总结

JSBox 的安装包格式是开发复杂、可维护脚本的基石。它鼓励良好的项目组织，并充分利用了 JSBox 内置的资源管理、模块化、本地化和配置功能。熟练掌握这种格式，将使你的 JSBox 开发体验更上一层楼。 
