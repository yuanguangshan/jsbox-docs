> JSBox 自带的文本检测模块，用于方便的提取文本中的特定内容

# $detector

严格的说来，这部分能提供的内容，用原生 JavaScript 的正则表达式同样可以实现，这里只是一个相对简洁的实现方式。

# $detector.date(string)

将文本中所有的日期提取出来：

```js
const dates = $detector.date("2017年10月10日");
```

# $detector.address(string)

将文本中所有的地址提取出来：

```js
const addresses = $detector.address("");
```

# $detector.link(string)

将文本中所有的链接提取出来：

```js
const links = $detector.link("http://apple.com hello http://xteko.com");
```

# $detector.phoneNumber(string)

将文本中所有的电话号码提取出来：

```js
const phoneNumbers = $detector.phoneNumber("18666666666 hello 18777777777");
```

---

## 文件内容解读与示例

### 用途说明

`$detector` API 是 JSBox 提供的一个用于**智能识别和提取**文本中特定类型数据的模块。它能够从非结构化的文本中，自动识别出日期、地址、链接和电话号码等信息。虽然这些功能理论上可以通过复杂的正则表达式实现，但 `$detector` 提供了一种更简洁、更可靠、更符合语义的方式来完成这些任务，因为它通常利用了 iOS 系统内置的数据检测能力。

### 核心概念：语义化识别

与简单的字符串匹配不同，`$detector` 能够理解文本的**语义**。例如，它不仅能识别“2024-08-19”，也能识别“明天下午三点”这样的日期表达。这使得它在处理用户输入的自然语言文本时，比纯粹的正则表达式更加强大和灵活。

### 方法详解与示例

`$detector` 的所有方法都接收一个字符串作为输入，并返回一个包含所有识别结果的**数组**。如果未识别到任何内容，则返回空数组。

#### 1. `$detector.date(string)`: 识别日期

- **用途**: 从文本中提取所有可能的日期和时间信息。

**示例**：

```javascript
const text1 = "会议定于2024年8月19日下午3点举行。";
const dates1 = $detector.date(text1);
console.log("日期1:", dates1); // 输出: ["2024年8月19日下午3点"]

const text2 = "请在下周二上午提交报告。";
const dates2 = $detector.date(text2);
console.log("日期2:", dates2); // 输出: ["下周二上午"]
```

#### 2. `$detector.address(string)`: 识别地址

- **用途**: 从文本中提取所有可能的地理地址信息。

**示例**：

```javascript
const text = "我的地址是北京市朝阳区建国路88号，邮编100022。";
const addresses = $detector.address(text);
console.log("地址:", addresses); // 输出: ["北京市朝阳区建国路88号"]
```

#### 3. `$detector.link(string)`: 识别链接

- **用途**: 从文本中提取所有符合 URL 格式的链接。

**示例**：

```javascript
const text = "访问 JSBox 官网：https://www.jsbox.com 或在百度搜索：www.baidu.com。";
const links = $detector.link(text);
console.log("链接:", links); // 输出: ["https://www.jsbox.com", "www.baidu.com"]
```

#### 4. `$detector.phoneNumber(string)`: 识别电话号码

- **用途**: 从文本中提取所有符合电话号码格式的字符串。

**示例**：

```javascript
const text = "我的手机号是13800138000，公司座机010-12345678，紧急联系人13912345678。";
const phoneNumbers = $detector.phoneNumber(text);
console.log("电话号码:", phoneNumbers); // 输出: ["13800138000", "010-12345678", "13912345678"]
```

### 示例代码：文本信息提取器

下面的示例将创建一个简单的 UI，让用户输入一段文本，然后点击按钮，脚本将自动识别并列出文本中的日期、链接和电话号码。

```javascript
$ui.render({
  props: {
    title: "文本信息提取器"
  },
  views: [
    {
      type: "text",
      props: {
        id: "input-text",
        placeholder: "请输入一段包含日期、链接、电话的文本...",
        lines: 0 // 允许多行输入
      },
      layout: (make, view) => {
        make.top.left.right.inset(10);
        make.height.equalTo(150);
      }
    },
    {
      type: "button",
      props: { title: "开始识别" },
      layout: (make, view) => {
        make.top.equalTo($("input-text").bottom).offset(10);
        make.centerX.equalTo(view.super);
        make.width.equalTo(120);
      },
      events: {
        tapped: (sender) => {
          const inputText = $("input-text").text;
          if (!inputText) {
            $ui.alert("请输入文本！");
            return;
          }

          const dates = $detector.date(inputText);
          const links = $detector.link(inputText);
          const phoneNumbers = $detector.phoneNumber(inputText);

          let resultText = "识别结果:\n\n";
          resultText += `日期: ${dates.length > 0 ? dates.join(", ") : "无"}\n`;
          resultText += `链接: ${links.length > 0 ? links.join(", ") : "无"}\n`;
          resultText += `电话: ${phoneNumbers.length > 0 ? phoneNumbers.join(", ") : "无"}\n`;

          $("result-label").text = resultText;
        }
      }
    },
    {
      type: "label",
      props: {
        id: "result-label",
        lines: 0, // 允许多行显示结果
        font: $font(16),
        textColor: $color("darkGray")
      },
      layout: (make, view) => {
        make.top.equalTo(view.prev.bottom).offset(20);
        make.left.right.bottom.inset(10);
      }
    }
  ]
});

**代码解读**：

1.  用户在 `input-text` 文本框中输入内容。
2.  点击“开始识别”按钮后，脚本获取输入文本。
3.  分别调用 `$detector.date()`, `$detector.link()`, `$detector.phoneNumber()` 来识别不同类型的信息。
4.  将识别结果格式化后，显示在 `result-label` 标签中。

`$detector` 模块是处理非结构化文本、从中提取关键信息的强大工具，它能让你轻松实现许多智能化的文本处理功能。 
