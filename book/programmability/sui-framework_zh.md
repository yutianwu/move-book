# Sui框架

Sui框架是[包清单](./../concepts/manifest)中的默认依赖集。它依赖于[标准库](./../move-basics/standard-library)并提供Sui特定的功能，包括与存储的交互，以及Sui特定的原生类型和模块。

_为了方便起见，我们将Sui框架中的模块分为多个类别。但它们仍然是同一框架的一部分。_

## 核心

<!-- Custom CSS addition in the theme/custom.css  -->
<div class="modules-table">

| 模块                                                                                         | 描述                                                             | 章节                                          |
| ---------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- | ------------------------------------------------ |
| [sui::address](https://docs.sui.io/references/framework/sui/address)                           | 为[address类型](./../move-basics/address)添加转换方法 | [地址](./../move-basics/address)              |
| [sui::transfer](https://docs.sui.io/references/framework/sui/transfer)                         | 实现对象的存储操作                           | [从对象开始](./../object)          |
| [sui::tx_context](https://docs.sui.io/references/framework/sui/tx_context)                     | 包含`TxContext`结构体和读取它的方法                  | [交易上下文](./transaction-context)     |
| [sui::object](https://docs.sui.io/references/framework/sui/object)                             | 定义创建对象所需的`UID`和`ID`类型          | [从对象开始](./../object)          |
| [sui::clock](https://docs.sui.io/references/framework/sui/clock)                               | 定义`Clock`类型及其方法                                | [时期和时间](./epoch-and-time)               |
| [sui::dynamic_field](https://docs.sui.io/references/framework/sui/dynamic_field)               | 实现添加、使用和删除动态字段的方法                | [动态字段](./dynamic-fields)               |
| [sui::dynamic_object_field](https://docs.sui.io/references/framework/sui/dynamic_object_field) | 实现添加、使用和删除动态对象字段的方法         | [动态对象字段](./dynamic-object-fields) |
| [sui::event](https://docs.sui.io/references/framework/sui/event)                               | 允许为链下监听器发出事件                          | [事件](./events)                               |
| [sui::package](https://docs.sui.io/references/framework/sui/package)                           | 定义`Publisher`类型和包升级方法                | [Publisher](./publisher), 包升级       |
| [sui::display](https://docs.sui.io/references/framework/sui/display)                           | 实现`Display`对象以及创建和更新它的方法        | [显示](./display)                             |

</div>

## 集合

<div class="modules-table">

| 模块                                                                         | 描述                                                       | 章节                                      |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------- | -------------------------------------------- |
| [sui::vec_set](https://docs.sui.io/references/framework/sui/vec_set)           | 实现集合类型                                             | [集合](./collections)                 |
| [sui::vec_map](https://docs.sui.io/references/framework/sui/vec_map)           | 实现带向量键的映射                                 | [集合](./collections)                 |
| [sui::table](https://docs.sui.io/references/framework/sui/table)               | 实现`Table`类型和与之交互的方法       | [动态集合](./dynamic-collections) |
| [sui::linked_table](https://docs.sui.io/references/framework/sui/linked_table) | 实现`LinkedTable`类型和与之交互的方法 | [动态集合](./dynamic-collections) |
| [sui::bag](https://docs.sui.io/references/framework/sui/bag)                   | 实现`Bag`类型和与之交互的方法         | [动态集合](./dynamic-collections) |
| [sui::object_table](https://docs.sui.io/references/framework/sui/object_table) | 实现`ObjectTable`类型和与之交互的方法 | [动态集合](./dynamic-collections) |
| [sui::object_bag](https://docs.sui.io/references/framework/sui/object_bag)     | 实现`ObjectBag`类型和与之交互的方法   | [动态集合](./dynamic-collections) |

</div>

## 实用程序

<div class="modules-table">

| 模块                                                             | 描述                                                | 章节                                 |
| ------------------------------------------------------------------ | ---------------------------------------------------------- | --------------------------------------- |
| [sui::bcs](https://docs.sui.io/references/framework/sui/bcs)       | 实现BCS编码和解码函数         | [二进制规范序列化](./bcs) |
| [sui::borrow](https://docs.sui.io/references/framework/sui/borrow) | 实现按_值_借用的借用机制 | [烫手山芋](./hot-potato-pattern)      |
| [sui::hex](https://docs.sui.io/references/framework/sui/hex)       | 实现十六进制编码和解码函数         | -                                       |
| [sui::types](https://docs.sui.io/references/framework/sui/types)   | 提供检查类型是否为一次性见证的方法 | [一次性见证](./one-time-witness)  |

</div>

## 导出地址

Sui框架导出两个命名地址：`sui = 0x2`和来自std依赖的`std = 0x1`。

```toml
[addresses]
sui = "0x2"

# 从MoveStdlib依赖导出
std = "0x1"
```

## 隐式导入

就像[标准库](./../move-basics/standard-library#implicit-imports)一样，Sui框架中的一些模块和类型是隐式导入的。这是在不显式`use`导入的情况下可用的模块和类型列表：

- sui::object
- sui::object::ID
- sui::object::UID
- sui::tx_context
- sui::tx_context::TxContext
- sui::transfer

## 源代码

Sui框架的源代码可在[Sui存储库](https://github.com/MystenLabs/sui/tree/main/crates/sui-framework/packages/sui-framework/sources)中获得。

<!--

Modules:

Coins:
- sui::pay
- sui::sui
- sui::coin
- sui::token
- sui::balance
- sui::deny_list

Commerce:
- sui::kiosk
- sui::display
- sui::kiosk_extension
- sui::transfer_policy


Utilities:
+ sui::bcs
+ sui::hex
- sui::math (deprecated)
+ sui::types
+ sui::borrow


- sui::authenticator

- sui::priority_queue
- sui::table_vec

- sui::url
- sui::versioned

- sui::prover
- sui::random

- sui::bls12381
- sui::ecdsa_k1
- sui::ecdsa_r1
- sui::ecvrf
- sui::ed25519
(also mention verifier 16 growth)
- sui::group_ops
- sui::hash
- sui::hmac
- sui::poseidon
- sui::zklogin_verified_id
- sui::zklogin_verified_issuer

 -->