---
title: '引用 | 参考'
description: ''
---

# 引用

Move有两种引用类型：不可变`&`和可变`&mut`。不可变引用是只读的，不能修改底层值（或其任何字段）。可变引用允许通过该引用进行写入修改。Move的类型系统强制执行所有权规则，防止引用错误。

## 引用操作符

Move提供用于创建和扩展引用以及将可变引用转换为不可变引用的操作符。在这里和其他地方，我们使用符号`e: T`表示"表达式`e`具有类型`T`"。

| 语法      | 类型                                                  | 描述                                                    |
| ----------- | ----------------------------------------------------- | -------------------------------------------------------------- |
| `&e`        | `&T`，其中`e: T`且`T`是非引用类型     | 创建对`e`的不可变引用                           |
| `&mut e`    | `&mut T`，其中`e: T`且`T`是非引用类型 | 创建对`e`的可变引用。                             |
| `&e.f`      | `&T`，其中`e.f: T`                                   | 创建对结构体`e`的字段`f`的不可变引用。      |
| `&mut e.f`  | `&mut T`，其中`e.f: T`                               | 创建对结构体`e`的字段`f`的可变引用。          |
| `freeze(e)` | `&T`，其中`e: &mut T`                                | 将可变引用`e`转换为不可变引用。 |

`&e.f`和`&mut e.f`操作符都可以用于创建对结构体的新引用或扩展现有引用：

```move
let s = S { f: 10 };
let f_ref1: &u64 = &s.f; // 可行
let s_ref: &S = &s;
let f_ref2: &u64 = &s_ref.f // 也可行
```

只要两个结构体在同一个模块中，具有多个字段的引用表达式就可以工作：

```move
public struct A { b: B }
public struct B { c : u64 }
fun f(a: &A): &u64 {
    &a.b.c
}
```

最后，注意不允许对引用的引用：

```move
let x = 7;
let y: &u64 = &x;
// highlight-error
let z: &&u64 = &y; // 错误！不会编译
```

## 通过引用读取和写入

可变和不可变引用都可以被读取以产生引用值的副本。

只有可变引用可以被写入。写入`*x = v`丢弃先前存储在`x`中的值并用`v`更新它。

两种操作都使用类似C的`*`语法。但是，注意读取是表达式，而写入是必须出现在等号左侧的变更。

| 语法     | 类型                                | 描述                         |
| ---------- | ----------------------------------- | ----------------------------------- |
| `*e`       | `T`，其中`e`是`&T`或`&mut T`   | 读取`e`指向的值    |
| `*e1 = e2` | `()`，其中`e1: &mut T`且`e2: T` | 用`e2`更新`e1`中的值。 |

为了读取引用，底层类型必须具有[`copy`能力](../abilities_zh)，因为读取引用会创建值的新副本。此规则防止资产的复制：

```move
fun copy_coin_via_ref_bad(c: Coin) {
    let c_ref = &c;
    // highlight-error
    let counterfeit: Coin = *c_ref; // 不允许！
    pay(c);
    pay(counterfeit);
}
```

相反：为了写入引用，底层类型必须具有[`drop`能力](../abilities_zh)，因为写入引用会丢弃（或"删除"）旧值。此规则防止资源值的销毁：

```move
fun destroy_coin_via_ref_bad(mut ten_coins: Coin, c: Coin) {
    let ref = &mut ten_coins;
    // highlight-error
    *ref = c; // 错误！不允许——会销毁10个币！
}
```

## `freeze`推断

可变引用可以在期望不可变引用的上下文中使用：

```move
let mut x = 7;
let y: &u64 = &mut x;
```

这是可行的，因为在底层，编译器在需要的地方插入`freeze`指令。以下是`freeze`推断实际运行的更多示例：

