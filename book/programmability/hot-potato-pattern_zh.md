# 模式：烫手山芋

能力系统中的一种情况 - 没有任何能力的结构体 - 被称为_烫手山芋_。它不能被存储（既不能作为[对象](./../storage/key-ability)也不能作为[另一个结构体中的字段](./../storage/store-ability)），不能被[复制](./../move-basics/copy-ability)或[丢弃](./../move-basics/drop-ability)。因此，一旦构造，它必须被其模块[优雅地解包](./../move-basics/struct)，否则交易将因未使用的值没有drop而中止。

> 如果您熟悉支持_回调_的语言，您可以将烫手山芋视为调用回调函数的义务。如果您不调用它，交易将中止。

这个名字来自儿童游戏，球在玩家之间快速传递，当音乐停止时，没有玩家想成为最后一个拿着球的人，否则他们就出局了。这是对该模式的最佳说明 - 烫手山芋结构体的实例在调用之间传递，没有模块可以保留它。

## 定义烫手山芋

烫手山芋可以是任何没有能力的结构体。例如，以下结构体是一个烫手山芋：

```move file=packages/samples/sources/programmability/hot-potato-pattern.move anchor=definition

```

因为`Request`没有能力，不能被存储或忽略，模块必须提供一个函数来解包它。例如：

```move file=packages/samples/sources/programmability/hot-potato-pattern.move anchor=new_request

```

## 示例用法

在以下示例中，`Promise`烫手山芋用于确保从容器中取出的借用值被返回给它。`Promise`结构体包含借用对象的ID和容器的ID，确保借用的值没有被交换为另一个，并返回到正确的容器。

```move file=packages/samples/sources/programmability/hot-potato-pattern.move anchor=container_borrow

```

## 应用

下面我们列出烫手山芋模式的一些常见用例。

### 借用

如[上面的示例](#example-usage)所示，烫手山芋对于借用非常有效，可以保证借用的值返回到正确的容器。虽然示例专注于存储在`Option`中的值，但相同的模式可以应用于任何其他存储类型，比如[动态字段](./dynamic-fields)。

### 闪电贷

烫手山芋模式的典型例子是闪电贷。闪电贷是在同一交易中借入和偿还的贷款。借入的资金用于执行某些操作，偿还的资金返回给贷方。烫手山芋模式确保借入的资金返回给贷方。

这种模式的示例用法可能如下所示：

```move
// 从贷方借入资金。
let (asset_a, potato) = lender.borrow(amount);

// 使用借入的资金执行某些操作。
let asset_b = dex.trade(loan);
let proceeds = another_contract::do_something(asset_b);

// 保留佣金并将其余部分返回给贷方。
let pay_back = proceeds.split(amount, ctx);
lender.repay(pay_back, potato);
transfer::public_transfer(proceeds, ctx.sender());
```

### 可变路径执行

烫手山芋模式可用于在执行路径中引入变化。例如，如果有一个模块允许用"奖励积分"或美元购买`Phone`，烫手山芋可用于将购买与付款解耦。这种方法与一些商店的工作方式非常相似 - 您从货架上拿商品，然后去收银台付款。

```move file=packages/samples/sources/programmability/hot-potato-pattern.move anchor=phone_shop

```

这种解耦技术允许将购买逻辑与付款逻辑分离，使代码更模块化，更易于维护。`Ticket`可以拆分到自己的模块中，为付款提供基本接口，商店实现可以扩展以支持其他商品，而无需更改付款逻辑。

### 组合模式

烫手山芋可用于以组合方式将不同模块链接在一起。其模块可以定义与烫手山芋交互的方式，例如，用类型签名标记它，或从中提取一些信息。这样，烫手山芋可以在不同模块之间传递，甚至在同一交易中的不同包之间传递。

<!-- TODO: add [Request Pattern](./request-pattern) -->

最重要的组合模式是请求模式，我们将在下一节中介绍。

### 在Sui框架中的使用

该模式在Sui框架中以各种形式使用。以下是一些示例：

- [sui::borrow][borrow-framework] - 使用烫手山芋确保借用的值返回到正确的容器。
- [sui::transfer_policy][transfer-policy-framework] - 定义`TransferRequest` - 一个烫手山芋，只有在满足所有条件时才能被消费。
- [sui::token][token-framework] - 在闭环代币系统中，`ActionRequest`携带有关执行操作的信息，并类似于`TransferRequest`收集批准。

[borrow-framework]: https://docs.sui.io/references/framework/sui-framework/borrow
[transfer-policy-framework]: https://docs.sui.io/references/framework/sui-framework/transfer_policy
[token-framework]: https://docs.sui.io/references/framework/sui-framework/token

## 总结

- 烫手山芋是没有能力的结构体，它必须配备创建和销毁它的方法。
- 烫手山芋用于确保在交易结束前采取某些行动，类似于回调。
- 烫手山芋最常见的用例是借用、闪电贷、可变路径执行和组合模式。