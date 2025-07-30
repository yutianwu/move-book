---
title: '元组和单元 | 参考'
description: ''
---

# 元组和单元

Move并不像其他将元组作为[一等值](https://en.wikipedia.org/wiki/First-class_citizen)的语言那样完全支持元组。但是，为了支持多个返回值，Move有类似元组的表达式。这些表达式在运行时不会产生具体值（字节码中没有元组），因此它们非常有限：

- 它们只能出现在表达式中（通常在函数的返回位置）。
- 它们不能绑定到局部变量。
- 它们不能绑定到局部变量。
- 它们不能存储在结构体中。
- 元组类型不能用于实例化泛型。

类似地，[单元`()`](https://en.wikipedia.org/wiki/Unit_type)是Move源语言为了基于表达式而创建的类型。单元值`()`不会产生任何运行时值。我们可以将单元`()`视为空元组，适用于元组的任何限制也适用于单元。

考虑到这些限制，在语言中拥有元组可能感觉很奇怪。但在其他语言中元组最常见的用例之一是允许函数返回多个值。一些语言通过强制用户编写包含多个返回值的结构体来解决这个问题。但是在Move中，您不能将引用放入[结构体](./../structs_zh)中。这要求Move支持多个返回值。这些多个返回值在字节码级别都被推送到堆栈上。在源级别，这些多个返回值使用元组表示。

## 字面量

元组通过括号内以逗号分隔的表达式列表创建。

| 语法          | 类型                                                                         | 描述                                                  |
| --------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------ |
| `()`            | `(): ()`                                                                     | 单元，空元组，或元数为0的元组               |
| `(e1, ..., en)` | `(e1, ..., en): (T1, ..., Tn)`，其中`e_i: Ti`，使得`0 < i <= n`且`n > 0` | `n`元组，元数为`n`的元组，有`n`个元素的元组 |

注意`(e)`不具有类型`(e): (t)`，换句话说，没有只有一个元素的元组。如果括号内只有一个元素，括号仅用于消歧，不具有任何其他特殊含义。

有时，有两个元素的元组称为"对"，有三个元素的元组称为"三元组"。

### 示例

```move
module 0::example;

// 这三个函数都是等价的

// 当没有提供返回类型时，假定为`()`
fun returns_unit_1() { }

// 空表达式块中有隐式()值
fun returns_unit_2(): () { }

// `returns_unit_1`和`returns_unit_2`的显式版本
fun returns_unit_3(): () { () }


fun returns_3_values(): (u64, bool, address) {
    (0, false, @0x42)
}
fun returns_4_values(x: &u64): (&u64, u8, u128, vector<u8>) {
    (x, 0, 1, b"foobar")
}
```

## 操作

目前可以对元组执行的唯一操作是解构。

### 解构

对于任何大小的元组，它们都可以在`let`绑定或赋值中解构。

例如：

```move
module 0x42::example;

// 这三个函数都是等价的
fun returns_unit() {}
fun returns_2_values(): (bool, bool) { (true, false) }
fun returns_4_values(x: &u64): (&u64, u8, u128, vector<u8>) { (x, 0, 1, b"foobar") }

fun examples(cond: bool) {
    let () = ();
    let (mut x, mut y): (u8, u64) = (0, 1);
    let (mut a, mut b, mut c, mut d) = (@0x0, 0, false, b"");

    () = ();
    (x, y) = if (cond) (1, 2) else (3, 4);
    (a, b, c, d) = (@0x1, 1, true, b"1");
}

fun examples_with_function_calls() {
    let () = returns_unit();
    let (mut x, mut y): (bool, bool) = returns_2_values();
    let (mut a, mut b, mut c, mut d) = returns_4_values(&0);

    () = returns_unit();
    (x, y) = returns_2_values();
    (a, b, c, d) = returns_4_values(&1);
}
```

有关更多详细信息，请参阅[Move变量](./../variables_zh)。

## 子类型

与引用一起，元组是Move中唯一具有[子类型](https://en.wikipedia.org/wiki/Subtyping)的类型。元组只有在与引用的子类型关系中才有子类型（以协变方式）。

例如：

```move
let x: &u64 = &0;
let y: &mut u64 = &mut 1;

// (&u64, &mut u64)是(&u64, &u64)的子类型
// 因为&mut u64是&u64的子类型
let (a, b): (&u64, &u64) = (x, y);

// (&mut u64, &mut u64)是(&u64, &u64)的子类型
// 因为&mut u64是&u64的子类型
let (c, d): (&u64, &u64) = (y, y);

// highlight-error-start
// 错误！(&u64, &mut u64)不是(&mut u64, &mut u64)的子类型
// 因为&u64不是&mut u64的子类型
let (e, f): (&mut u64, &mut u64) = (x, y);
// highlight-error-end
```

## 所有权

如上所述，元组值在运行时并不真正存在。目前它们不能存储到局部变量中（但这个功能可能在未来的某个时候到来）。因此，元组目前只能被移动，因为复制它们需要首先将它们放入局部变量中。