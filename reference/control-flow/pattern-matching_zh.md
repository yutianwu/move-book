---
title: '模式匹配 | 参考'
description: ''
---

# 模式匹配

`match`表达式是一个强大的控制结构，允许您将值与一系列模式进行比较，然后根据首先匹配的模式执行代码。模式可以是从简单字面量到复杂嵌套结构体和枚举定义的任何内容。与基于`bool`类型测试表达式改变控制流的`if`表达式相反，`match`表达式可以对任何类型的值进行操作并选择多个分支中的一个。

`match`表达式可以匹配Move值以及可变或不可变引用，相应地绑定子模式。

例如：

```move
fun run(x: u64): u64 {
    match (x) {
        1 => 2,
        2 => 3,
        x => x,
    }
}

run(1); // 返回2
run(2); // 返回3
run(3); // 返回3
run(0); // 返回0
```

## `match`语法

`match`接受一个表达式和一系列由逗号分隔的非空_匹配分支_。

每个匹配分支由一个模式（`p`）、一个可选的守卫（`if (g)`，其中`g`是类型为`bool`的表达式）、一个箭头（`=>`）和一个在模式匹配时要执行的分支表达式（`e`）组成。例如，

```move
match (expression) {
    pattern1 if (guard_expression) => expression1,
    pattern2 => expression2,
    pattern3 => { expression3, expression4, ... },
}
```

匹配分支从顶部到底部依次检查，第一个匹配的模式（如果存在守卫表达式，则守卫表达式评估为`true`）将被执行。

注意`match`中的匹配分支系列必须是穷尽的，这意味着被匹配类型的每个可能值都必须被`match`中的某个模式覆盖。如果匹配分支系列不是穷尽的，编译器将引发错误。

## 模式语法

如果值等于模式，并且变量和通配符（例如，`x`、`y`、`_`或`..`）"等于"任何东西，则模式匹配该值。

模式用于匹配值。模式可以是

| 模式              | 描述                                                            |
| -------------------- | ---------------------------------------------------------------------- |
| 字面量              | 字面量值，如`1`、`true`、`@0x1`                           |
| 常量             | 常量值，例如，`MyConstant`                                   |
| 变量             | 变量，例如，`x`、`y`、`z`                                        |
| 通配符             | 通配符，例如，`_`                                                  |
| 构造器          | 构造器模式，例如，`MyStruct { x, y }`、`MyEnum::Variant(x)` |
| At模式           | At模式，例如，`x @ MyEnum::Variant(..)`                         |
| Or模式           | Or模式，例如，`MyEnum::Variant(..) \| MyEnum::OtherVariant(..)` |
| 多元通配符 | 多元通配符，例如，`MyEnum::Variant(..)`                    |
| 可变绑定      | 可变绑定模式，例如，`mut x`                               |

Move中的模式具有以下语法：

```bnf
pattern = <literal>
        | <constant>
        | <variable>
        | _
        | C { <variable> : inner-pattern ["," <variable> : inner-pattern]* } // 其中C是结构体或枚举变体
        | C ( inner-pattern ["," inner-pattern]* ... )                       // 其中C是结构体或枚举变体
        | C                                                                  // 其中C是枚举变体
        | <variable> @ top-level-pattern
        | pattern | pattern
        | mut <variable>
inner-pattern = pattern
              | ..     // 多元通配符
```

一些模式的示例：

```move
// 字面量模式
1

// 常量模式
MyConstant

// 变量模式
x

// 通配符模式
_

// 匹配具有字段`1`和`true`的`MyEnum::Variant`的构造器模式
MyEnum::Variant(1, true)

// 匹配具有字段`1`的`MyEnum::Variant`并将第二个字段的值绑定到`x`的构造器模式
MyEnum::Variant(1, x)

// 匹配`MyEnum::Variant`变体中多个字段的多元通配符模式
MyEnum::Variant(..)

// 匹配`MyStruct`的`x`字段并将`y`字段绑定到`other_variable`的构造器模式
MyStruct { x, y: other_variable }

// 匹配`MyEnum::Variant`并将整个值绑定到`x`的at模式
x @ MyEnum::Variant(..)

// 匹配`MyEnum::Variant`或`MyEnum::OtherVariant`的or模式
MyEnum::Variant(..) | MyEnum::OtherVariant(..)

// 与上面的or模式相同，但使用显式通配符
MyEnum::Variant(_, _) | MyEnum::OtherVariant(_, _)

// 匹配`MyEnum::Variant`或`MyEnum::OtherVariant`并将u64字段绑定到`x`的or模式
MyEnum::Variant(x, _) | MyEnum::OtherVariant(_, x)

// 匹配`OtherEnum::V`并且如果内部`MyEnum`是`MyEnum::Variant`的构造器模式
OtherEnum::V(MyEnum::Variant(..))
```

