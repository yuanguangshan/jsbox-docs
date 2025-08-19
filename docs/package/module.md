# require('path')

`require` 这个方法的机制是将被引入的脚本文件变成一个模块，从而有自己的命名空间，可以解决变量名暴露的问题。

需要模块化的文件需要使用 `module.exports`:


```js
function sayHello() {
  $ui.alert($l10n('HELLO_WORLD'));
}

module.exports = {
  sayHello
}
```

sayHello 原本是不能被外部访问的，你可以通过配置 module.exports 来为其提供一个在外部引用时的命名。

所以引入的时候则可以使用：

```js
const app = require('./scripts/app');

app.sayHello();
```

这个机制将各个文件的代码良好的隔离了开来，并且不会出现命名冲突。

注意：请尽可能的使用相对路径，自动补全机制对于绝对路径引入的模块不起作用。

# $include('path')

另外一个方式是使用 `$include` 方式来引入代码，这个机制与 require 完全不同，他不会产生模块化的效果，而仅仅是将代码从一个文件中引入过来，并且执行一遍，效果等同于 eval 一段源码。

通过以上两种方式我们可以有效地组织源码，从而做出架构更好的设计。

---

## 文件内容解读与示例

### 用途说明

本文档详细介绍了 JSBox 的**模块化机制**，这是组织和管理大型脚本项目的关键。它引入了 `require()` 和 `module.exports` 这对核心组合，用于实现代码的封装和复用，并与 `$include()` 进行了对比，帮助开发者编写更清晰、更易维护的代码。

### 核心概念：模块化编程

在 JavaScript 中，模块化是一种将代码拆分成独立、可复用单元的方法。每个模块都有自己的私有作用域，只暴露（导出）需要对外提供的功能。这解决了传统 JavaScript 中所有代码都在全局作用域下运行，容易导致变量名冲突和代码难以管理的问题。

### `require()` 与 `module.exports`：模块化编程的核心

这是 JSBox 中实现模块化的标准方式，遵循 CommonJS 规范。

1.  **`module.exports`**: 
    -   **用途**: 在模块内部，`module.exports` 是一个特殊的对象，用于定义该模块要**对外暴露**（导出）哪些函数、变量或对象。只有被 `module.exports` 导出的内容，才能在其他文件中被 `require()` 引入。
    -   **示例**：`my_math.js` 模块
        ```javascript
        // scripts/my_math.js
        let PI = 3.14159; // 这是一个私有变量，外部无法直接访问

        function add(a, b) {
          return a + b;
        }

        function subtract(a, b) {
          return a - b;
        }

        // 导出 add 和 subtract 函数，以及一个常量
        module.exports = {
          add: add,
          subtract: subtract,
          version: "1.0"
        };
        ```

2.  **`require(path)`**: 
    -   **用途**: 在一个文件中引入另一个模块。`path` 是相对于当前脚本的模块文件路径（例如 `./scripts/my_math`）。
    -   **返回值**: `require()` 返回的是被引入模块的 `module.exports` 对象。
    -   **示例**：在 `main.js` 中使用 `my_math.js`
        ```javascript
        // main.js
        const math = require('./scripts/my_math'); // 引入模块

        console.log("5 + 3 =", math.add(5, 3)); // 调用导出的函数
        console.log("模块版本:", math.version); // 访问导出的属性
        // console.log(math.PI); // 错误：PI 未导出，是私有变量
        ```

**优势**: `require()` 和 `module.exports` 机制实现了代码隔离，避免了全局变量污染和命名冲突，提高了代码的组织性和可维护性。

### `$include(path)`：简单的代码注入

`$include` 的机制与 `require` 完全不同。它不会产生模块化的效果，而仅仅是将指定文件中的代码**直接复制粘贴**到当前文件的位置，并立即执行。

-   **特点**: 
    -   **无模块化**: 被 `$include` 的文件中的所有变量都会进入当前文件的全局作用域，可能导致命名冲突。
    -   **无返回值**: `$include` 没有返回值。
-   **适用场景**: 适用于简单的代码片段复用，或者在调试时临时注入代码。应谨慎使用以避免全局变量污染。

**示例**：

```javascript
// scripts/constants.js
const APP_NAME = "My App";
const VERSION = "1.0";

// main.js
$include('./scripts/constants.js');

console.log("应用名称:", APP_NAME); // APP_NAME 在这里是可用的
console.log("版本:", VERSION);
```

### 示例代码：模块化 UI 与数据处理

下面的示例将一个脚本的 UI 渲染逻辑和数据处理逻辑分离到两个不同的模块中。

```javascript
// main.js

// 引入 UI 渲染模块和数据处理模块
const uiRenderer = require('./scripts/ui_renderer');
const dataProcessor = require('./scripts/data_processor');

async function main() {
  $ui.render({
    props: { title: "模块化示例" },
    views: [
      {
        type: "label",
        props: { id: "data-label", text: "正在加载数据...", lines: 0 },
        layout: $layout.fill
      }
    ]
  });

  // 调用数据处理模块获取数据
  const data = await dataProcessor.fetchData();

  // 调用 UI 渲染模块更新界面
  uiRenderer.updateUIWithData(data);
}

main();

// --- scripts/ui_renderer.js --- (这是一个单独的文件)
// module.exports = {
//   updateUIWithData: (data) => {
//     $("data-label").text = `数据加载完成:\n${JSON.stringify(data, null, 2)}`;
//   }
// };

// --- scripts/data_processor.js --- (这是一个单独的文件)
// module.exports = {
//   fetchData: async () => {
//     // 模拟网络请求
//     await $wait(2);
//     return { message: "Hello from data processor!", timestamp: Date.now() };
//   }
// };
```

### 总结

`require()` 和 `module.exports` 是 JSBox 中实现代码模块化的标准方式，推荐用于构建大型、结构化的脚本项目。它们能够有效隔离代码，避免命名冲突，并提升代码的可维护性和复用性。而 `$include()` 仅适用于简单的代码注入，应谨慎使用以避免全局变量污染。 
