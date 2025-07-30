# 交易上下文

每个交易都有执行上下文。上下文是在执行期间对程序可用的一组预定义变量。例如，每个交易都有一个发送者地址，交易上下文包含一个保存发送者地址的变量。

交易上下文通过`TxContext`结构体对程序可用。该结构体在[`sui::tx_context`][tx-context-framework]模块中定义，包含以下字段：

[tx-context-framework]: https://docs.sui.io/references/framework/sui/tx_context

```move
module sui::tx_context;

/// Information about the transaction currently being executed.
/// This cannot be constructed by a transaction--it is a privileged object created by
/// the VM and passed in to the entrypoint of the transaction as `&mut TxContext`.
public struct TxContext has drop {
    /// The address of the user that signed the current transaction
    sender: address,
    /// Hash of the current transaction
    tx_hash: vector<u8>,
    /// The current epoch number
    epoch: u64,
    /// Timestamp that the epoch started at
    epoch_timestamp_ms: u64,
    /// Counter recording the number of fresh id's created while executing
    /// this transaction. Always 0 at the start of a transaction
    ids_created: u64
}
```

交易上下文不能手动构造或直接修改。它由系统创建，并在交易中作为引用传递给函数。在[交易](./../concepts/what-is-a-transaction)中调用的任何函数都可以访问上下文，并可以将其传递到嵌套调用中。

> `TxContext`必须是函数签名中的最后一个参数。

## 读取交易上下文

除了`ids_created`之外，`TxContext`中的所有字段都有getter。getter在`sui::tx_context`模块中定义，对程序可用。getter不需要`&mut`，因为它们不修改上下文。

```move file=packages/samples/sources/programmability/transaction-context.move anchor=reading

```

## 可变性

`TxContext`需要在系统中创建新对象（或只是`UID`）。新的UID从交易摘要派生，为了摘要唯一，需要有一个变化的参数。Sui为此使用`ids_created`字段。每次创建新UID时，`ids_created`字段都会增加1。这样，摘要始终是唯一的。

在内部，它表示为`derive_id`函数：

```move
native fun derive_id(tx_hash: vector<u8>, ids_created: u64): address;
```

## 生成唯一地址

底层的`derive_id`函数也可以在您的程序中用于生成唯一地址。函数本身不暴露，但在`sui::tx_context`模块中提供了包装器函数`fresh_object_address`。如果您需要在程序中生成唯一标识符，这可能很有用。

```move
module sui::tx_context;

/// Create an `address` that has not been used. As it is an object address, it will never
/// occur as the address for a user.
/// In other words, the generated address is a globally unique object ID.
public fun fresh_object_address(ctx: &mut TxContext): address {
    let ids_created = ctx.ids_created;
    let id = derive_id(*&ctx.tx_hash, ids_created);
    ctx.ids_created = ids_created + 1;
    id
}
```