# $xml

JSBox 内置了一个简单易用的 XML 解析器，你可以用它来解析 XML 和 HTML 文档，并支持 `xPath` 和 `CSS selector` 两种查询方式。

# $xml.parse(object)

使用 `parse` 函数完成对字符串或文件的解析：

```js

let xml = 
`
<note>
  <to>Tove</to>
  <from>Jani</from>
  <heading>Reminder</heading>
  <body>Don't forget me this weekend!</body>
</note>
`;

let doc = $xml.parse({
  string: xml, // Or data: data
  mode: "xml", // Or "html", default to xml
});
```

# Document

$xml.parse() 返回一个 XML Document 对象：

```js
let version = doc.version;
let rootElement = doc.rootElement;
```

# Element

Element 对象表示 XML Document 上面的某一个节点，具有如下属性：

属性 | 类型 | 读写 | 说明
---|---|---|---
node | string | 只读 | 包括 tag 在内的字符串表示
document | $xmlDoc | 只读 | 此节点的 Document
blank | bool | 只读 | 命名空间
tag | string | 只读 | tag
lineNumber | number | 只读 | 行数
attributes | object | 只读 | attributes
parent | $xmlElement | 只读 | 父节点
previous | $xmlElement | 只读 | 前一个兄弟节点
next | $xmlElement | 只读 | 后一个兄弟节点
string | string | 只读 | 节点的字符串表示
number | number | 只读 | 节点的数字表示
date | Date | 只读 | 节点的 Date 类型表示

# Element.firstChild(object)

Document 对象和 Element 对象都通过 xPath, selector 和 tag 获取第一个节点：

```js
let c1 = doc.rootElement.firstChild({
  "xPath": "//food/name"
});

let c2 = doc.rootElement.firstChild({
  "selector": "food > serving[units]"
});

let c3 = doc.rootElement.firstChild({
  "tag": "daily-values",
  "namespace": "namespace", // Optional
});
```

# Element.children(object)

Document 对象和 Element 对象都通过 tag 和 namespace 获取子节点：

```js
let children = doc.rootElement.children({
  "tag": "daily-values",
  "namespace": "namespace", // Optional
});

// Get all
let allChildren = doc.rootElement.children();
```

# Element.enumerate(object)

Document 对象和 Element 对象都通过 xPath 和 CSS selector 枚举：

```js
let element = doc.rootElement;
element.enumerate({
  xPath: "//food/name", // Or selector (e.g. food > serving[units])
  handler: (element, idx) => {

  }
});
```

# Element.value(object)

Document 对象和 Element 对象都通过 attribute 和 namespace 获取值：

```js
let value = doc.rootElement.value({
  "attribute": "attribute",
  "namespace": "namespace", // Optional
});
```

# Document.definePrefix(object)

Document 对象可以通过 definePrefix 定义命名空间前缀：

```js
doc.definePrefix({
  "prefix": "prefix",
  "namespace": "namespace"
});
```

请参考这个样例来了解更多：https://github.com/cyanzhong/xTeko/tree/master/extension-demos/xml-demo

---

## 文件内容解读与示例

### 用途说明

`$xml` API 提供了一个强大且易用的 **XML/HTML 解析器**。它允许你的 JSBox 脚本将 XML 或 HTML 格式的字符串（或文件数据）解析成一个结构化的**文档对象模型（DOM）**。一旦解析完成，你就可以像操作网页 DOM 一样，通过各种查询方法（如 XPath 或 CSS Selector）来遍历、查找和提取其中的数据。这对于网页数据抓取（Web Scraping）、处理 RSS/Atom 订阅源、解析 XML 配置文件等场景非常有用。

### 核心概念：解析与查询

1.  **解析 (`$xml.parse(options)`)**: 
    -   **输入**: 你需要提供要解析的 XML 或 HTML 内容，可以是字符串 (`string`) 或 `$data` 对象 (`data`)。
    -   **模式 (`mode`)**: 必须指定解析模式为 `"xml"` 或 `"html"`。`"html"` 模式对不规范的 HTML 容错性更好。
    -   **返回值**: 一个 `Document` 对象，它是整个解析后文档的根节点。

