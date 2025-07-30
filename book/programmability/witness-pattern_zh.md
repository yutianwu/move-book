# 模式：见证

见证是通过构造证明来证明存在性的模式。在编程的上下文中，见证是通过提供只有在属性成立时才能构造的值来证明系统某种属性的方式。

## Move中的见证

在[结构体](./../move-basics/struct)部分中，我们展示了结构体只能由定义它的模块创建 - 或者说_打包_。因此，在Move中，模块通过构造类型来证明对类型的所有权。这是Move中最重要的模式之一，广泛用于泛型类型实例化和授权。

实际上，为了使用见证，必须有一个期望见证作为参数的函数。在下面的示例中，`new`函数期望类型`T`的见证来创建`Instance<T>`实例。

> 通常情况下，见证结构体不被存储，因此函数可能要求类型具有[Drop](./../move-basics/drop-ability)能力。

```move
module book::witness;

/// A struct that requires a witness to be created.
public struct Instance<T> { t: T }

/// Create a new instance of `Instance<T>` with the provided T.
public fun new<T>(witness: T): Instance<T> {
    Instance { t: witness }
}
```

构造`Instance<T>`的唯一方法是使用类型`T`的实例调用`new`函数。这是Move中见证模式的基本示例。提供见证的模块通常有匹配的实现，如下面的模块`book::witness_source`：

```move
module book::witness_source;

use book::witness::{Self, Instance};

/// A struct used as a witness.
public struct W {}

/// Create a new instance of `Instance<W>`.
public fun new_instance(): Instance<W> {
    witness::new(W {})
}
```

结构体`W`的实例被传递到`new_instance`函数中以创建`Instance<W>`，从而证明模块`book::witness_source`拥有类型`W`。

## 实例化泛型类型

见证允许泛型类型用具体类型实例化。这对于从类型继承关联行为很有用，如果模块提供了这样做的能力，还可以选择扩展它们。

```move
module sui::balance;

/// A Supply of T. Used for minting and burning.
public struct Supply<phantom T> has store {
    value: u64,
}

/// Create a new supply for type T with the provided witness.
public fun create_supply<T: drop>(_w: T): Supply<T> {
    Supply { value: 0 }
}

/// Get the `Supply` value.
public fun supply_value<T>(supply: &Supply<T>): u64 {
    supply.value
}
```

在上面的示例中，这是从[Sui框架](./sui-framework)的[`balance`模块][balance-framework]借用的，`Supply`是一个泛型结构体，只能通过提供类型`T`的见证来构造。见证按值获取并被_丢弃_ - 因此`T`必须具有[drop](./../move-basics/drop-ability)能力。

[balance-framework]: https://docs.sui.io/references/framework/sui/balance

然后，实例化的`Supply<T>`可以用来铸造新的`Balance<T>`，其中`T`是供应的类型。

```move
module sui::balance;

/// Storable balance.
public struct Balance<phantom T> has store {
    value: u64,
}

/// Increase supply by `value` and create a new `Balance<T>` with this value.
public fun increase_supply<T>(self: &mut Supply<T>, value: u64): Balance<T> {
    assert!(value < (18446744073709551615u64 - self.value), EOverflow);
    self.value = self.value + value;
    Balance { value }
}
```

## 一次性见证

虽然结构体可以创建任意次数，但在某些情况下，结构体应该保证只创建一次。为此，Sui提供了"一次性见证" - 一种只能使用一次的特殊见证。我们在[下一节](./one-time-witness)中详细解释它。

## 总结

- 见证是通过构造证明来证明某种属性的模式。
- 在Move中，模块通过构造类型来证明对类型的所有权。
- 见证经常用于泛型类型实例化和授权。

## 下一步

在下一节中，我们将学习[一次性见证](./one-time-witness)模式。