> 提供了大量的用于文字处理的接口

# $text.uuid

生成一个 UUID 字符串：

```js
const uuid = $text.uuid;
```

# $text.tokenize(object)

对文本进行分词处理：

```js
$text.tokenize({
  text: "我能吞下玻璃而不伤身体",
  handler: function(results) {

  }
})
```

# $text.analysis(object)

// TODO: 对文本进行自然语言分析

# $text.lookup(string)

在系统内置词典中查找文本：

```js
$text.lookup("apple")
```

# $text.speech(object)

文本转成语音：

```js
$text.speech({
  text: "Hello, World!",
  rate: 0.5,
  language: "zh-CN", // optional
})
```

你可以将此过程暂停/继续或停止：

```js
const speaker = $text.speech({});
speaker.pause()
speaker.continue()
speaker.stop()
```

可以通过 `events` 来获取状态：

```js
$text.speech({
  text: "Hello, World!",
  events: {
    didStart: (sender) => {},
    didFinish: (sender) => {},
    didPause: (sender) => {},
    didContinue: (sender) => {},
    didCancel: (sender) => {},
  }
})
```

支持语言列表：

```
ar-SA
cs-CZ
da-DK
de-DE
el-GR
en-AU
en-GB
en-IE
en-US
en-US
en-ZA
es-ES
es-MX
fi-FI
fr-CA
fr-FR
he-IL
hi-IN
hu-HU
id-ID
it-IT
ja-JP
ko-KR
nl-BE
nl-NL
no-NO
pl-PL
pt-BR
pt-PT
ro-RO
ru-RU
sk-SK
sv-SE
th-TH
tr-TR
zh-CN
zh-HK
zh-TW
```

# $text.ttsVoices

获取当前设备支持的语音列表，可以选择一个用于指定 $text.speech 语音：

```js
const voices = $text.ttsVoices;
console.log(voices);

$text.speech({
  text: "Hello, World!",
  voice: voices[0]
});
```

Voice 对象结构

属性 | 类型 | 读写 | 说明
---|---|---|---
language | string | 只读 | 语言
identifier | string | 只读 | 标识符
name | string | 只读 | 名称
quality | number | 只读 | 语音质量
gender | number | 只读 | 声音性别
audioFileSettings | object | 只读 | 音频文件设置

# $text.base64Encode(string)

Base64 encode.

# $text.base64Decode(string)

Base64 decode.

# $text.URLEncode(string)

URL encode.

# $text.URLDecode(string)

URL decode.

# $text.HTMLEscape(string)

HTML escape.

# $text.HTMLUnescape(string)

HTML unescape.

# $text.MD5(string)

MD5.

# $text.SHA1(string)

SHA1.

# $text.SHA256(string)

SHA256.

# $text.convertToPinYin(text)

将文字转换成汉语拼音。

# $text.markdownToHtml(text)

将 Markdown 文本转换成 HTML 文本。

# $text.htmlToMarkdown(object)

将 HTML 文本转换成 Markdown 文本，这是一个异步接口：

```js
$text.htmlToMarkdown({
  html: "<p>Hey</p>",
  handler: markdown => {

  }
})

// Or
var markdown = await $text.htmlToMarkdown("<p>Hey</p>");
```

# $text.decodeData(object)

将 data 转换成字符串：

```js
const string = $text.decodeData({
  data: file,
  encoding: 4 // default, refer: https://developer.apple.com/documentation/foundation/nsstringencoding
});
```

# $text.sizeThatFits(object)

动态计算文字的高度：

```js
const size = $text.sizeThatFits({
  text: "Hello, World",
  width: 320,
  font: $font(20),
  lineSpacing: 15, // Optional
});
```

---

## 文件内容解读与示例

### 用途说明

`$text` API 是 JSBox 中一个功能极其丰富的**文本处理工具箱**。它提供了大量超越 JavaScript 原生字符串操作的能力，涵盖了从文本转语音、各种编码转换、格式转换到文本布局计算等多个方面。这些功能通常利用了 iOS 系统底层的能力，使得在 JSBox 中处理复杂文本任务变得非常便捷。

### 功能分类与示例

#### 1. 文本转语音 (Text-to-Speech - TTS)

-   **`$text.speech(options)`**: 将文本朗读出来。你可以控制语速 (`rate`)、语言 (`language`)，甚至指定具体的语音 (`voice`)。支持播放、暂停、继续和停止。
-   **`$text.ttsVoices`**: 获取当前设备支持的所有语音列表，方便你选择。

