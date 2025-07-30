# 存储函数

定义主要存储操作的模块是`sui::transfer`。它在所有依赖于[Sui框架](./../programmability/sui-framework)的包中都被隐式导入，所以，像其他隐式导入的模块（例如`std::option`或`std::vector`）一样，它不需要添加use语句。

## 概述

`transfer`模块提供函数来执行与我们解释的[所有权类型](./../object/ownership)匹配的所有三种存储操作：

> 在此页面上，我们只讨论所谓的_受限_存储操作，稍后我们将在引入`store`能力后涵盖_公共_操作。

1. _转移_ - 将对象发送到地址，将其置于_账户拥有_状态；
2. _共享_ - 将对象置于_共享_状态，使其对所有人可用；
3. _冻结_ - 将对象置于_不可变_状态，使其成为公共常量且永远不能更改。

`transfer`模块是大多数存储操作的首选，除了等待我们在下一章中讨论的[动态字段](./../programmability/dynamic-fields)的特殊情况。

## 所有权和引用：快速回顾

在[所有权和作用域](./../move-basics/ownership-and-scope)和[引用](./../move-basics/references)章节中，我们涵盖了Move中所有权和引用的基础知识。当使用存储函数时，理解这些概念很重要。以下是最重要要点的快速回顾：

- Move中的_移动_语义意味着值从一个作用域_移动_到另一个作用域。换句话说，如果类型的实例_按值_传递给函数，它会被_移动_到函数作用域，不能再在调用者作用域中访问。
- 要保持值的所有权，您可以_按引用_传递它。通过_不可变引用_`&T`或_可变引用_`&mut T`。然后值被_借用_，可以在被调用者作用域中访问，但所有者保持不变。

```move
/// 按值移动
public fun take<T>(value: T) { /* 值在这里被移动！ */ abort 0 }

/// 对于不可变引用
public fun borrow<T>(value: &T) { /* 值在这里被借用！可以读取 */ abort 0 }

/// 对于可变引用
public fun borrow_mut<T>(value: &mut T) { /* 值在这里被可变借用！ */ abort 0 }
```

<!-- TODO part on:
    - object does not have an associated storage type
    - the same type of object can be stored differently
    - the objects must be specified in the transaction by their ID
 -->

## 转移

`transfer::transfer`函数是用于将对象转移到另一个地址的公共函数。其签名如下，只接受具有[`key`能力](./key-ability)的类型和接收者的[地址](./../move-basics/address)。请注意，对象_按值_传递到函数中，因此它被_移动_到函数作用域，然后移动到接收者地址：

```move
module sui::transfer;

public fun transfer<T: key>(obj: T, recipient: address);
```

在下一个示例中，您可以看到如何在定义对象并将其发送给交易发送者的模块中使用它。

```move
module book::transfer_to_sender;

/// 具有`key`的结构体是对象。第一个字段是`id: UID`！
public struct AdminCap has key { id: UID }

/// `init`函数是模块发布时调用的特殊函数。
/// 这是创建应用程序对象的好地方。
fun init(ctx: &mut TxContext) {
    // 在此作用域中创建新的`AdminCap`对象。
    let admin_cap = AdminCap { id: object::new(ctx) };

    // 将对象转移给交易发送者。
    transfer::transfer(admin_cap, ctx.sender());

    // admin_cap消失了！不能再访问。
}

/// 将`AdminCap`对象转移给`recipient`。因此，接收者
/// 成为对象的所有者，只有他们可以访问它。
public fun transfer_admin_cap(cap: AdminCap, recipient: address) {
    transfer::transfer(cap, recipient);
}
```

当模块发布时，`init`函数将被调用，我们在那里创建的`AdminCap`对象将被_转移_给交易发送者。`ctx.sender()`函数返回当前交易的发送者地址。

一旦`AdminCap`被转移给发送者，例如，转移给`0xa11ce`，发送者，且只有发送者，将能够访问该对象。对象现在是_账户拥有_的。

> 账户拥有的对象受_真正所有权_约束——只有账户所有者可以访问它们。这是Sui存储模型中的基本概念。

让我们扩展示例，添加一个使用`AdminCap`来授权铸造新对象并将其转移到另一个地址的函数：

```move
/// 管理员可以`mint_and_transfer`的某个`Gift`对象。
public struct Gift has key { id: UID }

/// 创建新的`Gift`对象并将其转移给`recipient`。
public fun mint_and_transfer(
    _: &AdminCap, recipient: address, ctx: &mut TxContext
) {
    let gift = Gift { id: object::new(ctx) };
    transfer::transfer(gift, recipient);
}
```

`mint_and_transfer`函数是一个"可以"被任何人调用的公共函数，但它需要将`AdminCap`对象作为第一个参数按引用传递。没有它，函数将不可调用。这是限制对特权函数访问的简单方法，称为_[能力](./../programmability/capability)_。因为`AdminCap`对象是_账户拥有_的，只有`0xa11ce`能够调用`mint_and_transfer`函数。

发送给接收者的`Gift`也将是_账户拥有_的，每个礼物都是独特的，由接收者专门拥有。

快速回顾：

- `transfer`函数用于将对象发送到地址；
- 对象变为_账户拥有_，只能由接收者访问；
- 函数可以通过要求传递对象作为参数来设置门控，创建_能力_。

