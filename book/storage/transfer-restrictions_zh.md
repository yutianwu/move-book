# 受限和公共转移

我们在[前面章节](./storage-functions)中描述的存储操作默认是受限的——它们只能在定义对象的模块中调用。换句话说，类型必须是模块的_内部_类型才能在存储操作中使用。这种限制在Sui验证器中实现，并在字节码级别强制执行。

但是，为了允许对象在其他模块中被转移和存储，可以放宽这些限制。`sui::transfer`模块提供了一组_public\_\*_函数，允许在其他模块中调用存储操作。这些函数以`public_`为前缀，对所有模块和交易都可用。

## 公共存储操作

`sui::transfer`模块提供以下公共函数。它们与我们已经涵盖的函数几乎相同，但可以从任何模块调用。

```move
module sui::transfer;

/// `transfer`函数的公共版本。
public fun public_transfer<T: key + store>(object: T, to: address) {}

/// `share_object`函数的公共版本。
public fun public_share_object<T: key + store>(object: T) {}

/// `freeze_object`函数的公共版本。
public fun public_freeze_object<T: key + store>(object: T) {}
```

为了说明这些函数的用法，考虑以下示例：模块A定义了具有`key`和`key + store`能力的ObjectK和ObjectKS，模块B尝试为这些对象实现`transfer`函数。

> 在此示例中我们使用`transfer::transfer`，但`share_object`和`freeze_object`函数的行为是相同的。

```move
/// 分别定义具有`key`和`key + store`能力的`ObjectK`和`ObjectKS`
module book::transfer_a;

public struct ObjectK has key { id: UID }
public struct ObjectKS has key, store { id: UID }
```

```move
/// 从`transfer_a`导入`ObjectK`和`ObjectKS`类型，并尝试
/// 为它们实现不同的`transfer`函数
module book::transfer_b;

// 类型不是此模块的内部类型
use book::transfer_a::{ObjectK, ObjectKS};

// 失败！ObjectK不是`store`，且ObjectK不是此模块的内部类型
public fun transfer_k(k: ObjectK, to: address) {
    transfer::transfer(k, to);
}

// 失败！ObjectKS有`store`但函数不是公共的
public fun transfer_ks(ks: ObjectKS, to: address) {
    transfer::transfer(ks, to);
}

// 失败！ObjectK不是`store`，`public_transfer`需要`store`
public fun public_transfer_k(k: ObjectK, to: address) {
    transfer::public_transfer(k, to);
}

// 成功！ObjectKS有`store`且函数是公共的
public fun public_transfer_ks(ks: ObjectKS, to: address) {
    transfer::public_transfer(ks, to);
}
```

扩展上面的示例：

- ❌ `transfer_k`失败，因为ObjectK不是模块`transfer_b`的内部类型
- ❌ `transfer_ks`失败，因为ObjectKS不是模块`transfer_b`的内部类型
- ❌ `public_transfer_k`失败，因为ObjectK没有`store`能力
- ✅ `public_transfer_ks`成功，因为ObjectKS有`store`能力且转移是公共的

## `store`的含义

是否向类型添加`store`能力的决定应该谨慎做出。一方面，这实际上是类型被其他应用程序_使用_的要求。另一方面，它允许_包装_和更改预期的存储模型。例如，角色可能预期由账户拥有，但有了`store`能力，它可以被冻结（不能共享——这种转换是受限的）。