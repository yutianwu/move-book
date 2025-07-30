# 能力：介绍

Move 具有独特的类型系统，允许自定义_类型能力_。
[在前面的章节中](./struct)，我们介绍了 `struct` 的定义及其使用方法。
然而，`Artist` 和 `Record` 结构体的实例必须被解包才能使代码编译通过。这是没有_能力_的结构体的默认行为。

> 在本书中，你会看到名为 `Ability: <name>` 的章节，其中 `<name>` 是能力的名称。这些章节将详细介绍该能力，它是如何工作的，以及如何在 Move 中使用它。

## 什么是能力？

能力是一种允许类型具有特定行为的方式。它们是结构体声明的一部分，定义了结构体实例允许的行为。

## 能力语法

能力通过在结构体定义中使用 `has` 关键字后跟能力列表来设置。
能力之间用逗号分隔。Move 支持 4 种能力：`copy`、`drop`、`key` 和 `store`。每种能力为结构体实例定义了特定的行为。

```move
/// 这个结构体具有 `copy` 和 `drop` 能力。
public struct VeryAble has copy, drop {
    // field: Type1,
    // field2: Type2,
    // ...
}
```

## 概述

能力的快速概述：

> 除了[引用](references)之外，所有内置类型都具有 `copy`、`drop` 和 `store` 能力。引用具有 `copy` 和 `drop` 能力。

- `copy` - 允许结构体被_复制_。在 [Ability: Copy](./copy-ability) 章节中解释。
- `drop` - 允许结构体被_丢弃_或_废弃_。在 [Ability: Drop](./drop-ability) 章节中解释。
- `key` - 允许结构体在存储中作为_键_使用。在 [Ability: Key](./../storage/key-ability) 章节中解释。
- `store` - 允许结构体被_存储_在具有 _key_ 能力的结构体中。在 [Ability: Store](./../storage/store-ability) 章节中解释。

虽然在这里简要提及它们很重要，但我们将在后续章节中详细介绍每种能力，并提供如何使用它们的适当上下文。

## 无能力

没有能力的结构体不能被丢弃、复制或存储在存储中。我们称这样的结构体为_烫手山芋_。这是一个幽默的名称，但它很好地提醒我们，没有能力的结构体就像烫手山芋一样——只能被传递，需要特殊处理。烫手山芋是 Move 中最强大的模式之一，我们在 [Hot Potato Pattern](./../programmability/hot-potato-pattern) 章节中详细介绍了它。

## 进一步阅读

- Move 参考中的[类型能力](./../../reference/abilities)。