## 冻结

`transfer::freeze_object`函数是用于将对象置于_不可变_状态的公共函数。一旦对象被_冻结_，它永远不能更改，任何人都可以通过不可变引用访问它。

函数签名如下，只接受具有[`key`能力](./key-ability)的类型。就像所有其他存储函数一样，它_按值_接受对象：

```move
module sui::transfer;

public fun freeze_object<T: key>(obj: T);
```

让我们扩展前面的示例，添加一个允许管理员创建`Config`对象并冻结它的函数：

```move
/// 管理员可以`create_and_freeze`的某个`Config`对象。
public struct Config has key {
    id: UID,
    message: String
}

/// 创建新的`Config`对象并冻结它。
public fun create_and_freeze(
    _: &AdminCap,
    message: String,
    ctx: &mut TxContext
) {
    let config = Config {
        id: object::new(ctx),
        message
    };

    // 冻结对象使其变为不可变。
    transfer::freeze_object(config);
}

/// 从`Config`对象返回消息。
/// 可以通过不可变引用访问对象！
public fun message(c: &Config): String { c.message }
```

Config是一个具有`message`字段的对象，`create_and_freeze`函数创建新的`Config`并冻结它。一旦对象被冻结，任何人都可以通过不可变引用访问它。`message`函数是一个从`Config`对象返回消息的公共函数。Config现在通过其ID公开可用，任何人都可以读取消息。

> 函数定义与对象的状态无关。可以定义一个接受可变引用的函数，该引用用于用作冻结的对象。但是，它不能在冻结对象上调用。

`message`函数可以在不可变的`Config`对象上调用，但是，下面的两个函数不能在冻结对象上调用：

```move
// === 下面的函数不能在冻结对象上调用！ ===

/// 该函数可以定义，但不能在冻结对象上调用。
/// 只允许不可变引用。
public fun message_mut(c: &mut Config): &mut String { &mut c.message }

/// 删除`Config`对象，按值接受它。
/// 不能在冻结对象上调用！
public fun delete_config(c: Config) {
    let Config { id, message: _ } = c;
    id.delete()
}
```

总结：

- `transfer::freeze_object`函数用于将对象置于_不可变_状态；
- 一旦对象被_冻结_，它永远不能更改、删除或转移，任何人都可以通过不可变引用访问它；

## 拥有 -> 冻结

由于`transfer::freeze_object`签名接受任何具有`key`能力的类型，它可以接受在同一作用域中创建的对象，但也可以接受由账户拥有的对象。这意味着`freeze_object`函数可以用来_冻结_被_转移_给发送者的对象。出于安全考虑，我们不希望冻结`AdminCap`对象——允许任何人访问它将是安全风险。但是，我们可以冻结被铸造并转移给接收者的`Gift`对象：

> 单一所有者 -> 不可变转换是可能的！

```move
/// 冻结`Gift`对象使其变为不可变。
public fun freeze_gift(gift: Gift) {
    transfer::freeze_object(gift);
}
```

## 共享

`transfer::share_object`函数是用于将对象置于_共享_状态的公共函数。一旦对象被_共享_，任何人都可以通过可变引用（因此，也包括不可变引用）访问它。函数签名如下，只接受具有[`key`能力](./key-ability)的类型：

```move
module sui::transfer;

public fun share_object<T: key>(obj: T);
```

一旦对象被_共享_，它就作为可变引用公开可用。

## 特殊情况：共享对象删除

虽然共享对象通常不能按值获取，但有一个特殊情况可以——如果获取它的函数删除了对象。这是Sui存储模型中的特殊情况，用于允许删除共享对象。为了展示它如何工作，我们将创建一个函数来创建和共享Config对象，然后另一个函数来删除它：

```move
/// 创建新的`Config`对象并共享它。
public fun create_and_share(message: String, ctx: &mut TxContext) {
    let config = Config {
        id: object::new(ctx),
        message
    };

    // 共享对象使其变为共享。
    transfer::share_object(config);
}
```

`create_and_share`函数创建新的`Config`对象并共享它。对象现在作为可变引用公开可用。让我们创建一个删除共享对象的函数：

```move
/// 删除`Config`对象，按值接受它。
/// 可以在共享对象上调用！
public fun delete_config(c: Config) {
    let Config { id, message: _ } = c;
    id.delete()
}
```

`delete_config`函数按值接受`Config`对象并删除它，Sui验证器将允许此调用。但是，如果函数将`Config`对象返回或尝试`freeze`或`transfer`它，Sui验证器将拒绝交易。

```move
// 不行！
public fun transfer_shared(c: Config, to: address) {
    transfer::transfer(c, to);
}
```

总结：

- `share_object`函数用于将对象置于_共享_状态；
- 一旦对象被_共享_，任何人都可以通过可变引用访问它；
- 共享对象可以被删除，但不能被转移或冻结。

## 下一步

现在您知道了`transfer`模块的主要功能，您可以开始在Sui上构建涉及存储操作的更复杂的应用程序。在下一章中，我们将涵盖[Store能力](./store-ability)，它允许在对象内存储数据并放宽我们这里几乎没有涉及的转移限制。之后，我们将涵盖[UID和ID](./uid-and-id)类型，这是Sui存储模型中最重要的类型。