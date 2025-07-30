# 二进制规范序列化

二进制规范序列化（BCS）是一种用于结构化数据的二进制编码格式。它最初在Diem中设计，后来成为Move的标准序列化格式。BCS简单、高效、确定性，并且易于在任何编程语言中实现。

> 完整的格式规范可在[BCS存储库](https://github.com/zefchain/bcs)中获得。

## 格式

BCS是一种二进制格式，支持最大256位的无符号整数、选项、布尔值、单元（空值）、固定和可变长度序列以及映射。该格式被设计为确定性的，这意味着相同的数据将始终序列化为相同的字节。

> "BCS不是自描述格式。因此，为了反序列化消息，必须提前知道消息类型和布局"来自[README](https://github.com/zefchain/bcs)

整数以小端格式存储，可变长度整数使用可变长度编码方案编码。序列以其长度作为ULEB128前缀，枚举存储为变体的索引后跟数据，映射存储为键值对的有序序列。

结构体被视为字段序列，字段按照在结构体中定义的顺序序列化。字段使用与顶级数据相同的规则进行序列化。

## 使用BCS

[Sui框架](./sui-framework)包含sui::bcs模块用于编码和解码数据。编码函数是VM的原生函数，解码函数在Move中实现。

## 编码

要编码数据，请使用`bcs::to_bytes`函数，它将数据引用转换为字节向量。此函数支持编码任何类型，包括结构体。

```move
module std::bcs;

public native fun to_bytes<T>(t: &T): vector<u8>;
```

以下示例显示如何使用BCS编码结构体。`to_bytes`函数可以接受任何值并将其编码为字节向量。

```move file=packages/samples/sources/programmability/bcs.move anchor=encode

```

### 编码结构体

结构体的编码与简单类型类似。以下是如何使用BCS编码结构体：

```move file=packages/samples/sources/programmability/bcs.move anchor=encode_struct

```

## 解码

由于BCS不进行自描述且Move是静态类型的，解码需要对数据类型的先验知识。`sui::bcs`模块提供各种函数来协助此过程。

### 包装器API

BCS在Move中作为包装器实现。解码器按值接受字节，然后允许调用者通过调用不同的解码函数（以`peel_*`为前缀）来_剥离_数据。数据从字节中分离出来，剩余字节保留在包装器中，直到调用`into_remainder_bytes`函数。

```move file=packages/samples/sources/programmability/bcs.move anchor=decode

```

在解码过程中，通常在单个`let`语句中使用多个变量。这使代码更易读，并有助于避免不必要的数据复制。

```move file=packages/samples/sources/programmability/bcs.move anchor=chain_decode

```

### 解码向量

虽然大多数原始类型都有专用的解码函数，但向量需要特殊处理，这取决于元素的类型。对于向量，首先需要解码向量的长度，然后在循环中解码每个元素。

```move file=packages/samples/sources/programmability/bcs.move anchor=decode_vector

```

对于最常见的场景，`bcs`模块提供了一套基本的向量解码函数：

- `peel_vec_address(): vector<address>`
- `peel_vec_bool(): vector<bool>`
- `peel_vec_u8(): vector<u8>`
- `peel_vec_u64(): vector<u64>`
- `peel_vec_u128(): vector<u128>`
- `peel_vec_vec_u8(): vector<vector<u8>>` - 字节向量的向量

### 解码选项

<!--
> 巧合的是，Option在Move中是一个向量，与BCS中带有单个变体的枚举表示重叠，使得Rust中的Option与Move中的Option完全兼容。
-->

[选项](./../move-basics/option)表示为0或1个元素的向量。要读取选项，您需要将其视为向量并检查其长度（第一个字节 - 1或0）。

```move file=packages/samples/sources/programmability/bcs.move anchor=decode_option

```

> 如果您需要解码自定义类型的选项，请使用上面代码片段中的方法。

对于最常见的场景，`bcs`模块提供了一套基本的选项解码函数：

- `peel_option_address(): Option<address>`
- `peel_option_bool(): Option<bool>`
- `peel_option_u8(): Option<u8>`
- `peel_option_u64(): Option<u64>`
- `peel_option_u128(): Option<u128>`

### 解码结构体

结构体逐字段解码，没有标准函数可以自动将字节解码为Move结构体，这将违反Move的类型系统。相反，您需要手动解码每个字段。

```move file=packages/samples/sources/programmability/bcs.move anchor=decode_struct

```

## 总结

二进制规范序列化是结构化数据的高效二进制格式，确保跨平台的一致序列化。Sui框架提供了处理BCS的全面工具，通过内置函数允许广泛的功能。