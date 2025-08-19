# $objc_retain(object)

在有些时候通过 Runtime 声明的对象会被系统释放掉，如果你想要长期持有一个对象，可以使用这个方法：

```js
const manager = $objc("Manager").invoke("new");
$objc_retain(manager)
```

这将在整个脚本运行期间保持 manger 不被释放。

# $objc_relase(object)

与 retain 相对应的函数，目的是手动释放掉对象：

```js
$objc_release(manager)
```

---

## 文件内容解读与示例

### 用途说明

本文档介绍了 JSBox Runtime 环境中的**内存管理**概念，特别是如何使用 `$objc_retain()` 和 `$objc_release()` 来手动控制 Objective-C 对象的生命周期。这是一个**非常高级且危险**的话题，通常只在处理特定原生对象生命周期问题时才需要。

### 核心概念：ARC 与手动内存管理

-   **ARC (Automatic Reference Counting)**: iOS 和 Objective-C 大部分情况下使用 ARC 自动管理内存。当一个 Objective-C 对象的引用计数（reference count）归零时，ARC 会自动释放它所占用的内存。
-   **JSBox Runtime 的挑战**: 在 JSBox 的 JavaScript 环境中，当一个 Objective-C 对象被桥接过来时，JSBox 会自动管理其引用计数。然而，在某些复杂场景下（例如，原生回调、异步操作、或将原生对象存储在 JavaScript 变量中但原生层已不再持有它），JSBox 的自动管理机制可能无法正确判断一个原生对象是否仍然被需要，导致其被过早释放（deallocated），从而引发崩溃。
-   **`$objc_retain()` 和 `$objc_release()` 的作用**: 这两个函数允许开发者**手动干预** Objective-C 对象的引用计数。它们是 Objective-C 中手动内存管理（MRC）时代的遗留，但在 ARC 环境下，它们仍然可以用于解决特定的引用计数问题。
    -   **`$objc_retain(object)`**: 增加 Objective-C 对象的引用计数。这告诉系统“我（JavaScript 环境）还需要这个对象，请不要释放它”。
    -   **`$objc_release(object)`**: 减少 Objective-C 对象的引用计数。这告诉系统“我不再需要这个对象了”。

### 警告

**不当使用 `$objc_retain()` 和 `$objc_release()` 会导致严重的内存问题**：

-   **内存泄漏 (Memory Leak)**: 如果你 `retain` 了一个对象，但忘记了在不再需要时 `release` 它，那么这个对象将永远不会被释放，导致内存占用持续增加。
-   **崩溃 (Crash)**: 如果你 `release` 了一个已经被释放的对象，或者 `release` 了不属于你持有的对象，会导致“野指针”（dangling pointer）问题，从而引发应用崩溃。

**对于大多数开发者而言，应尽量避免使用这两个函数。** 它们仅为解决特定、复杂且难以通过其他方式解决的 Runtime 内存问题而存在。

### 示例代码：概念性演示

下面的示例仅为演示 `$objc_retain` 和 `$objc_release` 的概念，**不建议在实际生产环境中使用，除非你完全理解其内存管理机制。**

```javascript
// 假设我们通过 Runtime 创建了一个原生对象
// 例如，一个自定义的 Manager 类实例，它可能在某个原生回调中被创建
const manager = $objc("NSObject").invoke("new"); // 简化为一个 NSObject 实例

console.log("对象创建后，引用计数理论上为 1。");

// 1. 使用 $objc_retain 增加引用计数
// 这告诉系统，JavaScript 环境现在也持有了这个对象的一个强引用
$objc_retain(manager);
console.log("调用 $objc_retain 后，引用计数理论上为 2。");

// 模拟一个异步操作，在这个操作中我们可能需要用到 manager 对象
$delay(3, () => {
  console.log("异步操作完成，尝试使用 manager 对象...");
  try {
    // 假设 manager 有一个方法叫 description
    const desc = manager.invoke("description").jsValue();
    console.log("Manager 描述:", desc);
  } catch (e) {
    console.error("异步回调中访问 manager 失败，可能已被释放:", e);
  }

  // 2. 使用 $objc_release 减少引用计数
  // 这告诉系统，JavaScript 环境不再需要这个对象了
  $objc_release(manager);
  console.log("调用 $objc_release 后，引用计数理论上为 1。");

  // 此时，如果原生层也没有其他引用，对象就会被释放
});

console.log("脚本主线程继续执行，等待异步回调...");
```

**代码解读**：

这个示例模拟了一个场景：一个原生对象 `manager` 被创建，然后通过 `$objc_retain` 增加其引用计数，以确保它在异步操作期间不会被过早释放。在异步操作完成后，通过 `$objc_release` 减少引用计数，以平衡之前的 `retain`，避免内存泄漏。

### 总结

`$objc_retain()` 和 `$objc_release()` 是 JSBox Runtime 中用于手动内存管理的“高级工具”。它们是为解决特定、复杂场景下的原生对象生命周期问题而存在的，但使用它们需要对 Objective-C 的内存管理有深入的理解，否则极易引入新的问题。对于大多数开发者而言，应尽量避免使用，而依赖 JSBox 自动的内存管理机制。 
