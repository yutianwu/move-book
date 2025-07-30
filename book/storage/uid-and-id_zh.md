# UID和ID

`UID`类型在`sui::object`模块中定义，是`ID`的包装器，而`ID`反过来包装了`address`类型。Sui上的UID保证是唯一的，在对象被删除后不能重用。

```move
module sui::object;

/// UID是对象的唯一标识符
public struct UID has store {
    id: ID
}

/// ID是地址的包装器
public struct ID has store, drop {
    bytes: address
}
```

<!-- User doesn't know anything about TxContext yet... -->

## 新鲜UID生成：

- UID从`tx_hash`和一个为每个新UID递增的`index`派生。
- `derive_id`函数在`sui::tx_context`模块中实现，这就是为什么UID生成需要TxContext的原因。
- Sui验证器不允许使用不是在同一函数中创建的UID。这防止UID被预生成并在对象被解包后重用。

新的UID通过`object::new(ctx)`函数创建。它接受TxContext的可变引用，并返回新的UID。

```move
let ctx = &mut tx_context::dummy();
let uid = object::new(ctx);
```

在Sui上，`UID`充当对象的表示，并允许定义对象的行为和特征。关键特征之一——[动态字段](./../programmability/dynamic-fields)——因为`UID`类型的显式性而成为可能。此外，它允许[转移到对象(TTO)](https://docs.sui.io/concepts/transfers/transfer-to-object)，我们将在本章后面解释。

## UID生命周期

`UID`类型通过`object::new(ctx)`函数创建，通过`object::delete(uid)`函数销毁。`object::delete`_按值_消费UID，除非值从对象中解包，否则无法删除它。

```move
let ctx = &mut tx_context::dummy();

let char = Character {
    id: object::new(ctx)
};

let Character { id } = char;
id.delete();
```

## 保持UID

`UID`在对象结构体解包后不需要立即删除。有时它可能携带[动态字段](./../programmability/dynamic-fields)或通过[转移到对象](./transfer-to-object)转移给它的对象。在这种情况下，UID可能被保留并存储在单独的对象中。

## 删除证明

返回对象的UID的能力可以在称为_删除证明_的模式中使用。这是一种很少使用的技术，但在某些情况下可能有用，例如，创建者或应用程序可能通过将删除的ID交换为某种奖励来激励删除对象。

在框架开发中，这种方法可以用来忽略/绕过"获取"对象的某些限制。如果有一个容器在转移上强制执行某些逻辑，比如Kiosk所做的，可能有一个通过提供删除证明来跳过检查的特殊场景。

这是探索和研究的开放主题之一，可以以各种方式使用。

## ID

在谈论`UID`时，我们也应该提到`ID`类型。它是`address`类型的包装器，用于表示地址指针。通常，`ID`用于指向对象，但是，没有限制，也不保证`ID`指向现有对象。

> ID可以作为交易参数在[交易块](./../concepts/what-is-a-transaction)中接收。或者，可以使用`to_id()`函数从`address`值创建ID。

<!--
```move

TODO: !!!!

```
-->

## fresh_object_address

TxContext提供`fresh_object_address`函数，可以用来创建唯一地址和`ID`——这在为用户操作分配唯一标识符的某些应用程序中可能有用——例如，市场中的order_id。