```move
fun takes_immut_returns_immut(x: &u64): &u64 { x }

// 返回值上的freeze推断
fun takes_mut_returns_immut(x: &mut u64): &u64 { x }

fun expression_examples() {
    let mut x = 0;
    let mut y = 0;
    takes_immut_returns_immut(&x); // 无推断
    takes_immut_returns_immut(&mut x); // 推断freeze(&mut x)
    takes_mut_returns_immut(&mut x); // 无推断

    assert!(&x == &mut y, 42); // 推断freeze(&mut y)
}

fun assignment_examples() {
    let x = 0;
    let y = 0;
    let imm_ref: &u64 = &x;

    imm_ref = &x; // 无推断
    imm_ref = &mut y; // 推断freeze(&mut y)
}
```

### 子类型

通过这种`freeze`推断，Move类型检查器可以将`&mut T`视为`&T`的子类型。如上所示，这意味着在使用`&T`值的任何表达式的任何地方，也可以使用`&mut T`值。这个术语在错误消息中用于简洁地指示在提供`&T`的地方需要`&mut T`。例如

```move
module a::example {
    fun read_and_assign(store: &mut u64, new_value: &u64) {
        *store = *new_value
    }

    fun subtype_examples() {
        let mut x: &u64 = &0;
        let mut y: &mut u64 = &mut 1;

        x = &mut 1; // 有效
        // highlight-error
        y = &2; // 错误！无效！

        read_and_assign(y, x); // 有效
        // highlight-error
        read_and_assign(x, y); // 错误！无效！
    }
}
```

将产生以下错误消息

```text
error:

    ┌── example.move:11:9 ───
    │
 12 │         y = &2; // invalid!
    │         ^ Invalid assignment to local 'y'
    ·
 12 │         y = &2; // invalid!
    │             -- The type: '&{integer}'
    ·
  9 │         let mut y: &mut u64 = &mut 1;
    │                    -------- Is not a subtype of: '&mut u64'
    │

error:

    ┌── example.move:14:9 ───
    │
 15 │         read_and_assign(x, y); // invalid!
    │         ^^^^^^^^^^^^^^^^^^^^^ Invalid call of 'a::example::read_and_assign'. Invalid argument for parameter 'store'
    ·
  8 │         let mut x: &u64 = &0;
    │                    ---- The type: '&u64'
    ·
  3 │     fun read_and_assign(store: &mut u64, new_value: &u64) {
    │                                -------- Is not a subtype of: '&mut u64'
    │
```

目前唯一其他具有子类型的类型是[元组](./tuples_zh)

## 所有权

可变和不可变引用总是可以被复制和扩展，_即使存在相同引用的现有副本或扩展_：

```move
fun reference_copies(s: &mut S) {
  let s_copy1 = s; // 可以
  let s_extension = &mut s.f; // 也可以
  let s_copy2 = s; // 仍然可以
  ...
}
```

这对于熟悉Rust所有权系统的程序员来说可能令人惊讶，因为Rust会拒绝上面的代码。Move的类型系统在处理[复制](./../variables_zh#move-and-copy)方面更宽松，但在确保写入前可变引用的唯一所有权方面同样严格。

### 引用不能被存储

引用和元组是_唯一_不能作为结构体字段值存储的类型，这也意味着它们不能存在于存储或[对象](./../abilities/object_zh)中。程序执行期间创建的所有引用在Move程序终止时将被销毁；它们完全是短暂的。这也适用于所有没有`store`能力的类型：任何非`store`类型的值必须在程序终止前被销毁。[能力](./../abilities_zh)，但请注意，引用和元组通过从一开始就不允许在结构体中而更进一步。

这是Move和Rust之间的另一个区别，Rust允许引用存储在结构体内部。

可以想象一个更复杂、更具表现力的类型系统将允许引用存储在结构体中。我们可以允许在没有`store`[能力](./../abilities_zh)的结构体内部的引用，但核心困难是Move有一个相当复杂的跟踪静态引用安全的系统。类型系统的这个方面也必须扩展以支持在结构体内部存储引用。简而言之，Move的引用安全系统必须扩展以支持存储的引用，这是我们在语言发展过程中关注的事情。

<!-- TODO actually document a sketch of the borrow rules -->