2.  **查询 (XPath, CSS Selector, Tag)**: 
    -   一旦你有了 `Document` 对象或任何 `Element` 对象，就可以使用强大的查询语言来定位你需要的节点。
    -   **XPath**: 一种用于在 XML 文档中导航的语言，非常灵活和强大，适合复杂查询。
    -   **CSS Selector**: 熟悉 Web 开发的开发者会非常熟悉，使用 CSS 样式选择器语法来查找元素（例如 `div.container > p#intro`）。
    -   **Tag**: 最简单的查询方式，直接通过元素的标签名（如 `"div"`, `"a"`）来查找。

### 核心对象与方法

-   **`Document` 对象**: 由 `$xml.parse()` 返回，代表整个文档。它有 `rootElement` 属性，指向文档的根元素。
-   **`Element` 对象**: 代表 XML/HTML 树中的一个节点。它有以下常用属性：
    -   `tag`: 元素的标签名（如 `"div"`, `"p"`）。
    -   `attributes`: 一个对象，包含元素的所有属性（如 `id`, `class`, `href`）。
    -   `string`: 元素的文本内容（不包含子元素的标签）。
    -   `value({ attribute: "attrName" })`: 获取指定属性的值。
-   **查询方法**: 
    -   `element.firstChild(query)`: 查找并返回第一个匹配的子元素。
    -   `element.children(query)`: 查找并返回所有匹配的子元素数组。
    -   `element.enumerate(query, handler)`: 遍历所有匹配的元素，并对每个元素执行回调函数。这对于处理多个结果非常方便。

### 示例代码：解析 HTML 页面并提取数据

下面的示例将解析一个简单的 HTML 字符串，并提取页面的标题和所有链接的文本及 URL。

```javascript
const sampleHtml = `
<html>
<head>
  <title>JSBox 示例页面</title>
</head>
<body>
  <h1>欢迎来到 JSBox 世界</h1>
  <p>这是一个关于 <a href="https://www.jsbox.com">JSBox 官网</a> 的链接。</p>
  <p>你可以在这里找到 <a href="https://docs.jsbox.com">官方文档</a>。</p>
  <div class="footer">
    <a href="https://github.com/cyanzhong">作者 GitHub</a>
  </div>
</body>
</html>
`;

// 1. 解析 HTML 字符串
const doc = $xml.parse({
  string: sampleHtml,
  mode: "html" // 指定为 HTML 模式
});

if (doc) {
  // 2. 获取页面标题
  const titleElement = doc.rootElement.firstChild({ tag: "title" });
  const pageTitle = titleElement ? titleElement.string : "无标题";
  console.log("页面标题:", pageTitle);

  // 3. 提取所有链接
  const links = [];
  doc.rootElement.enumerate({
    selector: "a", // 使用 CSS Selector 查找所有 <a> 标签
    handler: (element, idx) => {
      links.push({
        text: element.string, // 链接的文本内容
        href: element.value({ attribute: "href" }) // 链接的 href 属性值
      });
    }
  });

  console.log("提取到的链接:");
  links.forEach(link => {
    console.log(`- 文本: ${link.text}, URL: ${link.href}`);
  });

  $ui.alert({
    title: "解析完成",
    message: `标题: ${pageTitle}\n找到 ${links.length} 个链接。`
  });
} else {
  $ui.alert("HTML 解析失败！");
}
```

**代码解读**：

1.  我们使用 `$xml.parse()` 将 `sampleHtml` 字符串解析为 `doc` 对象，并指定 `mode: "html"` 以正确处理 HTML 结构。
2.  通过 `doc.rootElement.firstChild({ tag: "title" })`，我们使用 `tag` 查询方式获取 `<title>` 标签的第一个子元素，然后通过 `.string` 属性获取其文本内容。
3.  为了获取所有链接，我们使用了 `doc.rootElement.enumerate({ selector: "a" })`。`enumerate` 方法会遍历所有匹配 CSS Selector `"a"` 的元素，并在 `handler` 回调中为每个元素执行逻辑。在回调中，我们通过 `element.string` 获取链接文本，通过 `element.value({ attribute: "href" })` 获取 `href` 属性的值。

`$xml` 模块是处理结构化文本数据的利器。熟练掌握 XPath 和 CSS Selector 的查询语法，将极大地扩展你的脚本处理网页内容的能力。 
