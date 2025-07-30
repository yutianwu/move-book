---
title: 'Sui对象 | 参考'
---

# Sui对象

对于Sui，`key`用来表示一个_对象_。对象是在Sui中存储数据的唯一方式——允许数据在交易之间持久化。

有关更多详细信息，请参阅Sui文档：

- [对象模型](https://docs.sui.io/concepts/object-model)
- [对象的Move规则](https://docs.sui.io/concepts/sui-move-concepts#global-unique)
- [转移对象](https://docs.sui.io/concepts/transfers)

## 对象规则

对象是具有[`key`](../abilities_zh.md#key)能力的[`struct`](../structs_zh.md)。结构体的第一个字段必须是`id: sui::object::UID`。这个32字节的字段（[`address`](../primitive-types/address_zh.md)的强类型包装器）用于唯一标识对象。

注意，由于`sui::object::UID`只有`store`能力（它没有`copy`或`drop`），所以没有对象具有`copy`或`drop`。

## 转移规则

对象可以在`sui::transfer`模块中改变所有权和转移。模块中的许多函数都有"公共"和"私有"变体，其中"私有"变体只能在定义对象类型的模块内部调用。"公共"变体只有在对象具有`store`时才能调用。

例如，如果我们在模块`my_module`中定义了两个对象`A`和`B`：

```move
module a::my_module;

public struct A has key {
    id: sui::object::UID,
}

public struct B has key, store {
    id: sui::object::UID,
}
```

`A`只能在`a::my_module`内部使用`sui::transfer::transfer`转移，而`B`可以在任何地方使用`sui::transfer::public_transfer`转移。这些规则通过Sui中的自定义类型系统（字节码验证器）规则强制执行。