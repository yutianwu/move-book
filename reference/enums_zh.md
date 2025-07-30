---
title: '枚举 | 参考手册'
description: ''
---

# 枚举

_枚举_ 是用户定义的数据结构，包含一个或多个 _变体_。每个变体可以选择性地包含类型化字段。这些字段的数量和类型对于枚举中的每个变体可以不同。枚举中的字段可以存储任何非引用、非元组类型，包括其他结构体或枚举。

作为一个简单的例子，考虑在 Move 中的以下枚举定义：

```move
public enum Action {
    Stop,
    Pause { duration: u32 },
    MoveTo { x: u64, y: u64 },
    Jump(u64),
}
```

这声明了一个枚举 `Action`，表示游戏中可以采取的不同动作——你可以 `Stop`、`Pause` 指定持续时间、`MoveTo` 特定位置，或 `Jump` 到特定高度。

与结构体类似，枚举可以有[能力](./abilities)来控制可以对它们执行哪些操作。但是需要注意的是，枚举不能具有 `key` 能力，因为它们不能作为顶层对象。

## 定义枚举

枚举必须在模块中定义，枚举必须至少包含一个变体，枚举的每个变体可以没有字段、有位置字段或有命名字段。以下是每种情况的一些示例：

```move
module a::m;

public enum Foo has drop {
    VariantWithNoFields,
    //                 ^ 注意：在变体声明后有尾随逗号是可以的
}
public enum Bar has copy, drop {
    VariantWithPositionalFields(u64, bool),
}
public enum Baz has drop {
    VariantWithNamedFields { x: u64, y: bool, z: Bar },
}
```

枚举在其任何变体中都不能是递归的，因此以下枚举定义是不允许的，因为它们在至少一个变体中是递归的。

错误的：

```move
module a::m;

public enum Foo {
    Recursive(Foo),
    //        ^ 错误：递归枚举变体
}
public enum List {
    Nil,
    Cons { head: u64, tail: List },
    //                      ^ 错误：递归枚举变体
}
public enum BTree<T> {
    Leaf(T),
    Node { left: BTree<T>, right: BTree<T> },
    //           ^ 错误：递归枚举变体
}

// 相互递归的枚举也不允许
public enum MutuallyRecursiveA {
    Base,
    Other(MutuallyRecursiveB),
    //    ^^^^^^^^^^^^^^^^^^ 错误：递归枚举变体
}

public enum MutuallyRecursiveB {
    Base,
    Other(MutuallyRecursiveA),
    //    ^^^^^^^^^^^^^^^^^^ 错误：递归枚举变体
}
```

## 可见性

所有枚举都声明为 `public`。这意味着枚举的类型可以从任何其他模块引用。但是，枚举的变体、每个变体中的字段，以及创建或销毁枚举变体的能力都是定义该枚举的模块内部的。

### 能力

就像结构体一样，默认情况下枚举声明是线性的和临时的。要以非线性或非临时的方式使用枚举值——即复制、丢弃或存储在[对象](./abilities/object)中——你需要通过用 `has <ability>` 注解来赋予它额外的[能力](./abilities)：

```move
module a::m;

public enum Foo has copy, drop {
    VariantWithNoFields,
}
```

能力声明可以出现在枚举变体之前或之后，但只能使用其中一种，不能同时使用。如果在变体之后声明，能力声明必须以分号结尾：

```move
module a::m;

public enum PreNamedAbilities has copy, drop { Variant }
public enum PostNamedAbilities { Variant } has copy, drop;
public enum PostNamedAbilitiesInvalid { Variant } has copy, drop
//                                                              ^ 错误！缺少分号

public enum NamedInvalidAbilities has copy { Variant } has drop;
//                                                     ^ 错误！重复的能力声明
```

