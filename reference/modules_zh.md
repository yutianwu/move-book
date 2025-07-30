---
title: 'Modules | Reference'
description: ''
---

# 模块

**模块**是定义类型以及对这些类型进行操作的函数的核心程序单元。结构体类型定义 Move [存储](./abilities#key)的模式，模块函数定义与这些类型的值交互的规则。虽然模块本身也存储在存储中，但它们在 Move 程序内部不可访问。在区块链环境中，模块存储在链上，这个过程通常被称为"发布"。发布后，[`entry`](./functions#entry-modifier) 和 [`public`](./functions#visibility) 函数可以根据特定 Move 实例的规则被调用。

## 语法

模块具有以下语法：

```text
module <address>::<identifier> {
    (<use> | <type> | <function> | <constant>)*
}
```

其中 `<address>` 是指定模块包的有效[地址](./primitive-types/address)。

例如：

```move
module 0::test;

use std::debug;

const ONE: u64 = 1;

public struct Example has copy, drop { i: u64 }

public fun print(x: u64) {
    let sum = x + ONE;
    let example = Example { i: sum };
    debug::print(&sum)
}
```

## 名称

`module test_addr::test` 部分指定模块 `test` 将在[包设置](./packages)中为名称 `test_addr` 分配的数值[地址](./primitive-types/address)值下发布。

模块通常应该使用[命名地址](./primitive-types/address)声明（而不是直接使用数值）。例如：

```move
module test_addr::test;

use std::debug;
use test_addr::another_test;

public struct Example has copy, drop { a: address }

public fun print() {
    let example = Example { a: @test_addr };
    debug::print(&example)
}
```

这些命名地址通常与[包](./packages)的名称匹配。

由于命名地址仅存在于源语言级别和编译期间，命名地址将在字节码级别完全替换为其值。例如，如果我们有以下代码：

```move
fun example() {
    my_addr::m::foo(@my_addr);
}
```

并且我们将 `my_addr` 设置为 `0xC0FFEE` 进行编译，那么它在操作上等价于以下内容：

```move
fun example() {
    0xC0FFEE::m::foo(@0xC0FFEE);
}
```

虽然在源码级别这两种不同的访问是等价的，但最佳实践是始终使用命名地址而不是分配给该地址的数值。

模块名可以以 `a` 到 `z` 的小写字母或 `A` 到 `Z` 的大写字母开头。在第一个字符之后，模块名可以包含下划线 `_`、字母 `a` 到 `z`、字母 `A` 到 `Z` 或数字 `0` 到 `9`。

```move
module a::my_module {}
module a::foo_bar_42 {}
```

通常，模块名以小写字母开头。名为 `my_module` 的模块应该存储在名为 `my_module.move` 的源文件中。

## 成员

`module` 块内的所有成员可以以任何顺序出现。从根本上说，模块是[`types`](./structs)和[`functions`](./functions)的集合。[`use`](./uses) 关键字引用其他模块的成员。[`const`](./constants) 关键字定义可以在模块函数中使用的常量。

[`friend`](./friends) 语法是指定受信任模块列表的已弃用概念。该概念已被 [`public(package)`](./functions#visibility) 取代。

<!-- TODO member access rules -->