**示例**：朗读一段中文文本，并控制播放。

```javascript
const textToSpeak = "你好，JSBox！这是一个文本转语音的示例。";
let speakerInstance;

$ui.render({
  props: { title: "文本转语音" },
  views: [
    {
      type: "label",
      props: { text: textToSpeak, lines: 0 },
      layout: make => make.top.left.right.inset(20)
    },
    {
      type: "stack",
      props: {
        axis: $stackViewAxis.horizontal,
        distribution: $stackViewDistribution.fillEqually,
        spacing: 10
      },
      layout: make => {
        make.top.equalTo(view.prev.bottom).offset(20);
        make.left.right.inset(20);
        make.height.equalTo(40);
      },
      views: [
        { type: "button", title: "播放", events: { tapped: () => {
          speakerInstance = $text.speech({
            text: textToSpeak,
            language: "zh-CN",
            rate: 0.5, // 语速，0.0 - 1.0
            events: { didFinish: () => $ui.toast("播放完成") }
          });
        } } },
        { type: "button", title: "暂停", events: { tapped: () => speakerInstance && speakerInstance.pause() } },
        { type: "button", title: "继续", events: { tapped: () => speakerInstance && speakerInstance.continue() } },
        { type: "button", title: "停止", events: { tapped: () => speakerInstance && speakerInstance.stop() } }
      ]
    }
  ]
});
```

#### 2. 编码与解码

-   **`$text.base64Encode(string)` / `$text.base64Decode(string)`**: 进行 Base64 编码和解码。
-   **`$text.URLEncode(string)` / `$text.URLDecode(string)`**: 进行 URL 编码和解码，常用于处理 URL 中的特殊字符。
-   **`$text.HTMLEscape(string)` / `$text.HTMLUnescape(string)`**: 进行 HTML 实体转义和反转义，防止 XSS 攻击或正确显示 HTML 标签。

**示例**：URL 编码与解码

```javascript
const original = "https://example.com/搜索?q=你好 世界";
const encoded = $text.URLEncode(original);
console.log("编码后:", encoded); // https://example.com/%E6%90%9C%E7%B4%A2?q=%E4%BD%A0%E5%A5%BD%20%E4%B8%96%E7%95%8C
console.log("解码后:", $text.URLDecode(encoded)); // https://example.com/搜索?q=你好 世界
```

#### 3. 格式转换

-   **`$text.markdownToHtml(text)`**: 将 Markdown 文本转换为 HTML 文本。
-   **`$text.htmlToMarkdown(options)`**: 将 HTML 文本转换为 Markdown 文本（异步操作）。
-   **`$text.convertToPinYin(text)`**: 将中文字符串转换为拼音。

**示例**：Markdown 转 HTML

```javascript
const mdContent = "# 标题\n\n**粗体**和*斜体*。\n\n- 列表项1\n- 列表项2";
const htmlContent = $text.markdownToHtml(mdContent);
console.log("转换后的 HTML:\n", htmlContent);
```

#### 4. 唯一标识符与哈希

-   **`$text.uuid`**: 生成一个全局唯一的 UUID 字符串。
-   **`$text.MD5(string)` / `$text.SHA1(string)` / `$text.SHA256(string)`**: 计算字符串的 MD5、SHA1、SHA256 哈希值，常用于数据校验或加密。

**示例**：生成 UUID 和计算 MD5

```javascript
console.log("生成的 UUID:", $text.uuid);
console.log("'hello' 的 MD5:", $text.MD5("hello"));
```

#### 5. 文本布局计算

-   **`$text.sizeThatFits(options)`**: 根据给定的文本内容、宽度限制和字体，计算出文本所需的实际高度。这对于动态调整 UI 布局以适应不同长度的文本非常有用。

**示例**：计算文本高度

```javascript
const longText = "这是一段需要计算其高度的文本，它可能会根据给定的宽度进行换行。";
const calculatedSize = $text.sizeThatFits({
  text: longText,
  width: 200, // 限制宽度为 200
  font: $font(16),
  lineSpacing: 5 // 可选：行间距
});
console.log(`文本在宽度200下所需高度: ${calculatedSize.height}`);
```

`$text` 模块是 JSBox 中处理文本的瑞士军刀，它提供了许多高级功能，能够帮助你轻松实现复杂的文本处理任务。 
