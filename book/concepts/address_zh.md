# 地址

<!--

Chapter: Concepts
Goal: explain locations and addresses
Notes:
    - don't talk about the type
    - packages, accounts and objects are identified by addresses
    - addresses are 32 bytes long
    - addresses are unique
    - represented as hex strings (64 characters) prefixed with 0x
    - addresses are case insensitive

Links:
    - address type


- mention what an address is, because it identifies a package
    - address is used for packages, objects, and accounts
    - address is a 32-byte value
    - address is written in hexadecimal notation
    - don't describe the type yet
    - focus on the concept of address on blockchain and on Sui in particular

 -->

地址是区块链上某个位置的唯一标识符。它用于标识[包](./packages)、[账户](./what-is-an-account)和[对象](./../object/object-model)。
地址具有32字节的固定大小，通常表示为以`0x`为前缀的十六进制字符串。地址不区分大小写。

```move
0xe51ff5cd221a81c3d6e22b9e670ddf99004d71de4f769b0312b68c7c4872e2f1
```

上面的地址是一个有效地址的示例。它长度为64个字符（32字节），并以`0x`为前缀。

Sui还有保留地址，用于标识标准包和对象。保留地址通常是易于记忆和输入的简单值。例如，标准库的地址是`0x1`。短于32字节的地址，会在左侧用零填充。

```move
0x1 = 0x0000000000000000000000000000000000000000000000000000000000000001
```

以下是一些保留地址的示例：

- `0x1` - Sui标准库的地址（别名`std`）
- `0x2` - Sui框架的地址（别名`sui`）
- `0x6` - 系统`Clock`对象的地址

> 您可以在[附录B](../appendix/reserved-addresses)中找到所有保留地址。

## 延伸阅读

- Move中的[地址类型](../move-basics/address)
- [sui::address模块](https://docs.sui.io/references/framework/sui/address)