### 模式和变量

包含变量的模式将它们绑定到被匹配的匹配主题或主题子组件。然后可以在任何匹配守卫表达式中或在匹配分支的右侧使用这些变量。例如：

```move
public struct Wrapper(u64)

fun add_under_wrapper_unless_equal(wrapper: Wrapper, x: u64): Wrapper {
    match (wrapper) {
        Wrapper(y) if (y == x) => Wrapper(y),
        Wrapper(y) => Wrapper(y + x),
    }
}
add_under_wrapper_unless_equal(Wrapper(1), 2); // 返回Wrapper(3)
add_under_wrapper_unless_equal(Wrapper(2), 3); // 返回Wrapper(5)
add_under_wrapper_unless_equal(Wrapper(3), 3); // 返回Wrapper(3)
```

### 组合模式

模式可以嵌套，但模式也可以使用or操作符（`|`）组合。例如，如果模式`p1`或`p2`匹配主题，则`p1 | p2`成功。这种模式可以出现在任何地方——要么作为顶级模式，要么作为另一个模式中的子模式。

```move
public enum MyEnum has drop {
    Variant(u64, bool),
    OtherVariant(bool, u64),
}

fun test_or_pattern(x: u64): u64 {
    match (x) {
        MyEnum::Variant(1 | 2 | 3, true) | MyEnum::OtherVariant(true, 1 | 2 | 3) => 1,
        MyEnum::Variant(8, true) | MyEnum::OtherVariant(_, 6 | 7) => 2,
        _ => 3,
    }
}

test_or_pattern(MyEnum::Variant(3, true)); // 返回1
test_or_pattern(MyEnum::OtherVariant(true, 2)); // 返回1
test_or_pattern(MyEnum::Variant(8, true)); // 返回2
test_or_pattern(MyEnum::OtherVariant(false, 7)); // 返回2
test_or_pattern(MyEnum::OtherVariant(false, 80)); // 返回3
```

### 某些模式的限制

