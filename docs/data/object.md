# 对象属性

在 JavaScript 和 Native 通信的过程中，`对象传递`是一个比较麻烦的话题，以下数据类型可以自动完成 JavaScript 和 Native 的转换：

  Objective-C type  |   JavaScript type
--------------------|---------------------
        nil         |     undefined
      NSNull        |        null
      NSString      |       string
      NSNumber      |   number, boolean
    NSDictionary    |   Object object
      NSArray       |    Array object
      NSDate        |     Date object
      NSBlock (1)   |   Function object (1)
        id (2)      |   Wrapper object (2)
      Class (3)     | Constructor object (3)

但是显然，在我们通过 JavaScript 调用原生接口时，会通过接口返回各种各样的类型，例如返回一个图片，当我们调用接口的时候，也可能传递一个图片/二进制文件等等数据。

当 Native 给 JavaScript 返回数据时，将会把数据封装成一个 Object 类型，JavaScript 可以通过属性名获取到内部的 property。

例如，当我们实现列表点击事件时，会实现这样的代码：

```js
didSelect: function(tableView, indexPath) {
  var row = indexPath.row
}
```

其中的 `indexPath` 本质上是 iOS 内部的 IndexPath 类型，在这个例子里面我们可以通过 `.section` 和 `.row` 来访问到它内部的属性。

对于具体有哪些属性，我们需要一个专门的表来列举全部内容，方便随时查阅：[对象属性表](object/data.md)。

# $props

`$props` 方法可以获取一个对象所有的属性名：

```js
const props = $props("string");
```

$props 是一个简单的 js 方法，实现如下：

```js
const $props = object => {
  const result = [];
  for (; object != null; object = Object.getPrototypeOf(object)) {
    const names = Object.getOwnPropertyNames(object);
    for (let idx=0; idx<names.length; idx++) {
      const name = names[idx];
      if (!result.includes(name)) {
        result.push(name)
      }
    }
  }
  return result
};
```

# $desc

`$desc` 方法可以获取一个对象的结构：

```js
let desc = $desc(object);
console.log(desc);
```

---

## 文件内容解读与示例

### 用途说明：原生对象在 JSBox 中的表现

本文档解释了 JSBox 如何将复杂的**原生 iOS 对象**（如图片、颜色、日期、文件数据等）无缝地桥接到 JavaScript 环境中。当你通过 JSBox 的 API 调用原生功能并获得返回结果时，这些结果通常是原生对象。JSBox 会将它们封装成 JavaScript 对象，让你能够像操作普通 JS 对象一样访问它们的属性和方法。

### 对象桥接机制

JSBox 内部有一套机制，能够自动将常见的 Objective-C 类型转换为对应的 JavaScript 类型：

- **基本类型**: `NSString` -> `string`, `NSNumber` -> `number`/`boolean`, `NSArray` -> `Array`, `NSDictionary` -> `Object` 等，这些转换是自动且透明的。
- **复杂类型**: 对于 `UIImage`, `NSData`, `NSIndexPath` 等更复杂的原生对象，JSBox 会将它们包装成 JavaScript 的 `Object` 类型。这意味着你可以通过点语法（`.`）来访问这些对象的属性，例如 `indexPath.row` 或 `image.width`。

### 内省工具：`$props` 和 `$desc`

JSBox 提供了两个非常有用的全局方法，帮助你探索这些被桥接过来的原生对象的内部结构：

#### 1. `$props(object)`: 获取对象的所有属性名

- **用途**: 当你拿到一个对象，但不确定它有哪些属性时，`$props` 可以列出该对象及其原型链上所有可访问的属性名。这对于调试和探索未知 API 返回的对象非常有用。
- **示例**: 
  ```javascript
  // 假设你在 list 的 didSelect 事件中收到了 indexPath
  // didSelect: function(sender, indexPath, data) {
  //   console.log("indexPath 的所有属性:", $props(indexPath));
  //   // 可能会输出类似：["section", "row", "length", "description", ...]
  // }

  // 也可以用于普通 JS 对象
  const myObject = { name: "JSBox", version: 2.0 };
  console.log("myObject 的所有属性:", $props(myObject)); // 输出: ["name", "version"]
  ```

#### 2. `$desc(object)`: 获取对象的结构描述

- **用途**: `$desc` 方法会返回一个字符串，详细描述了对象的内部结构和当前值。这对于深入理解原生对象在 JSBox 中的表现形式非常有帮助。
- **示例**: 
  ```javascript
  const myColor = $color("red");
  console.log("myColor 的描述:", $desc(myColor));
  // 输出可能类似：<UIColor: 0x600000000000; red: 1.000000; green: 0.000000; blue: 0.000000; alpha: 1.000000>

  const mySize = $size(100, 50);
  console.log("mySize 的描述:", $desc(mySize));
  // 输出可能类似：{width: 100, height: 50}
  ```

### 总结

`object.md` 解释了 JSBox 如何让原生 iOS 对象在 JavaScript 中变得“可操作”。`$props` 和 `$desc` 是你探索和调试这些对象的强大工具。当你需要了解某个特定原生对象（如 `$image` 对象、`$data` 对象）具体有哪些属性和方法时，你需要查阅文档中提到的**[对象属性表](object/data.md)**（这是一个外部链接，包含了所有具体对象的详细属性列表）。 
