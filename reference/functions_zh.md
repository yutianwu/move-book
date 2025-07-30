---
title: 'Functions | Reference'
description: ''
---

# 函数

函数在模块内部声明，定义模块的逻辑和行为。函数可以被重用，既可以从其他函数调用，也可以作为执行的入口点。

## 声明

函数使用 `fun` 关键字声明，后面跟着函数名、类型参数、参数、返回类型，最后是函数体。

```text
<visibility>? <entry>? <macro>? fun <identifier><[type_parameters: constraint],*>([identifier: type],*): <return_type> <function_body>
```

例如

```move
fun foo<T1, T2>(x: u64, y: T1, z: T2): (T2, T1, u64) { (z, y, x) }
```

### 可见性

模块函数默认只能在同一模块内调用。这些内部（有时称为私有）函数不能从其他模块调用或作为入口点。

```move
module a::m {
    fun foo(): u64 { 0 }
    fun calls_foo(): u64 { foo() } // 有效
}

module b::other {
    fun calls_m_foo(): u64 {
        a::m::foo() // 错误！
//      ^^^^^^^^^^^ 'foo' 是 'a::m' 内部的
    }
}
```

要允许从其他模块访问，函数必须声明为 `public` 或 `public(package)`。与可见性相关的是，[`entry`](#entry-modifier) 函数可以作为执行的入口点调用。

#### `public` 可见性

`public` 函数可以被定义在*任何*模块中的*任何*函数调用。如下例所示，`public` 函数可以被以下方式调用：

- 同一模块中定义的其他函数，
- 其他模块中定义的函数，或
- 作为执行的入口点。

```move
module a::m {
    public fun foo(): u64 { 0 }
    fun calls_foo(): u64 { foo() } // 有效
}

module b::other {
    fun calls_m_foo(): u64 {
        a::m::foo() // 有效
    }
}
```

有关执行入口点的更多详细信息，请参见[下面的部分](#entry-modifier)。

#### `public(package)` 可见性

`public(package)` 可见性修饰符是 `public` 修饰符的更受限形式，以便更好地控制函数的使用位置。`public(package)` 函数可以被以下方式调用：

- 同一模块中定义的其他函数，或
- 同一包（同一地址）中定义的其他函数

```move
module a::m {
    public(package) fun foo(): u64 { 0 }
    fun calls_foo(): u64 { foo() } // 有效
}

module a::n {
    fun calls_m_foo(): u64 {
        a::m::foo() // 有效，也在 `a` 中
    }
}

module b::other {
    fun calls_m_foo(): u64 {
        a::m::foo() // 错误！
//      ^^^^^^^^^^^ 'foo' 只能从 `a` 中的模块调用
    }
}
```

#### 已弃用的 `public(friend)` 可见性

在添加 `public(package)` 之前，`public(friend)` 用于允许同一包中的函数有限的公共访问，但允许的模块列表必须由被调用者的模块显式列举。更多详细信息请参见[朋友](./friends)。

### `entry` 修饰符

除了 `public` 函数之外，您可能在模块中有一些函数希望用作执行的入口点。`entry` 修饰符旨在允许模块函数启动执行，而无需向其他模块公开功能。

本质上，`public` 和 `entry` 函数的组合定义了模块的"主"函数，它们指定了 Move 程序可以开始执行的位置。

不过要记住，`entry` 函数*仍然*可以被其他 Move 函数调用。因此，虽然它们*可以*作为 Move 程序的起点，但并不限于这种情况。

例如：

```move
module a::m {
    entry fun foo(): u64 { 0 }
    fun calls_foo(): u64 { foo() } // 有效！
}

module a::n {
    fun calls_m_foo(): u64 {
        a::m::foo() // 错误！
//      ^^^^^^^^^^^ 'foo' 是 'a::m' 内部的
    }
}
```

`entry` 函数可能对其参数和返回类型有限制。不过，这些限制特定于 Move 的每个单独部署。

[关于 Sui 上 `entry` 函数的文档可以在这里找到。](https://docs.sui.io/concepts/sui-move-concepts/entry-functions)

为了便于测试，`entry` 函数可以从[`#[test]` 和 `#[test_only]`](./unit-testing) 上下文中调用。

```move
module a::m {
    entry fun foo(): u64 { 0 }
}
module a::m_test {
    #[test]
    fun my_test(): u64 { a::m::foo() } // 有效！
    #[test_only]
    fun my_test_helper(): u64 { a::m::foo() } // 有效！
}
```

### `macro` 修饰符

与普通函数不同，`macro` 函数在运行时不存在。相反，这些函数在编译期间在每个调用点内联替换。这些 `macro` 函数利用此编译过程来提供超出标准函数的功能，例如接受高阶 *lambda* 样式函数作为参数。这些 lambda 参数也在编译期间展开，允许您将函数体的部分作为参数传递给宏。例如，考虑以下简单的循环宏，其中循环体作为 lambda 提供：

```move
macro fun n_times($n: u64, $body: |u64| -> ()) {
    let n = $n;
    let mut i = 0;
    while (i < n) {
        $body(i);
        i = i + 1;
    }
}

fun example() {
    let mut sum = 0;
    n_times!(10, |x| sum = sum + x );
}
```

更多信息请参见[宏](./functions/macros)章节。

### 名称

函数名可以以字母 `a` 到 `z` 开头。在第一个字符之后，函数名可以包含下划线 `_`、字母 `a` 到 `z`、字母 `A` 到 `Z` 或数字 `0` 到 `9`。

```move
fun fOO() {}
fun bar_42() {}
fun bAZ_19() {}
```

### 类型参数

在名称之后，函数可以有类型参数

```move
fun id<T>(x: T): T { x }
fun example<T1: copy, T2>(x: T1, y: T2): (T1, T1, T2) { (copy x, x, y) }
```

更多详细信息，请参见 [Move 泛型](./generics)。

### 参数

函数参数使用局部变量名后跟类型注释声明

```move
fun add(x: u64, y: u64): u64 { x + y }
```

我们将其读作 `x` 具有类型 `u64`

函数根本不必有任何参数。

```move
fun useless() { }
```

这对于创建新的或空的数据结构的函数非常常见

```move
module a::example;

public struct Counter { count: u64 }

fun new_counter(): Counter {
    Counter { count: 0 }
}
```

### 返回类型

在参数之后，函数指定其返回类型。

```move
fun zero(): u64 { 0 }
```

这里 `: u64` 表示函数的返回类型是 `u64`。

使用[元组](./primitive-types/tuples)，函数可以返回多个值：

```move
fun one_two_three(): (u64, u64, u64) { (0, 1, 2) }
```

如果未指定返回类型，函数具有隐式返回类型 unit `()`。这些函数是等价的：

```move
fun just_unit(): () { () }
fun just_unit() { () }
fun just_unit() { }
```

如[元组部分](./primitive-types/tuples)中所述，这些元组"值"在运行时不作为值存在。这意味着返回 unit `()` 的函数在执行期间不返回任何值。

### 函数体

函数体是一个表达式块。函数的返回值是序列中的最后一个值

```move
fun example(): u64 {
    let mut x = 0;
    x = x + 1;
    x // 返回 'x'
}
```

更多信息请参见[下面关于返回的部分](#returning-values)

有关表达式块的更多信息，请参见 [Move 变量](./variables)。

### 原生函数

一些函数没有指定函数体，而是由 VM 提供函数体。这些函数标记为 `native`。

在不修改 VM 源代码的情况下，程序员无法添加新的原生函数。此外，`native` 函数的意图是用于标准库代码或给定 Move 环境所需的功能。

您可能看到的大多数 `native` 函数都在标准库代码中，例如 `vector`

```move
module std::vector {
    native public fun length<Element>(v: &vector<Element>): u64;
    ...
}
```

## 调用

调用函数时，名称可以通过别名或完全限定指定

```move
module a::example {
    public fun zero(): u64 { 0 }
}

module b::other {
    use a::example::{Self, zero};
    fun call_zero() {
        // 使用上面的 `use`，所有这些调用都是等价的
        a::example::zero();
        example::zero();
        zero();
    }
}
```

调用函数时，必须为每个参数提供一个实参。

```move
module a::example {
    public fun takes_none(): u64 { 0 }
    public fun takes_one(x: u64): u64 { x }
    public fun takes_two(x: u64, y: u64): u64 { x + y }
    public fun takes_three(x: u64, y: u64, z: u64): u64 { x + y + z }
}

module b::other {
    fun call_all() {
        a::example::takes_none();
        a::example::takes_one(0);
        a::example::takes_two(0, 1);
        a::example::takes_three(0, 1, 2);
    }
}
```

类型参数可以被指定或推断。两种调用都是等价的。

```move
module a::example {
    public fun id<T>(x: T): T { x }
}

module b::other {
    fun call_all() {
        a::example::id(0);
        a::example::id<u64>(0);
    }
}
```

更多详细信息，请参见 [Move 泛型](./generics)。

## 返回值

函数的结果，即其"返回值"，是其函数体的最终值。例如

```move
fun add(x: u64, y: u64): u64 {
    x + y
}
```

这里的返回值是 `x + y` 的结果。

[如上所述](#function-body)，函数体是一个[表达式块](./variables)。表达式块可以排列各种语句，块中的最终表达式将是该块的值

```move
fun double_and_add(x: u64, y: u64): u64 {
    let double_x = x * 2;
    let double_y = y * 2;
    double_x + double_y
}
```

这里的返回值是 `double_x + double_y` 的结果

### `return` 表达式

函数隐式返回其函数体评估的值。然而，函数也可以使用显式的 `return` 表达式：

```move
fun f1(): u64 { return 0 }
fun f2(): u64 { 0 }
```

这两个函数是等价的。在这个稍微复杂的例子中，函数减去两个 `u64` 值，但如果第二个值太大，会提前返回 `0`：

```move
fun safe_sub(x: u64, y: u64): u64 {
    if (y > x) return 0;
    x - y
}
```

注意这个函数的主体也可以写成 `if (y > x) 0 else x - y`。

然而，`return` 真正发挥作用是在其他控制流结构的深处退出。在这个例子中，函数遍历向量以找到给定值的索引：

```move
use std::vector;
use std::option::{Self, Option};

fun index_of<T>(v: &vector<T>, target: &T): Option<u64> {
    let mut i = 0;
    let n = vector::length(v);
    while (i < n) {
        if (vector::borrow(v, i) == target) return option::some(i);
        i = i + 1
    };

    option::none()
}
```

使用不带参数的 `return` 是 `return ()` 的简写。也就是说，以下两个函数是等价的：

```move
fun foo() { return }
fun foo() { return () }
```