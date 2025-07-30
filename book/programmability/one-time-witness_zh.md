# 一次性见证

虽然常规[见证](./witness-pattern)是静态证明类型所有权的好方法，但在某些情况下，我们需要确保见证只实例化一次。这就是一次性见证（OTW）的目的。

<!--
Notes to self:
  - background first or definition first - which one is better?
  - why would someone read this section?
  - if we removed the OTW from docs, then we should give definition first.
-->

## 定义

OTW是一种特殊类型的见证，只能使用一次。它不能手动创建，并且保证每个模块唯一。如果类型遵循以下规则，Sui适配器将其视为OTW：

1. 只有`drop`能力。
2. 没有字段。
3. 不是泛型类型。
4. 以模块名称命名，全部大写字母。

这是OTW的示例：

```move file=packages/samples/sources/programmability/one-time-witness.move anchor=definition

```

OTW不能手动构造，任何尝试这样做的代码都会导致编译错误。OTW可以作为[模块初始化器](./module-initializer)中的第一个参数接收。由于`init`函数每个模块只调用一次，因此OTW保证只实例化一次。

## 强制执行OTW

要检查类型是否为OTW，[Sui框架](./sui-framework)的`sui::types`模块提供了一个特殊函数`is_one_time_witness`，可用于检查类型是否为OTW。

```move file=packages/samples/sources/programmability/one-time-witness.move anchor=usage

```

<!-- ## Background

Before we get to actual definition of the OTW, let's consider a simple example. We want to build a generic implementation of a Coin type, which can be initialized with a witness. A instance of a witness `T` is used to create a new `TreasuryCap<T>` which is then used to mint a new `Coin<T>`.

```move
module book::simple_coin {

    /// Controls the supply of the Coin.
    public struct TreasuryCap<phantom T> has key, store {
        id: UID,
        total_supply: u64,
    }

    /// The Coin type where the `T` is a witness.
    public struct Coin<phantom T> has key, store {
        id: UID,
        value: u64,
    }

    /// Create a new TreasuryCap with a witness.
    /// Vulnerable: we can create multiple TreasuryCap<T> with the same witness.
    public fun new<T: drop>(_: T, ctx: &mut TxContext): TreasuryCap<T> {
        TreasuryCap { id: object::new(ctx), total_supply: 0 }
    }

    /// We use a regular witness to authorize the minting.
    public fun mint<T>(
        treasury: &mut TreasuryCap<T>,
        value: u64,
        ctx: &mut TxContext
    ) {
        treasury.total_supply = treasury.total_supply + value;
        Coin { id: object::new(ctx), value }
    }
}
```

A dishonest developer would be able to create multiple `TreasuryCap`s with the same witness, and mint more `Coin`s than expected. Here is an example of such a malicious module:

```move
module book::simple_coin_cheater {
    /// The Coin witness.
    public struct Move has drop {}

    /// Initialize the TreasuryCap with the Move witness.
    /// ...and do it twice! >_<
    fun init(ctx: &mut TxContext) {
        let treasury_cap = book::simple_coin::new(Move {}, ctx);
        let secret_treasury = book::simple_coin::new(Move {}, ctx);

        transfer::public_transfer(treasury_cap, ctx.sender())
        transfer::public_transfer(secret_treasury, ctx.sender())
    }
}

```

The example above has no protection against issuing multiple `TreasuryCap`s with the same witness, and in real-world application, this creates a problem of trust. If it was a human decision to support a Coin based on this implementation, they would have to make sure that:

- there is only one `TreasuryCap` for a given `T`.
- the module cannot be upgraded to issue more `TreasuryCap`s.
- the module code does not contain any backdoors to issue more `TreasuryCap`s.

However, it is not possible to check any of these conditions inside the Move code. And to prevent the need for trust, Sui introduces the OTW pattern.

## Solving the Coin Problem

To solve the case of multiple `TreasuryCap`s, we can use the OTW pattern. By defining the `COIN_OTW` type as an OTW, we can ensure that the `COIN_OTW` is used only once. The `COIN_OTW` is then used to create a new `TreasuryCap` and mint a new `Coin`.

```move

With

```move
module book::coin_otw {

    /// The OTW for the `book::coin_otw` module.
    struct COIN_OTW has drop {}

    /// Receive the instance of `COIN_OTW` as the first argument.
    fun init(otw: COIN_OTW, ctx: &mut TxContext) {
        let treasury_cap = book::simple_coin::new(COIN_OTW {}, ctx);
        transfer::public_transfer(treasury_cap, ctx.sender())
    }
}
```


 -->

<!-- ## Case Study: Coin

TODO: add a story behind TreasuryCap and Coin

-->

## 总结

OTW模式是确保类型只使用一次的好方法。大多数开发者应该了解如何定义和接收OTW，而OTW检查和强制执行主要在库和框架中需要。例如，`sui::coin`模块在`coin::create_currency`方法中需要OTW，因此强制`coin::TreasuryCap`只创建一次。

OTW是一个强大的工具，为[Publisher](./publisher)对象奠定了基础，我们将在下一节中介绍。

<!--

## Questions
- What other ways could be used to prevent multiple `TreasuryCap`s?
- Are there any other ways to use the OTW?

 -->