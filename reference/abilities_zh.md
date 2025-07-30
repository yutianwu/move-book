---
title: '能力 | 参考手册'
description: ''
---

# 能力

能力是 Move 中的一种类型系统特性，用于控制给定类型的值可以执行哪些操作。这个系统对值的"线性"类型行为提供细粒度控制，以及值是否以及如何在存储中使用（由 Move 的具体部署定义，例如区块链的存储概念）。这是通过限制对某些字节码指令的访问来实现的，因此一个值要与字节码指令一起使用，它必须具有所需的能力（如果需要的话——并非每个指令都受能力限制）。

对于 Sui，`key` 用于表示一个[对象](./abilities/object)。对象是存储的基本单位，每个对象都有唯一的 32 字节 ID。`store` 则用于指示哪些数据可以存储在对象内部，同时也用于指示哪些类型可以在其定义模块之外进行转移。

<!-- TODO future section on detailed walk through maybe. We have some examples at the end but it might be helpful to explain why we have precisely this set of abilities

If you are already somewhat familiar with abilities from writing Move programs, but are still confused as to what is going on, it might be helpful to skip to the [motivating walkthrough](#motivating-walkthrough) section to get an idea of what the system is setup in the way that it is. -->

## 四种能力

四种能力是：

- [`copy`](#copy)
  - 允许复制具有此能力的类型的值。
- [`drop`](#drop)
  - 允许弹出/丢弃具有此能力的类型的值。
- [`store`](#store)
  - 允许具有此能力的类型的值存在于存储中的值内部。
  - 对于 Sui，`store` 控制哪些数据可以存储在[对象](./abilities/object)内部。
    `store` 还控制哪些类型可以在其定义模块之外进行转移。
- [`key`](#key)
  - 允许类型作为存储的"键"。表面上这意味着该值可以是存储中的顶层值；换句话说，它不需要包含在另一个值中就可以存在于存储中。
  - 对于 Sui，`key` 用于表示一个[对象](./abilities/object)。

### `copy`

`copy` 能力允许复制具有该能力的类型的值。它限制了使用 [`copy`](./variables#move-and-copy) 操作符从本地变量复制值的能力，以及通过引用使用[解引用 `*e`](./primitive-types/references#reading-and-writing-through-references) 复制值的能力。

如果一个值具有 `copy` 能力，那么该值内部包含的所有值都具有 `copy` 能力。

### `drop`

`drop` 能力允许丢弃具有该能力的类型的值。所谓丢弃，是指该值不被转移并在 Move 程序执行时被有效销毁。因此，这个能力限制了在多个位置忽略值的能力，包括：

- 不在本地变量或参数中使用该值
- 不在[通过 `;` 的序列](./variables#expression-blocks)中使用该值
- 在[赋值](./variables#assignments)中覆盖变量中的值
- 在[写入 `*e1 = e2`](./primitive-types/references#reading-and-writing-through-references) 时通过引用覆盖值。

如果一个值具有 `drop` 能力，那么该值内部包含的所有值都具有 `drop` 能力。

### `store`

`store` 能力允许具有此能力的类型的值存在于存储中的值内部，*但*不一定作为存储中的顶层值。这是唯一不直接限制操作的能力。相反，它在与 `key` 结合使用时限制存储中的存在。

如果一个值具有 `store` 能力，那么该值内部包含的所有值都具有 `store` 能力。

对于 Sui，`store` 承担双重职责。它控制哪些值可以出现在[对象](/storage/store-ability)内部，以及哪些对象可以在其定义模块之外进行[转移](./abilities/object#transfer-rules)。

### `key`

`key` 能力允许类型作为 Move 部署定义的存储操作的键。虽然它对每个 Move 实例都是特定的，但它限制所有存储操作，因此为了让类型与存储原语一起使用，该类型必须具有 `key` 能力。

如果一个值具有 `key` 能力，那么该值内部包含的所有值都具有 `store` 能力。这是唯一具有这种不对称性的能力。

对于 Sui，`key` 用于表示一个[对象](./abilities/object)。

## 内置类型

所有原始的内置类型都具有 `copy`、`drop` 和 `store` 能力。

- `bool`、`u8`、`u16`、`u32`、`u64`、`u128`、`u256` 和 `address` 都具有 `copy`、`drop` 和 `store` 能力。
- `vector<T>` 可能具有 `copy`、`drop` 和 `store` 能力，这取决于 `T` 的能力。
  - 更多详情请参见[条件能力和泛型类型](#conditional-abilities-and-generic-types)。
- 不可变引用 `&` 和可变引用 `&mut` 都具有 `copy` 和 `drop` 能力。
  - 这指的是复制和丢弃引用本身，而不是它们所引用的内容。
  - 引用不能出现在全局存储中，因此它们没有 `store` 能力。

注意，原始类型都没有 `key` 能力，这意味着它们都不能直接与存储操作一起使用。

## 注解结构体和枚举

要声明 `struct` 或 `enum` 具有能力，需要在数据类型名称之后、字段/变体之前或之后使用 `has <ability>` 声明。例如：

```move file=packages/reference/sources/abilities.move anchor=annotating_datatypes

```

在这种情况下：`Ignorable*` 具有 `drop` 能力。`Pair*` 和 `MyVec*` 都具有 `copy`、`drop` 和 `store` 能力。

所有这些能力对这些受限操作都有强大的保证。只有当值具有该能力时，才能对该值执行操作；即使该值深度嵌套在其他集合中！

因此：在声明结构体的能力时，会对字段提出某些要求。所有字段都必须满足这些约束。这些规则是必要的，以便结构体满足上述能力的可达性规则。如果结构体声明具有能力...

- `copy`，所有字段必须具有 `copy`。
- `drop`，所有字段必须具有 `drop`。
- `store`，所有字段必须具有 `store`。
- `key`，所有字段必须具有 `store`。
  - `key` 是目前唯一不需要自身的能力。

枚举可以具有除 `key` 之外的任何这些能力，枚举不能具有 `key` 能力，因为它们不能作为存储中的顶层值（对象）。但是，相同的规则适用于枚举变体的字段，就像适用于结构体字段一样。特别是，如果枚举声明具有能力...

- `copy`，所有变体的所有字段必须具有 `copy`。
- `drop`，所有变体的所有字段必须具有 `drop`。
- `store`，所有变体的所有字段必须具有 `store`。
- `key`，如前所述，枚举不允许使用。

例如：

```move
// 没有任何能力的结构体
public struct NoAbilities {}

public struct WantsCopy has copy {
    f: NoAbilities, // 错误：'NoAbilities' 没有 'copy' 能力
}

public enum WantsCopyEnum has copy {
    Variant1
    Variant2(NoAbilities), // 错误：'NoAbilities' 没有 'copy' 能力
}
```

类似地：

```move
// 没有任何能力的结构体
public struct NoAbilities {}

public struct MyData has key {
    f: NoAbilities, // 错误：'NoAbilities' 没有 'store' 能力
}

public struct MyDataEnum has store {
    Variant1,
    Variant2(NoAbilities), // 错误：'NoAbilities' 没有 'store' 能力
}
```

## 条件能力和泛型类型

当在泛型类型上注解能力时，并不保证该类型的所有实例都具有该能力。考虑这个结构体声明：

<!-- file=packages/reference/sources/abilities.move anchor=conditional_abilities -->

```move
public struct Cup<T> has copy, drop, store, key { item: T }
```

如果 `Cup` 可以容纳任何类型，无论其能力如何，那将非常有用。类型系统可以*看到*类型参数，因此如果它*看到*一个会违反该能力保证的类型参数，它应该能够从 `Cup` 中移除能力。

这种行为初看起来可能有点令人困惑，但如果我们考虑集合类型，它可能更容易理解。我们可以考虑内置类型 `vector` 具有以下类型声明：

```move
vector<T> has copy, drop, store;
```

我们希望 `vector` 与任何类型一起工作。我们不希望为不同能力有单独的 `vector` 类型。那么我们希望的规则是什么？恰好与我们在上面字段规则中想要的相同。因此，只有当内部元素可以被复制时，复制 `vector` 值才是安全的。只有当内部元素可以被忽略/丢弃时，忽略 `vector` 值才是安全的。而且，只有当内部元素可以在存储中时，将 `vector` 放入存储中才是安全的。

为了具有这种额外的表达能力，类型可能不具有它被声明的所有能力，这取决于该类型的实例化；相反，类型将具有的能力取决于其声明**和**其类型参数。对于任何类型，类型参数被悲观地假设在结构体内部使用，因此只有当类型参数满足上述字段要求时，才会授予能力。以上面的 `Cup` 为例：

- `Cup` 只有当 `T` 具有 `copy` 时才具有 `copy` 能力。
- 只有当 `T` 具有 `drop` 时才具有 `drop` 能力。
- 只有当 `T` 具有 `store` 时才具有 `store` 能力。
- 只有当 `T` 具有 `store` 时才具有 `key` 能力。

以下是每种能力的条件系统示例：

### 示例：条件 `copy`

```move
public struct NoAbilities {}
public struct S has copy, drop { f: bool }
public struct Cup<T> has copy, drop, store { item: T }

fun example(c_x: Cup<u64>, c_s: Cup<S>) {
    // 有效，'Cup<u64>' 具有 'copy' 能力，因为 'u64' 具有 'copy' 能力
    let c_x2 = copy c_x;
    // 有效，'Cup<S>' 具有 'copy' 能力，因为 'S' 具有 'copy' 能力
    let c_s2 = copy c_s;
}

fun invalid(c_account: Cup<signer>, c_n: Cup<NoAbilities>) {
    // 无效，'Cup<signer>' 没有 'copy' 能力。
    // 尽管 'Cup' 被声明具有 copy 能力，但实例没有 'copy' 能力
    // 因为 'signer' 没有 'copy' 能力
    let c_account2 = copy c_account;
    // 无效，'Cup<NoAbilities>' 没有 'copy' 能力
    // 因为 'NoAbilities' 没有 'copy' 能力
    let c_n2 = copy c_n;
}
```

### 示例：条件 `drop`

```move
public struct NoAbilities {}
public struct S has copy, drop { f: bool }
public struct Cup<T> has copy, drop, store { item: T }

fun unused() {
    Cup<bool> { item: true }; // 有效，'Cup<bool>' 具有 'drop' 能力
    Cup<S> { item: S { f: false }}; // 有效，'Cup<S>' 具有 'drop' 能力
}

fun left_in_local(c_account: Cup<signer>): u64 {
    let c_b = Cup<bool> { item: true };
    let c_s = Cup<S> { item: S { f: false }};
    // 有效返回：'c_account'、'c_b' 和 'c_s' 有值
    // 但 'Cup<signer>'、'Cup<bool>' 和 'Cup<S>' 具有 'drop' 能力
    0
}

fun invalid_unused() {
    // 无效，不能忽略 'Cup<NoAbilities>'，因为它没有 'drop' 能力。
    // 尽管 'Cup' 被声明具有 'drop' 能力，但实例没有 'drop' 能力
    // 因为 'NoAbilities' 没有 'drop' 能力
    Cup<NoAbilities> { item: NoAbilities {} };
}

fun invalid_left_in_local(): u64 {
    let n = Cup<NoAbilities> { item: NoAbilities {} };
    // 无效返回：'c_n' 有值
    // 而 'Cup<NoAbilities>' 没有 'drop' 能力
    0
}
```

### 示例：条件 `store`

```move
public struct Cup<T> has copy, drop, store { item: T }

// 'MyInnerData' 声明具有 'store' 能力，所以所有字段都需要 'store' 能力
struct MyInnerData has store {
    yes: Cup<u64>, // 有效，'Cup<u64>' 具有 'store' 能力
    // no: Cup<signer>, 无效，'Cup<signer>' 没有 'store' 能力
}

// 'MyData' 声明具有 'key' 能力，所以所有字段都需要 'store' 能力
struct MyData has key {
    yes: Cup<u64>, // 有效，'Cup<u64>' 具有 'store' 能力
    inner: Cup<MyInnerData>, // 有效，'Cup<MyInnerData>' 具有 'store' 能力
    // no: Cup<signer>, 无效，'Cup<signer>' 没有 'store' 能力
}
```

### 示例：条件 `key`

```move
public struct NoAbilities {}
public struct MyData<T> has key { f: T }

fun valid(addr: address) acquires MyData {
    // 有效，'MyData<u64>' 具有 'key' 能力
    transfer(addr, MyData<u64> { f: 0 });
}

fun invalid(addr: address) {
   // 无效，'MyData<NoAbilities>' 没有 'key' 能力
   transfer(addr, MyData<NoAbilities> { f: NoAbilities {} })
   // 无效，'MyData<NoAbilities>' 没有 'key' 能力
   borrow<NoAbilities>(addr);
   // 无效，'MyData<NoAbilities>' 没有 'key' 能力
   borrow_mut<NoAbilities>(addr);
}

// 模拟存储操作
native public fun transfer<T: key>(addr: address, value: T);
```