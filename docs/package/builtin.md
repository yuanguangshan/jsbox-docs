# 内置模块

为了更方便使用，JSBox 内置了一些常见的模块：

- [underscore](https://underscorejs.org/)
- [moment](https://momentjs.com/)
- [crypto-js](https://github.com/brix/crypto-js)
- [cheerio](https://cheerio.js.org/)
- [lodash](https://lodash.com/)
- [ramba](https://ramdajs.com/)

这些模块无需打包在安装包内部即可被脚本通过 require 使用，例如：

```js
const lodash = require("lodash");
```

请参考模块各自的文档来使用，在本文档中不再赘述。

如果有其他模块的使用需求，可以通过 [browserify](http://browserify.org/) 进行打包，没有 native 依赖并符合 [CommonJS](https://en.wikipedia.org/wiki/CommonJS) 标准的模块一般可以使用。

---

## 文件内容解读与示例

### 用途说明：JSBox 的“开箱即用”模块

本文档介绍了 JSBox 运行时环境中**预先内置的 JavaScript 模块**。这些模块是常用的第三方库，它们被直接集成到 JSBox 内部，开发者可以在自己的脚本中直接通过 `require()` 语句使用它们，而无需手动下载、打包或管理这些库的文件。这极大地简化了开发流程，减少了脚本体积，并提升了脚本的加载速度。

### 核心概念：无需安装，直接引用

-   **全局可用**: 这些内置模块就像 Node.js 环境中的核心模块一样，它们是 JSBox 运行时环境的一部分，而不是你脚本项目的一部分。
-   **`require()` 语法**: 使用标准的 CommonJS `require()` 语法来引入这些模块。

### 内置模块列表与用途简述

JSBox 内置了以下常用模块，它们覆盖了数据处理、日期操作、加密、HTML 解析等多个方面：

-   **`underscore` / `lodash`**: 两个功能相似的实用工具库，提供了大量用于数组、对象、函数等操作的便捷方法，是 JavaScript 开发中的“瑞士🇨🇭军刀”。`lodash` 通常被认为是 `underscore` 的增强版。
-   **`moment`**: 强大的日期和时间处理库，用于解析、格式化、操作和显示日期时间。
-   **`crypto-js`**: 加密算法库，提供了 MD5、SHA 系列哈希、AES 加密/解密等多种密码学算法。
-   **`cheerio`**: 一个轻量级的、类似于 jQuery 的服务器端 HTML 解析器，用于快速、灵活地解析 HTML 字符串并进行 DOM 操作。
-   **`ramda`**: 一个专注于函数式编程的工具库，提供了许多纯函数，有助于编写更简洁、可测试的代码。

### 示例代码：使用 `moment` 和 `lodash`

下面的示例将演示如何在 JSBox 脚本中直接使用 `moment` 库来格式化日期，以及使用 `lodash` 库来处理数组。

```javascript
// 引入内置模块
const moment = require("moment");
const _ = require("lodash");

$ui.render({
  props: { title: "内置模块示例" },
  views: [
    {
      type: "label",
      props: { id: "date-label", lines: 0 },
      layout: make => make.top.left.right.inset(20)
    },
    {
      type: "label",
      props: { id: "array-label", lines: 0 },
      layout: make => make.top.equalTo(view.prev.bottom).offset(20).left.right.inset(20)
    },
    {
      type: "button",
      props: { title: "执行示例" },
      layout: make => make.top.equalTo(view.prev.bottom).offset(20).centerX.equalTo(view.super).width.equalTo(120),
      events: {
        tapped: () => {
          // 使用 moment 格式化当前日期
          const now = moment();
          const formattedDate = now.format("YYYY年MM月DD日 HH:mm:ss");
          $("date-label").text = `当前时间 (Moment.js):\n${formattedDate}`;

          // 使用 lodash 处理数组
          const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
          const evenNumbers = _.filter(numbers, n => n % 2 === 0); // 过滤偶数
          const doubledNumbers = _.map(evenNumbers, n => n * 2); // 翻倍
          const sumOfDoubled = _.sum(doubledNumbers); // 求和

          $("array-label").text = `原始数组: ${JSON.stringify(numbers)}\n偶数: ${JSON.stringify(evenNumbers)}\n翻倍后: ${JSON.stringify(doubledNumbers)}\n总和: ${sumOfDoubled}`;

          $ui.toast("示例已执行！");
        }
      }
    }
  ]
});
```

**代码解读**：

1.  通过 `const moment = require("moment");` 和 `const _ = require("lodash");`，我们直接引入了这两个内置模块。
2.  在按钮的 `tapped` 事件中，我们分别调用了 `moment()` 来获取当前时间并格式化，以及 `_.filter()`、`_.map()`、`_.sum()` 等 `lodash` 方法来对数组进行链式操作。

### 总结

JSBox 的内置模块极大地丰富了脚本的功能，让开发者能够利用成熟的第三方库来解决各种复杂的编程问题，而无需担心依赖管理。当你遇到需要进行日期处理、数据操作、加密解密或 HTML 解析等任务时，请优先考虑使用这些内置模块。 
