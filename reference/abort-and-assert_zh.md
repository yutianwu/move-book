---
title: '中止和断言 | 参考手册'
description: ''
---

# 中止和断言

[`return`](./functions) 和 `abort` 是两种控制流构造，用于结束执行，一个用于当前函数，一个用于整个交易。

有关 [`return` 的更多信息可以在链接的部分中找到](./functions#return-expression)

## `abort`

`abort` 是一个表达式，它要么不接受参数，要么只接受一个参数——类型为 `u64` 的**中止代码**。例如：

```move
abort
abort 42
```

`abort` 表达式会停止当前函数的执行，并回滚当前交易对状态所做的所有更改（注意这个保证必须由 Move 特定部署的适配器来维护）。没有机制可以"捕获"或以其他方式处理 `abort`。

幸运的是，在 Move 中交易是全有或全无的，意味着只有当交易成功时，对存储的任何更改才会一次性全部生效。对于 Sui，这意味着没有对象会被修改。

由于这种交易性的更改提交，在中止后无需担心撤销更改。虽然这种方法缺乏灵活性，但它非常简单和可预测。

类似于 [`return`](./functions)，`abort` 在无法满足某种条件时用于退出控制流。

在这个例子中，函数将从向量中弹出两个项目，但如果向量没有两个项目，将提前中止

<!-- {{#include ../../packages/reference/sources/abort-and-assert.move}} -->

```move
use std::vector;

fun pop_twice<T>(v: &mut vector<T>): (T, T) {
    if (v.length() < 2) abort 42;
    (v.pop_back(), v.pop_back())
}
```

在控制流构造的深层嵌套中，这会更有用。例如，这个函数检查向量中的所有数字是否都小于指定的 `bound`。否则中止

```move
use std::vector;
fun check_vec(v: &vector<u64>, bound: u64) {
    let mut i = 0;
    let n = v.length();
    while (i < n) {
        let cur = v[i];
        if (cur > bound) abort 42;
        i = i + 1;
    }
}
```

### `assert`

`assert` 是 Move 编译器提供的内置宏操作。它接受两个参数，一个类型为 `bool` 的条件和一个类型为 `u64` 的代码

```move
assert!(condition: bool, code: u64)
```

由于该操作是宏，因此必须使用 `!` 调用。这是为了表明 `assert` 的参数是按表达式调用的。换句话说，`assert` 不是普通函数，在字节码级别不存在。它在编译器内部被替换为

```move
if (condition) () else abort code
```

`assert` 比单独使用 `abort` 更常用。上面的 `abort` 示例可以使用 `assert` 重写

```move
use std::vector;
fun pop_twice<T>(v: &mut vector<T>): (T, T) {
    assert!(v.length() >= 2, 42); // 现在使用 'assert'
    (v.pop_back(), v.pop_back())
}
```

和

```move
use std::vector;
fun check_vec(v: &vector<u64>, bound: u64) {
    let mut i = 0;
    let n = v.length();
    while (i < n) {
        let cur = v[i];
        assert!(cur <= bound, 42); // 现在使用 'assert'
        i = i + 1;
    }
}
```

注意，因为该操作被替换为这个 `if-else`，`code` 的参数并不总是被计算。例如：

```move
assert!(true, 1 / 0)
```

不会导致算术错误，它等价于

```move
if (true) () else abort (1 / 0)
```

所以算术表达式永远不会被计算！

### Move VM 中的中止代码

使用 `abort` 时，重要的是要理解 VM 如何使用 `u64` 代码。

通常，在成功执行后，Move VM 和特定部署的适配器确定对存储所做的更改。

如果到达 `abort`，VM 将指示错误。该错误中将包含两条信息：

- 产生中止的模块（包/地址值和模块名称）
- 中止代码。

例如

```move
module 0x2::example {
    public fun aborts() {
        abort 42
    }
}

module 0x3::invoker {
    public fun always_aborts() {
        0x2::example::aborts()
    }
}
```

如果一个交易，比如上面的函数 `always_aborts`，调用 `0x2::example::aborts`，VM 会产生一个错误，指示模块 `0x2::example` 和代码 `42`。

这对于在模块中将多个中止分组在一起很有用。

在这个例子中，模块有两个单独的错误代码在多个函数中使用

```move
module 0::example;

use std::vector;

const EEmptyVector: u64 = 0;
const EIndexOutOfBounds: u64 = 1;

// 将 i 移动到 j，将 j 移动到 k，将 k 移动到 i
public fun rotate_three<T>(v: &mut vector<T>, i: u64, j: u64, k: u64) {
    let n = v.length();
    assert!(n > 0, EEmptyVector);
    assert!(i < n, EIndexOutOfBounds);
    assert!(j < n, EIndexOutOfBounds);
    assert!(k < n, EIndexOutOfBounds);

    v.swap(i, k);
    v.swap(j, k);
}

public fun remove_twice<T>(v: &mut vector<T>, i: u64, j: u64): (T, T) {
    let n = v.length();
    assert!(n > 0, EEmptyVector);
    assert!(i < n, EIndexOutOfBounds);
    assert!(j < n, EIndexOutOfBounds);
    assert!(i > j, EIndexOutOfBounds);

    (v.remove(i), v.remove(j))
}
```

## `abort` 的类型

`abort i` 表达式可以有任何类型！这是因为两种构造都会从正常控制流中中断，所以它们永远不需要计算为该类型的值。

以下不是有用的，但它们会通过类型检查

```move
let y: address = abort 0;
```

这种行为在某些分支产生值但不是所有分支都产生值的分支指令情况下很有用。例如：

```move
let b =
    if (x == 0) false
    else if (x == 1) true
    else abort 42;
//       ^^^^^^^^ `abort 42` 的类型是 `bool`
```