更多详情，请参阅[注解能力](./abilities#annotating-structs-and-enums)部分。

## 命名

枚举和枚举中的变体必须以大写字母 `A` 到 `Z` 开头。在第一个字母之后，枚举名称可以包含下划线 `_`、小写字母 `a` 到 `z`、大写字母 `A` 到 `Z` 或数字 `0` 到 `9`。

```move
public enum Foo { Variant }
public enum BAR { Variant }
public enum B_a_z_4_2 { V_a_riant_0 }
```

这种以 `A` 到 `Z` 开头的命名限制是为了给未来的语言特性留出空间。

## 使用枚举

### 创建枚举变体

枚举类型的值可以通过指示枚举的变体，然后为变体中的每个字段提供值来创建（或"打包"）。变体名称必须始终由枚举名称限定。

与结构体类似，对于具有命名字段的变体，字段的顺序不重要，但需要提供字段名称。对于具有位置字段的变体，字段的顺序很重要，字段的顺序必须与变体声明中的顺序匹配。它还必须使用 `()` 而不是 `{}` 创建。如果变体没有字段，变体名称就足够了，不需要使用 `()` 或 `{}`。

```move
module a::m;

public enum Action has drop {
    Stop,
    Pause { duration: u32 },
    MoveTo { x: u64, y: u64 },
    Jump(u64),
}
public enum Other has drop {
    Stop(u64),
}

fun example() {
    // 注意：`Action` 的 `Stop` 变体没有字段，所以不需要括号或大括号。
    let stop = Action::Stop;
    let pause = Action::Pause { duration: 10 };
    let move_to = Action::MoveTo { x: 10, y: 20 };
    let jump = Action::Jump(10);
    // 注意：`Other` 的 `Stop` 变体确实有位置字段，所以我们需要提供它们。
    let other_stop = Other::Stop(10);
}
```

对于具有命名字段的变体，你也可以使用你可能从结构体中熟悉的简写语法来创建变体：

```move
let duration = 10;

let pause = Action::Pause { duration: duration };
// 等价于
let pause = Action::Pause { duration };
```

### 模式匹配枚举变体和解构

由于枚举值可以采用不同的形状，不允许像结构体字段那样对变体字段进行点访问。相反，要访问变体中的字段——无论是按值、不可变引用还是可变引用——你必须使用模式匹配。

你可以按值、不可变引用和可变引用对 Move 值进行模式匹配。按值模式匹配时，值被移动到匹配分支中。按引用模式匹配时，值被借用到匹配分支中（不可变或可变）。我们将在这里简要描述使用 `match` 进行模式匹配，但有关在 Move 中使用 `match` 进行模式匹配的更多信息，请参阅[模式匹配](./control-flow/pattern-matching)部分。

`match` 语句用于对 Move 值进行模式匹配，由多个 _匹配分支_ 组成。每个匹配分支由一个模式、一个箭头 `=>`、一个表达式和一个逗号 `,` 组成。模式可以是结构体、枚举变体、绑定（`x`、`y`）、通配符（`_` 或 `..`）、常量（`ConstValue`）或字面值（`true`、`42` 等）。值从上到下与每个模式匹配，并将匹配第一个结构上匹配该值的模式。一旦值匹配，就执行 `=>` 右侧的表达式。

此外，匹配分支可以有可选的 _守卫_，在模式匹配后但在执行表达式 _之前_ 检查。守卫由 `if` 关键字指定，后跟必须在 `=>` 之前计算为布尔值的表达式。

```move
module a::m;

public enum Action has drop {
    Stop,
    Pause { duration: u32 },
    MoveTo { x: u64, y: u64 },
    Jump(u64),
}

public struct GameState {
    // 包含游戏状态的字段
    character_x: u64,
    character_y: u64,
    character_height: u64,
    // ...
}

fun perform_action(stat: &mut GameState, action: Action) {
    match (action) {
        // 处理 `Stop` 变体
        Action::Stop => state.stop(),
        // 处理 `Pause` 变体
        // 如果持续时间为 0，什么都不做
        Action::Pause { duration: 0 } => (),
        Action::Pause { duration } => state.pause(duration),
        // 处理 `MoveTo` 变体
        Action::MoveTo { x, y } => state.move_to(x, y),
        // 处理 `Jump` 变体
        // 如果游戏不允许跳跃，则什么都不做
        Action::Jump(_) if (state.jumps_not_allowed()) => (),
        // 否则，跳到指定高度
        Action::Jump(height) => state.jump(height),
    }
}
```

要了解如何对枚举进行模式匹配以可变地更新其中的值，让我们看看以下简单枚举的示例，它有两个变体，每个变体都有一个字段。然后我们可以编写两个函数，一个只增加第一个变体的值，另一个只增加第二个变体的值：

```move
module a::m;

public enum SimpleEnum {
    Variant1(u64),
    Variant2(u64),
}

public fun incr_enum_variant1(simple_enum: &mut SimpleEnum) {
    match (simple_enum) {
        SimpleEnum::Variant1(mut value) => *value += 1,
        _ => (),
    }
}

public fun incr_enum_variant2(simple_enum: &mut SimpleEnum) {
    match (simple_enum) {
        SimpleEnum::Variant2(mut value) => *value += 1,
        _ => (),
    }
}
```

现在，如果我们有一个 `SimpleEnum` 的值，我们可以使用函数来增加此变体的值：

```move
let mut x = SimpleEnum::Variant1(10);
incr_enum_variant1(&mut x);
assert!(x == SimpleEnum::Variant1(11));
// 不会增加，因为它增加了不同的变体
incr_enum_variant2(&mut x);
assert!(x == SimpleEnum::Variant1(11));
```

当对没有 `drop` 能力的 Move 值进行模式匹配时，该值必须在每个匹配分支中被消费或解构。如果值在匹配分支中没有被消费或解构，编译器将引发错误。这是为了确保在匹配语句中处理所有可能的值。

例如，考虑以下代码：

```move
module a::m;

public enum X { Variant { x: u64 } }

public fun bad(x: X) {
    match (x) {
        _ => (),
    // ^ 错误！类型 `X` 的值在此匹配分支中没有被消费或解构
    }
}
```

要正确处理这个问题，你需要在匹配分支中解构 `X` 及其所有变体：

```move
module a::m;

public enum X { Variant { x: u64 } }

public fun good(x: X) {
    match (x) {
        // 好的！编译通过，因为值被解构了
        X::Variant { x: _ } => (),
    }
}
```

### 覆盖枚举值

只要枚举具有 `drop` 能力，你就可以用相同类型的新值覆盖枚举的值，就像你在 Move 中对其他值所做的那样。

```move
module a::m;

public enum X has drop {
    A(u64),
    B(u64),
}

public fun overwrite_enum(x: &mut X) {
    *x = X::A(10);
}
```

```move
let mut x = X::B(20);
overwrite_enum(&mut x);
assert!(x == X::A(10));
```