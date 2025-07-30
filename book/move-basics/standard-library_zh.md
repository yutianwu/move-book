# 标准库

<!-- The Move standard library provides a set of modules  -->

Move 标准库为原生类型和操作提供功能。它是一个不与存储交互，但为处理和操作数据提供基本工具的标准模块集合。它是 [Sui 框架](./../programmability/sui-framework) 的唯一依赖项，并与其一起导入。

## 最常见的模块

在本书中，我们详细介绍了标准库中大部分模块，然而，提供功能概览也很有帮助，这样您就可以了解有哪些可用功能以及哪个模块实现了它们。

<!-- Custom CSS addition in the theme/custom.css  -->
<div class="modules-table">

| 模块                                                                               | 描述                                                                       | 章节                                 |
| ---------------------------------------------------------------------------------- | -------------------------------------------------------------------------- | ------------------------------------ |
| [std::string](https://docs.sui.io/references/framework/std/string)               | 提供基本的字符串操作                                                       | [字符串](./string)                   |
| [std::ascii](https://docs.sui.io/references/framework/std/ascii)                 | 提供基本的 ASCII 操作                                                      | -                                    |
| [std::option](https://docs.sui.io/references/framework/std/option)               | 实现 `Option<T>`                                                          | [选项](./option)                     |
| [std::vector](https://docs.sui.io/references/framework/std/vector)               | 向量类型的原生操作                                                         | [向量](./vector)                     |
| [std::bcs](https://docs.sui.io/references/framework/std/bcs)                     | 包含 `bcs::to_bytes()` 函数                                               | [BCS](./../programmability/bcs)      |
| [std::address](https://docs.sui.io/references/framework/std/address)             | 包含单个 `address::length` 函数                                           | [地址](./address)                    |
| [std::type_name](https://docs.sui.io/references/framework/std/type_name)         | 允许运行时_类型反射_                                                       | [类型反射](./type-reflection)        |
| [std::hash](https://docs.sui.io/references/framework/std/hash)                   | 哈希函数：`sha2_256` 和 `sha3_256`                                        | -                                    |
| [std::debug](https://docs.sui.io/references/framework/std/debug)                 | 包含调试函数，仅在**测试**模式下可用                                       | -                                    |
| [std::bit_vector](https://docs.sui.io/references/framework/std/bit_vector)       | 提供位向量操作                                                             | -                                    |
| [std::fixed_point32](https://docs.sui.io/references/framework/std/fixed_point32) | 提供 `FixedPoint32` 类型                                                  | -                                    |

</div>

## 整数模块

Move 标准库提供一组与整数类型关联的函数。这些函数被分成多个模块，每个模块都与特定的整数类型关联。这些模块不应该被直接导入，因为它们的函数在每个整数值上都可用。

> 所有模块都提供相同的函数集合。即 `max`、`diff`、
> `divide_and_round_up`、`sqrt` 和 `pow`。

<!-- Custom CSS addition in the theme/custom.css  -->
<div class="modules-table">

| 模块                                                           | 描述                      |
| -------------------------------------------------------------- | ------------------------- |
| [std::u8](https://docs.sui.io/references/framework/std/u8)     | `u8` 类型的函数           |
| [std::u16](https://docs.sui.io/references/framework/std/u16)   | `u16` 类型的函数          |
| [std::u32](https://docs.sui.io/references/framework/std/u32)   | `u32` 类型的函数          |
| [std::u64](https://docs.sui.io/references/framework/std/u64)   | `u64` 类型的函数          |
| [std::u128](https://docs.sui.io/references/framework/std/u128) | `u128` 类型的函数         |
| [std::u256](https://docs.sui.io/references/framework/std/u256) | `u256` 类型的函数         |

</div>

## 导出的地址

标准库导出一个命名地址 - `std = 0x1`。注意别名 `std` 在这里定义。

```toml
[addresses]
std = "0x1"
```

## 隐式导入

一些模块被隐式导入，在模块中无需显式的 `use` 导入即可使用。对于标准库，这些模块和类型包括：

- std::vector
- std::option
- std::option::Option

## 不通过 Sui 框架导入 std

Move 标准库可以直接导入到包中。然而，仅有 `std` 是不足以构建有意义的应用程序的，因为它不提供任何存储功能，也无法与链上状态交互。

```toml
MoveStdlib = { git = "https://github.com/MystenLabs/sui.git", subdir = "crates/sui-framework/packages/move-stdlib", rev = "framework/mainnet" }
```

## 源代码

Move 标准库的源代码可在 [Sui 仓库](https://github.com/MystenLabs/sui/tree/main/crates/sui-framework/packages/move-stdlib/sources) 中找到。