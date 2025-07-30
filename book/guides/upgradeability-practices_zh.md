# 可升级性实践

要讨论可升级性的最佳实践，我们需要首先了解包中可以升级的内容。可升级性的基本前提是升级不应该破坏与先前版本的公共兼容性。模块中可以在依赖包中使用的部分不应该改变其静态签名。这适用于模块 - 模块不能从包中移除，公共结构体 - 它们可以在函数签名中使用，以及公共函数 - 它们可以从其他包中调用。

```move
// 模块不能从包中移除
module book::upgradable;

// 依赖关系可以更改（如果它们未在公共签名中使用）
use std::string::String;
use sui::event; // 可以移除

// 公共结构体不能移除且不能更改
public struct Book has key {
    id: UID,
    title: String,
}

// 公共结构体不能移除且不能更改
public struct BookCreated has copy, drop {
    /* ... */
}

// 公共函数不能移除且其签名永远不能更改
// 但实现可以更改
public fun create_book(ctx: &mut TxContext): Book {
    create_book_internal(ctx)

    // 可以移除和更改
    event::emit(BookCreated {
        /* ... */
    })
}

// 包可见性函数可以移除和更改
public(package) fun create_book_package(ctx: &mut TxContext): Book {
    create_book_internal(ctx)
}

// 入口函数只要不是公共的就可以移除和更改
entry fun create_book_entry(ctx: &mut TxContext): Book {
    create_book_internal(ctx)
}

// 私有函数可以移除和更改
fun create_book_internal(ctx: &mut TxContext): Book {
    abort 0
}
```

<!--
## 使用 entry 和 friend 函数

TODO: 添加关于 entry 和 friend 函数的部分
-->

## 对象版本控制

<!-- 这种实践是基于共享状态的函数版本锁定 -->

为了弃用包的先前版本，可以对对象进行版本控制。只要对象包含版本字段，并且使用该对象的代码期望并断言特定版本，代码就可以强制迁移到新版本。通常，升级后，可以使用管理员函数来更新共享状态的版本，以便可以使用新版本的代码，而旧版本会因版本不匹配而中止。

```move
module book::versioned_state;

const EVersionMismatch: u64 = 0;

const VERSION: u8 = 1;

/// 共享状态（也可以是拥有的）
public struct SharedState has key {
    id: UID,
    version: u8,
    /* ... */
}

public fun mutate(state: &mut SharedState) {
    assert!(state.version == VERSION, EVersionMismatch);
    // ...
}
```

## 使用动态字段进行配置版本控制

<!-- 这种实践是用于对象内容/结构的版本控制 -->

在 Sui 中有一个常见模式，允许在保持相同对象签名的同时更改对象的存储配置。这是通过保持基础对象简单且有版本控制，并将实际配置对象作为动态字段添加来实现的。使用这种_锚点_模式，配置可以通过包升级进行更改，同时保持相同的基础对象签名。

```move
module book::versioned_config;

use sui::vec_map::VecMap;
use std::string::String;

/// 基础对象
public struct Config has key {
    id: UID,
    version: u16
}

/// 实际配置
public struct ConfigV1 has store {
    data: Bag,
    metadata: VecMap<String, String>
}

// ...
```

## 模块化架构

本节即将推出！

<!-- TODO: 为模块化架构添加两种模式：对象能力（SuiFrens）和见证者注册表（SuiNS） -->