---
title: '循环 | 参考'
description: ''
---

# Move中的循环构造

许多程序需要对值进行迭代，Move提供了`while`和`loop`形式来允许您在这些情况下编写代码。此外，您还可以通过使用`break`（退出循环）和`continue`（跳过此次迭代的剩余部分并返回到控制流结构顶部）在执行期间修改这些循环的控制流。

## `while`循环

`while`构造重复执行主体（类型为单元的表达式），直到条件（类型为`bool`的表达式）评估为`false`。

以下是一个简单的`while`循环示例，它计算从`1`到`n`的数字总和：

```move
fun sum(n: u64): u64 {
    let mut sum = 0;
    let mut i = 1;
    while (i <= n) {
        sum = sum + i;
        i = i + 1
    };

    sum
}
```

也允许无限`while`循环：

```move
fun foo() {
    while (true) { }
}
```

### 在`while`循环内使用`break`

在Move中，`while`循环可以使用`break`提前退出。例如，假设我们在向量中寻找一个值的位置，如果找到就想要`break`：

```move
fun find_position(values: &vector<u64>, target_value: u64): Option<u64> {
    let size = vector::length(values);
    let mut i = 0;
    let mut found = false;

    while (i < size) {
        if (vector::borrow(values, i) == &target_value) {
            found = true;
            break
        };
        i = i + 1
    };

    if (found) {
        option::some(i)
    } else {
        option::none<u64>()
    }
}
```

在这里，如果借用的向量值等于我们的目标值，我们将`found`标志设置为`true`，然后调用`break`，这将导致程序退出循环。

最后，注意`while`循环的`break`不能带值：`while`循环总是返回单元类型`()`，因此`break`也是如此。

### 在`while`循环内使用`continue`

与`break`类似，Move的`while`循环可以调用`continue`来跳过循环主体的一部分。这允许我们在不满足条件时跳过部分计算，如以下示例所示：

```move
fun sum_even(values: &vector<u64>): u64 {
    let size = vector::length(values);
    let mut i = 0;
    let mut even_sum = 0;

    while (i < size) {
        let number = *vector::borrow(values, i);
        i = i + 1;
        if (number % 2 == 1) continue;
        even_sum = even_sum + number;
    };
    even_sum
}
```

此代码将迭代提供的向量。对于每个条目，如果该条目是偶数，它将添加到`even_sum`中。但是，如果不是，它将调用`continue`，跳过求和操作并返回到`while`循环条件检查。

## `loop`表达式

`loop`表达式重复执行循环主体（类型为`()`的表达式），直到遇到`break`：

```move
fun sum(n: u64): u64 {
    let mut sum = 0;
    let mut i = 1;

    loop {
       i = i + 1;
       if (i >= n) break;
       sum = sum + i;
    };

    sum
}
```

没有`break`，循环将永远继续。在下面的示例中，程序将永远运行，因为`loop`没有`break`：

```move
fun foo() {
    let mut i = 0;
    loop { i = i + 1 }
}
```

### 在`loop`中使用带值的`break`

与总是返回`()`的`while`循环不同，`loop`可以使用`break`返回一个值。这样做时，整个`loop`表达式评估为该类型的值。例如，我们可以使用`loop`和`break`重写上面的`find_position`，如果找到就立即返回索引：

```move
fun find_position(values: &vector<u64>, target_value: u64): Option<u64> {
    let size = vector::length(values);
    let mut i = 0;

    loop {
        if (vector::borrow(values, i) == &target_value) {
            break option::some(i)
        } else if (i >= size) {
            break option::none()
        };
        i = i + 1;
    }
}
```

此循环将带着选项结果中断，并且作为函数主体中的最后一个表达式，将产生该值作为最终函数结果。

### 在`loop`表达式内使用`continue`

如您所期望的，`continue`也可以在`loop`内使用。这是使用带有`break`和`continue`的`loop`而不是`while`重写的先前`sum_even`函数。

```move
fun sum_even(values: &vector<u64>): u64 {
    let size = vector::length(values);
    let mut i = 0;
    let mut even_sum = 0;

    loop {
        if (i >= size) break;
        let number = *vector::borrow(values, i);
        i = i + 1;
        if (number % 2 == 1) continue;
        even_sum = even_sum + number;
    };
    even_sum
}
```

## `while`和`loop`的类型

在Move中，循环是类型化表达式。`while`表达式总是具有类型`()`。

```move
let () = while (i < 10) { i = i + 1 };
```

如果`loop`包含`break`，表达式具有break的类型。没有值的break具有单元类型`()`。

```move
(loop { if (i < 10) i = i + 1 else break }: ());
let () = loop { if (i < 10) i = i + 1 else break };

let x: u64 = loop { if (i < 10) i = i + 1 else break 5 };
let x: u64 = loop { if (i < 10) { i = i + 1; continue} else break 5 };
```

此外，如果循环包含多个break，它们必须都返回相同的类型：

```move
// 无效 -- 第一个break返回()，第二个返回5
let x: u64 = loop { if (i < 10) break else break 5 };
```

如果`loop`没有`break`，`loop`可以具有任何类型，就像`return`、`abort`、`break`和`continue`一样。

```move
(loop (): u64);
(loop (): address);
(loop (): &vector<vector<u8>>);
```

如果您需要更精确的控制流，例如跳出嵌套循环，下一章将介绍Move中标记控制流的使用。