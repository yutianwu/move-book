# 地址类型

<!--

Chapter: Basic Syntax
Goal: Introduce the address type
Notes:
    - a special type
    - named addresses via the Move.toml
    - address literals
    - 0x2 is 0x0000000...02

Links:
    - address concept
    - transaction context
    - Move.toml
    - your first move

 -->

Move 使用一种称为[地址](./../concepts/address)的特殊类型来表示地址。它是一个 32 字节的值，可以表示区块链上的任何地址。地址可以用两种形式编写：以 0x 为前缀的十六进制地址和命名地址。

```move file=packages/samples/sources/move-basics/address.move anchor=address_literal

```

地址字面量以 `@` 符号开头，后跟十六进制数字或标识符。十六进制数字被解释为 32 字节值。标识符会在 [Move.toml](./../concepts/manifest) 文件中查找，并由编译器替换为相应的地址。如果在 Move.toml 文件中找不到标识符，编译器将抛出错误。

## 转换

Sui Framework 提供了一组辅助函数来处理地址。由于地址类型是 32 字节值，它可以转换为 `u256` 类型，反之亦然。它也可以与 `vector<u8>` 类型相互转换。

示例：将地址转换为 `u256` 类型并转换回来。

```move file=packages/samples/sources/move-basics/address.move anchor=to_u256

```

示例：将地址转换为 `vector<u8>` 类型并转换回来。

```move file=packages/samples/sources/move-basics/address.move anchor=to_bytes

```

示例：将地址转换为字符串。

```move file=packages/samples/sources/move-basics/address.move anchor=to_string

```

## 进一步阅读

- Move 参考中的[地址](./../../reference/primitive-types/address)。
- [sui::address](https://docs.sui.io/references/framework/sui/address) 模块文档。