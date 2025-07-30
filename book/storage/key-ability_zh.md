# Key能力

在[基本语法](./../move-basics)章节中，我们已经涵盖了四种能力中的两种——[Drop](./../move-basics/drop-ability)和[Copy](./../move-basics/copy-ability)。它们影响值在作用域中的行为，与存储没有直接关系。现在是时候涵盖`key`能力了，它允许结构体被存储。

从历史上看，`key`能力是为了将类型标记为_存储中的键_而创建的。具有`key`能力的类型可以存储在存储的顶级，并且可以被账户或地址_直接拥有_。随着[对象模型](./../object)的引入，`key`能力自然成为了对象的定义能力。

<!-- TODO: What is Sui Verifier - link, later -->

## 对象定义

具有`key`能力的结构体被视为对象，可以在存储函数中使用。Sui验证器将要求结构体的第一个字段命名为`id`并具有`UID`类型。

```move
public struct Object has key {
    id: UID, // 必需
    name: String,
}

/// 创建一个具有唯一ID的新对象
public fun new(name: String, ctx: &mut TxContext): Object {
    Object {
        id: object::new(ctx), // 创建一个新的UID
        name,
    }
}
```

具有`key`能力的结构体仍然是结构体，可以有任意数量的字段和关联函数。打包、访问或解包结构体没有特殊的处理或语法。

然而，由于对象结构体的第一个字段必须是`UID`类型——一个不可复制和不可丢弃的类型（我们很快就会讲到！），结构体在传递上不能拥有`drop`和`copy`能力。因此，对象在设计上是不可丢弃的。

<!-- ## Asset Definition

In the context of the [Object Model](./../object/digital-assets), an object with the `key` ability can be considered an asset. It is non-discardable, unique, and can be *owned*.
 -->

## 具有`key`能力的类型

由于具有`key`的类型需要`UID`，Move中的原生类型都不能拥有`key`能力，[标准库](./../move-basics/standard-library)类型也不能。`key`能力只存在于[Sui框架](./../programmability/sui-framework)和自定义类型中。

## 下一步

Key能力在Move中定义对象，对象旨在被_存储_。在下一节中，我们介绍`sui::transfer`模块，它为对象提供原生存储函数。

## 延伸阅读

- Move参考中的[类型能力](./../../reference/abilities)。