`mut`和`..`模式在何时、何地以及如何使用方面也有特定条件，详见[特定模式的限制](#limitations-on-specific-patterns)。从高层次来看，`mut`修饰符只能用于变量模式，`..`模式只能在构造器模式中使用一次——而不能作为顶级模式。

以下是`..`模式的_无效_用法，因为它用作顶级模式：

```move
match (x) {
    .. => 1,
    // 错误：`..`模式只能在构造器模式中使用
}

match (x) {
    MyStruct(.., ..) => 1,
    // 错误：    ^^  `..`模式在构造器模式中只能使用一次
}
```

### 模式类型

模式不是表达式，但它们仍然是有类型的。这意味着模式的类型必须与它匹配的值的类型匹配。例如，模式`1`具有整数类型，模式`MyEnum::Variant(1, true)`具有类型`MyEnum`，模式`MyStruct { x, y }`具有类型`MyStruct`，`OtherStruct<bool> { x: true, y: 1}`具有类型`OtherStruct<bool>`。如果您尝试匹配与匹配中模式类型不同的表达式，这将导致类型错误。例如：

```move
match (1) {
    // `true`字面量模式是`bool`类型，所以这是一个类型错误。
    true => 1,
    // 类型错误：期望类型u64，发现bool
    _ => 2,
}
```

类似地，以下内容也会导致类型错误，因为`MyEnum`和`MyStruct`是不同的类型：

```move
match (MyStruct { x: 0, y: 0 }) {
    MyEnum::Variant(..) => 1,
    // 类型错误：期望类型MyEnum，发现MyStruct
}
```

## 匹配

在深入模式匹配的具体细节以及值"匹配"模式意味着什么之前，让我们看一些示例来提供对概念的直觉。

```move
fun test_lit(x: u64): u8 {
    match (x) {
        1 => 2,
        2 => 3,
        _ => 4,
    }
}
test_lit(1); // 返回2
test_lit(2); // 返回3
test_lit(3); // 返回4
test_lit(10); // 返回4

fun test_var(x: u64): u64 {
    match (x) {
        y => y,
    }
}
test_var(1); // 返回1
test_var(2); // 返回2
test_var(3); // 返回3
...

const MyConstant: u64 = 10;
fun test_constant(x: u64): u64 {
    match (x) {
        MyConstant => 1,
        _ => 2,
    }
}
test_constant(MyConstant); // 返回1
test_constant(10); // 返回1
test_constant(20); // 返回2

fun test_or_pattern(x: u64): u64 {
    match (x) {
        1 | 2 | 3 => 1,
        4 | 5 | 6 => 2,
        _ => 3,
    }
}
test_or_pattern(3); // 返回1
test_or_pattern(5); // 返回2
test_or_pattern(70); // 返回3

fun test_or_at_pattern(x: u64): u64 {
    match (x) {
        x @ (1 | 2 | 3) => x + 1,
        y @ (4 | 5 | 6) => y + 2,
        z => z + 3,
    }
}
test_or_pattern(2); // 返回3
test_or_pattern(5); // 返回7
test_or_pattern(70); // 返回73
```

从这些示例中要注意的最重要的事情是，如果值等于模式，则模式匹配值，通配符/变量模式匹配任何东西。这对于字面量、变量和常量都是如此。例如，在`test_lit`函数中，值`1`匹配模式`1`，值`2`匹配模式`2`，值`3`匹配通配符`_`。类似地，在`test_var`函数中，值`1`和值`2`都匹配模式`y`。

变量`x`匹配（或"等于"）任何值，通配符`_`匹配任何值（但只匹配一个值）。Or模式就像逻辑OR，如果值匹配or模式中的任何模式，则值匹配模式，所以`p1 | p2 | p3`应该读作"匹配p1、或p2、或p3"。

### 匹配构造器

模式匹配包括构造器模式的概念。这些模式允许您检查和访问结构体和枚举的深层内容，是模式匹配最强大的部分之一。构造器模式，加上变量绑定，允许您通过值的结构匹配值，并提取您关心的值的部分以在匹配分支的右侧使用。

考虑以下：

```move
fun f(x: MyEnum): u64 {
    match (x) {
        MyEnum::Variant(1, true) => 1,
        MyEnum::OtherVariant(_, 3) => 2,
        MyEnum::Variant(..) => 3,
        MyEnum::OtherVariant(..) => 4,
    }
}
f(MyEnum::Variant(1, true)); // 返回1
f(MyEnum::Variant(2, true)); // 返回3
f(MyEnum::OtherVariant(false, 3)); // 返回2
f(MyEnum::OtherVariant(true, 3)); // 返回2
f(MyEnum::OtherVariant(true, 2)); // 返回4
```

这是说"如果`x`是具有字段`1`和`true`的`MyEnum::Variant`，则返回`1`。如果它是具有第一个字段的任何值和第二个字段为`3`的`MyEnum::OtherVariant`，则返回`2`。如果它是具有任何字段的`MyEnum::Variant`，则返回`3`。最后，如果它是具有任何字段的`MyEnum::OtherVariant`，则返回`4`"。

您也可以嵌套模式。因此，如果您想匹配1、2或10，而不仅仅是在前面的`MyEnum::Variant`中匹配1，您可以使用or模式：

```move
fun f(x: MyEnum): u64 {
    match (x) {
        MyEnum::Variant(1 | 2 | 10, true) => 1,
        MyEnum::OtherVariant(_, 3) => 2,
        MyEnum::Variant(..) => 3,
        MyEnum::OtherVariant(..) => 4,
    }
}
f(MyEnum::Variant(1, true)); // 返回1
f(MyEnum::Variant(2, true)); // 返回1
f(MyEnum::Variant(10, true)); // 返回1
f(MyEnum::Variant(10, false)); // 返回3
```

### 能力约束

此外，匹配绑定受到与Move其他方面相同的能力限制。特别是，如果您尝试使用通配符匹配没有`drop`的值（非引用），编译器将发出错误信号，因为通配符期望丢弃值。类似地，如果您使用绑定器绑定非`drop`值，它必须在匹配分支的右侧使用。此外，如果您完全解构该值，您已经解包了它，匹配[非`drop`结构体解包](./../structs_zh#destroying-structs-via-pattern-matching)的语义。有关`drop`能力的更多详细信息，请参阅[能力部分的`drop`](./../abilities_zh#drop)。

```move
public struct NonDrop(u64)

fun drop_nondrop(x: NonDrop): u64 {
    match (x) {
        NonDrop(1) => 1,
        _ => 2
        // 错误：无法对非可丢弃值进行通配符匹配
    }
}

fun destructure_nondrop(x: NonDrop): u64 {
    match (x) {
        NonDrop(1) => 1,
        NonDrop(_) => 2
        // 可以！
    }
}

fun use_nondrop(x: NonDrop): NonDrop {
    match (x) {
        NonDrop(1) => NonDrop(8),
        x => x
    }
}
```

## 穷尽性

Move中的`match`表达式必须是_穷尽的_：被匹配类型的每个可能值都必须被匹配的某个分支中的某个模式覆盖。如果匹配分支系列不是穷尽的，编译器将引发错误。注意，任何具有守卫表达式的分支都不会对匹配穷尽性有贡献，因为它可能在运行时无法匹配。

例如，只有当`u8`上的匹配匹配从0到255（包括）的_每个_数字时，匹配才是穷尽的，除非存在通配符或变量模式。类似地，`bool`上的匹配需要匹配`true`和`false`，除非存在通配符或变量模式。

对于结构体，因为类型只有一种构造器，只需要匹配一个构造器，但结构体内的字段也需要穷尽匹配。相反，枚举可能定义多个变体，每个变体都必须匹配（包括任何子字段）才能将匹配视为穷尽。

因为下划线和变量是匹配任何东西的通配符，它们被计算为匹配在该位置匹配的类型的所有值。此外，多元通配符模式`..`可用于匹配结构体或枚举变体中的多个值。

要查看一些_非穷尽_匹配的示例，请考虑以下：

```move
public enum MyEnum {
    Variant(u64, bool),
    OtherVariant(bool, u64),
}

public struct Pair<T>(T, T)

fun f(x: MyEnum): u8 {
    match (x) {
        MyEnum::Variant(1, true) => 1,
        MyEnum::Variant(_, _) => 1,
        MyEnum::OtherVariant(_, 3) => 2,
        // 错误：不穷尽，因为值`MyEnum::OtherVariant(_, 4)`没有匹配。
    }
}

fun match_pair_bool(x: Pair<bool>): u8 {
    match (x) {
        Pair(true, true) => 1,
        Pair(true, false) => 1,
        Pair(false, false) => 1,
        // 错误：不穷尽，因为值`Pair(false, true)`没有匹配。
    }
}
```

然后可以通过在匹配分支末尾添加通配符模式或完全匹配剩余值来使这些示例穷尽：

```move
fun f(x: MyEnum): u8 {
    match (x) {
        MyEnum::Variant(1, true) => 1,
        MyEnum::Variant(_, _) => 1,
        MyEnum::OtherVariant(_, 3) => 2,
        // 现在穷尽，因为这将匹配MyEnum::OtherVariant的所有值
        MyEnum::OtherVariant(..) => 2,

    }
}

fun match_pair_bool(x: Pair<bool>): u8 {
    match (x) {
        Pair(true, true) => 1,
        Pair(true, false) => 1,
        Pair(false, false) => 1,
        // 现在穷尽，因为这将匹配Pair<bool>的所有值
        Pair(false, true) => 1,
    }
}
```

## 守卫

如前所述，您可以通过在模式后添加`if`子句来为匹配分支添加守卫。这个守卫将在模式匹配_之后_但在箭头右侧的表达式被评估_之前_运行。如果守卫表达式评估为`true`，则将评估箭头右侧的表达式，如果评估为`false`，则将被视为匹配失败，并检查`match`表达式中的下一个匹配分支。

```move
fun match_with_guard(x: u64): u64 {
    match (x) {
        1 if (false) => 1,
        1 => 2,
        _ => 3,
    }
}

match_with_guard(1); // 返回2
match_with_guard(0); // 返回3
```

守卫表达式可以在评估期间引用模式中绑定的变量。但是，请注意，_变量在守卫中仅作为不可变引用可用_，无论匹配的模式如何——即使变量上有可变性说明符或模式是按值匹配的。

```move
fun incr(x: &mut u64) {
    *x = *x + 1;
}

fun match_with_guard_incr(x: u64): u64 {
    match (x) {
        x if ({ incr(&mut x); x == 1 }) => 1,
        // 错误：    ^^^ 对不可变值的无效借用
        _ => 2,
    }
}

fun match_with_guard_incr2(x: &mut u64): u64 {
    match (x) {
        x if ({ incr(&mut x); x == 1 }) => 1,
        // 错误：    ^^^ 对不可变值的无效借用
        _ => 2,
    }
}
```

此外，重要的是要注意，任何具有守卫表达式的匹配分支都不会被考虑用于穷尽性目的，因为编译器无法静态评估守卫表达式。

## 特定模式的限制

对于何时可以在模式中使用`..`和`mut`模式修饰符，有一些限制。

### 可变性用法

`mut`修饰符可以放在变量模式上，以指定_变量_将在匹配分支的右侧表达式中被修改。请注意，由于`mut`修饰符只表示变量将被修改，而不是底层数据，这可以用于所有类型的匹配（按值、不可变引用和可变引用）。

请注意，`mut`修饰符只能应用于变量，而不能应用于其他类型的模式。

```move
public struct MyStruct(u64)

fun top_level_mut(x: MyStruct): u64 {
    match (x) {
        mut MyStruct(y) => 1,
        // 错误：不能在非变量模式上使用mut
    }
}

fun mut_on_immut(x: &MyStruct): u64 {
    match (x) {
        MyStruct(mut y) => {
            y = &(*y + 1);
            *y
        }
    }
}

fun mut_on_value(x: MyStruct): u64 {
    match (x) {
        MyStruct(mut y) => {
            *y = *y + 1;
            *y
        },
    }
}

fun mut_on_mut(x: &mut MyStruct): u64 {
    match (x) {
        MyStruct(mut y) => {
            *y = *y + 1;
            *y
        },
    }
}

let mut x = MyStruct(1);

mut_on_mut(&mut x); // 返回2
x.0; // 返回2

mut_on_immut(&x); // 返回3
x.0; // 返回2

mut_on_value(x); // 返回3
```

### `..`用法

`..`模式只能在构造器模式中用作匹配任意数量字段的通配符——编译器将`..`扩展为在构造器模式中的任何缺失字段中插入`_`（如果有）。因此`MyStruct(_, _, _)`与`MyStruct(..)`相同，`MyStruct(1, _, _)`与`MyStruct(1, ..)`相同。由于这个原因，`..`模式的使用方式和位置有一些限制：

- 在构造器模式中只能使用**一次**；
- 在位置参数中，它可以在构造器中的模式的开头、中间或末尾使用；
- 在命名参数中，它只能在构造器中的模式的末尾使用；

```move
public struct MyStruct(u64, u64, u64, u64) has drop;

public struct MyStruct2 {
    x: u64,
    y: u64,
    z: u64,
    w: u64,
}

fun wild_match(x: MyStruct): u64 {
    match (x) {
        MyStruct(.., 1) => 1,
        // 可以！`..`模式可以在构造器模式的开头使用
        MyStruct(1, ..) => 2,
        // 可以！`..`模式可以在构造器模式的末尾使用
        MyStruct(1, .., 1) => 3,
        // 可以！`..`模式可以在构造器模式的中间使用
        MyStruct(1, .., 1, 1) => 4,
        MyStruct(..) => 5,
    }
}

fun wild_match2(x: MyStruct2): u64 {
    match (x) {
        MyStruct2 { x: 1, .. } => 1,
        MyStruct2 { x: 1, w: 2 .. } => 2,
        MyStruct2 { .. } => 3,
    }
}
```