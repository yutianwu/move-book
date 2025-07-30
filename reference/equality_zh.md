---
title: '相等性 | 参考手册'
description: ''
---

# 相等性

Move 支持两种相等操作 `==` 和 `!=`

## 操作

| 语法   | 操作   | 描述                                                                 |
| ------ | ----- | ------------------------------------------------------------------- |
| `==`   | 相等   | 如果两个操作数具有相同的值，返回 `true`，否则返回 `false`                    |
| `!=`   | 不等   | 如果两个操作数具有不同的值，返回 `true`，否则返回 `false`                    |

### 类型

相等（`==`）和不等（`!=`）操作只有在两个操作数是相同类型时才有效

```move
0 == 0; // `true`
1u128 == 2u128; // `false`
b"hello" != x"00"; // `true`
```

相等性和不等性也适用于 _所有_ 用户定义的类型！

```move
module 0::example;

public struct S has copy, drop { f: u64, s: vector<u8> }

fun always_true(): bool {
    let s = S { f: 0, s: b"" };
    s == s
}

fun always_false(): bool {
    let s = S { f: 0, s: b"" };
    s != s
}
```

如果操作数具有不同的类型，会有类型检查错误

```move
1u8 == 1u128; // 错误!
//     ^^^^^ 期望类型为 'u8' 的参数
b"" != 0; // 错误!
//     ^ 期望类型为 'vector<u8>' 的参数
```

### 引用的类型

当比较[引用](./primitive-types/references)时，引用的类型（不可变或可变）并不重要。这意味着你可以比较不可变引用 `&` 和同一底层类型的可变引用 `&mut`。

```move
let i = &0;
let m = &mut 1;

i == m; // `false`
m == i; // `false`
m == m; // `true`
i == i; // `true`
```

上面的代码等价于在需要时对每个可变引用应用显式冻结

```move
let i = &0;
let m = &mut 1;

i == freeze(m); // `false`
freeze(m) == i; // `false`
m == m; // `true`
i == i; // `true`
```

但是，底层类型必须是相同类型

```move
let i = &0;
let s = &b"";

i == s; // 错误!
//   ^ 期望类型为 '&u64' 的参数
```

### 自动借用

从 Move 2024 版本开始，如果其中一个操作数是引用而另一个不是，`==` 和 `!=` 操作符会自动借用它们的操作数。这意味着以下代码可以正常工作而不会有任何错误：

```move
let r = &0;

// 在所有情况下，`0` 都会自动被借用为 `&0`
r == 0; // `true`
0 == r; // `true`
r != 0; // `false`
0 != r; // `false`
```

这种自动借用总是不可变借用。

## 限制

`==` 和 `!=` 在比较时都会消耗值。因此，类型系统强制要求类型必须具有 [`drop`](./abilities) 能力。回想一下，没有 [`drop` 能力](./abilities)，所有权必须在函数结束时转移，这样的值只能在其声明模块内显式销毁。如果这些直接与相等 `==` 或不等 `!=` 一起使用，值将被销毁，这将打破 [`drop` 能力](./abilities)的安全保证！

```move
module 0::example;

public struct Coin has store { value: u64 }
fun invalid(c1: Coin, c2: Coin) {
    c1 == c2 // 错误!
//  ^^    ^^ 这些资产将被销毁!
}
```

但是，程序员总是可以先借用值而不是直接比较值，引用类型具有 [`drop` 能力](./abilities)。例如

```move
module 0::example;

public struct Coin has store { value: u64 }
fun swap_if_equal(c1: Coin, c2: Coin): (Coin, Coin) {
    let are_equal = &c1 == c2; // 有效，注意 `c2` 被自动借用
    if (are_equal) (c2, c1) else (c1, c2)
}
```

## 避免额外复制

虽然程序员 _可以_ 比较任何类型具有 [`drop`](./abilities) 能力的值，但程序员通常应该通过引用进行比较以避免昂贵的复制。

```move
let v1: vector<u8> = function_that_returns_vector();
let v2: vector<u8> = function_that_returns_vector();
assert!(copy v1 == copy v2, 42);
//      ^^^^       ^^^^
use_two_vectors(v1, v2);

let s1: Foo = function_that_returns_large_struct();
let s2: Foo = function_that_returns_large_struct();
assert!(copy s1 == copy s2, 42);
//      ^^^^       ^^^^
use_two_foos(s1, s2);
```

这段代码是完全可接受的（假设 `Foo` 具有 [`drop`](./abilities) 能力），只是不够高效。高亮的复制可以被移除并替换为借用

```move
let v1: vector<u8> = function_that_returns_vector();
let v2: vector<u8> = function_that_returns_vector();
assert!(&v1 == &v2, 42);
//      ^      ^
use_two_vectors(v1, v2);

let s1: Foo = function_that_returns_large_struct();
let s2: Foo = function_that_returns_large_struct();
assert!(&s1 == &s2, 42);
//      ^      ^
use_two_foos(s1, s2);
```

`==` 本身的效率保持不变，但是 `copy` 被移除，因此程序更高效。