# 字符串

虽然 Move 没有表示字符串的内置类型，但它在[标准库](./standard-library)中有两个字符串的标准实现。`std::string` 模块定义了一个 `String` 类型和用于 UTF-8 编码字符串的方法，第二个模块 `std::ascii` 提供了一个 ASCII `String` 类型及其方法。

> Sui 执行环境会在交易输入中自动将字节向量转换为 `String`。因此，在许多情况下，
> 在[交易块](./../concepts/what-is-a-transaction)中直接构造字符串是不必要的。

<!--

## Bytestring Literal

TODO:

- reference vector
- reference literals - [Expression](./expression#literals)

-->

## 字符串就是字节

无论您使用哪种类型的字符串，重要的是要知道字符串只是字节。`string` 和 `ascii` 模块提供的包装器就是这样：包装器。它们确实提供了安全检查和处理字符串的方法，但归根结底，它们只是字节向量。

```move file=packages/samples/sources/move-basics/string.move anchor=custom

```

## 使用 UTF-8 字符串

虽然标准库中有两种类型的字符串（`string` 和 `ascii`），但应该将 `string` 模块视为默认选择。它有许多常见操作的原生实现，利用低级优化的运行时代码来获得卓越的性能。相比之下，`ascii` 模块完全用 Move 实现，依赖于更高级的抽象，使其不太适合对性能要求严格的任务。

### 定义

`std::string` 模块中的 `String` 类型定义如下：

```move
module std::string;

/// A `String` holds a sequence of bytes which is guaranteed to be in utf8 format.
public struct String has copy, drop, store {
    bytes: vector<u8>,
}
```

_请参阅 [std::string][string-stdlib] 模块的完整文档。_

### 创建字符串

要创建新的 UTF-8 `String` 实例，您可以使用 `string::utf8` 方法。[标准库](./standard-library)在 `vector<u8>` 上提供了一个别名 `.to_string()` 以便使用。

```move file=packages/samples/sources/move-basics/string.move anchor=utf8

```

### 常见操作

UTF8 字符串提供了许多处理字符串的方法。字符串上最常见的操作是：连接、切片和获取长度。此外，对于自定义字符串操作，可以使用 `bytes()` 方法来获取底层的字节向量。

```move
let mut str = b"Hello,".to_string();
let another = b" World!".to_string();

// append(String) 将内容添加到字符串的末尾
str.append(another);

// `sub_string(start, end)` 复制字符串的一个切片
str.sub_string(0, 5); // "Hello"

// `length()` 返回字符串中的字节数
str.length(); // 12 (bytes)

// 方法也可以链式调用！获取子字符串的长度
str.sub_string(0, 5).length(); // 5 (bytes)

// 字符串是否为空
str.is_empty(); // false

// 获取底层字节向量进行自定义操作
let bytes: &vector<u8> = str.bytes();
```

### 安全的 UTF-8 操作

如果传入的字节不是有效的 UTF-8，默认的 `utf8` 方法可能会中止。如果您不确定传递的字节是否有效，应该使用 `try_utf8` 方法。它返回一个 `Option<String>`，如果字节不是有效的 UTF-8，则不包含值，否则包含字符串。

> 提示：以 `try_*` 开头的函数名称通常返回 `Option`。如果操作成功，结果会被包装在 `Some` 中。如果失败，函数返回 `None`。这种命名约定在 Move 中很常见，受到 Rust 的启发。

```move file=packages/samples/sources/move-basics/string.move anchor=safe_utf8

```

### UTF-8 限制

`string` 模块不提供访问字符串中单个字符的方式。这是因为 UTF-8 是可变长度编码，字符的长度可以是 1 到 4 个字节之间的任何值。同样，`length()` 方法返回字符串中的字节数，而不是字符数。

然而，像 `sub_string` 和 `insert` 这样的方法会验证字符边界，如果指定的索引位于字符中间，会中止执行。

## ASCII 字符串

这个部分即将推出！

## 进一步阅读

- [std::string][string-stdlib] 模块文档。
- [std::ascii][ascii-stdlib] 模块文档。

[enum-reference]: /reference/enums.html
[string-stdlib]: https://docs.sui.io/references/framework/std/string
[ascii-stdlib]: https://docs.sui.io/references/framework/std/ascii