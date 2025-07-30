---
title: 'Method Syntax | Reference'
description: ''
---

# 方法

作为语法便利，Move 中的一些函数可以作为值的"方法"调用。这通过使用 `.` 操作符调用函数来完成，其中 `.` 左侧的值是函数的第一个参数（有时称为接收者）。该值的类型静态确定调用哪个函数。这与其他一些语言的重要区别在于，这种语法可能表示动态调用，其中要调用的函数在运行时确定。在 Move 中，所有函数调用都是静态确定的。

简而言之，这种语法的存在是为了更容易调用函数，而不必使用 `use` 创建别名，也不必显式借用函数的第一个参数。此外，这可以使代码更易读，因为它减少了调用函数所需的样板代码量，并使链式函数调用更容易。

## 语法

调用方法的语法如下：

```text
<expression> . <identifier> <[type_arguments],*> ( <arguments> )
```

例如

```move
coin.value();
*nums.borrow_mut(i) = 5;
```

## 方法解析

调用方法时，编译器将根据接收者（`.` 左侧的参数）的类型静态确定调用哪个函数。编译器维护从类型和方法名到应该调用的模块和函数名的映射。此映射由当前范围内的 `use fun` 别名以及接收者类型定义模块中的相应函数创建。在所有情况下，接收者类型都是函数的第一个参数，无论是按值还是按引用。

在本节中，当我们说方法"解析"为函数时，我们的意思是编译器将静态地用正常的[函数](./functions)调用替换该方法。例如，如果我们有 `x.foo(e)`，其中 `foo` 解析为 `a::m::foo`，编译器将用 `a::m::foo(x, e)` 替换 `x.foo(e)`，可能会[自动借用](#automatic-borrowing) `x`。

### 定义模块中的函数

在类型的定义模块中，当类型是函数中的第一个参数时，编译器将自动为其类型的任何函数声明创建方法别名。例如，

```move
module a::m;

public struct X() has copy, drop, store;
public fun foo(x: &X) { ... }
public fun bar(flag: bool, x: &X) { ... }
```

函数 `foo` 可以作为类型 `X` 值的方法调用。但是，不是第一个参数（并且不会为 `bool` 创建一个，因为 `bool` 没有在该模块中定义）。例如，

```move
fun example(x: a::m::X) {
    x.foo(); // 有效
    // x.bar(true); 错误！
}
```

### `use fun` 别名

与传统的 [`use`](uses) 类似，`use fun` 语句创建一个局部于其当前范围的别名。这可能是当前模块或当前表达式块。但是，别名与类型相关联。

`use fun` 语句的语法如下：

```move
use fun <function> as <type>.<method alias>;
```

这为 `<function>` 创建一个别名，`<type>` 可以将其作为 `<method alias>` 接收。

例如

```move
module a::cup;

public struct Cup<T>(T) has copy, drop, store;

public fun cup_borrow<T>(c: &Cup<T>): &T {
    &c.0
}

public fun cup_value<T>(c: Cup<T>): T {
    let Cup(t) = c;
    t
}

public fun cup_swap<T: drop>(c: &mut Cup<T>, t: T) {
    c.0 = t;
}
```

我们现在可以为这些函数创建 `use fun` 别名

```move
module b::example;

use fun a::cup::cup_borrow as Cup.borrow;
use fun a::cup::cup_value as Cup.value;
use fun a::cup::cup_swap as Cup.set;

fun example(c: &mut Cup<u64>) {
    let _ = c.borrow(); // 解析为 a::cup::cup_borrow
    let v = c.value(); // 解析为 a::cup::cup_value
    c.set(v * 2); // 解析为 a::cup::cup_swap
}
```

注意 `use fun` 中的 `<function>` 不必是完全解析的路径，可以使用别名，因此上面示例中的声明可以等价地写成

```move
use a::cup::{Self, cup_swap};

use fun cup::cup_borrow as Cup.borrow;
use fun cup::cup_value as Cup.value;
use fun cup_swap as Cup.set;
```

虽然这些示例对于重命名当前模块中的函数很有用，但该功能对于在其他模块的类型上声明方法可能更有用。例如，如果我们想向 `Cup` 添加新的实用工具，我们可以使用 `use fun` 别名并仍然使用方法语法

```move
module b::example;

fun double(c: &Cup<u64>): Cup<u64> {
    let v = c.value();
    Cup::new(v * 2)
}
```

通常，我们会被迫调用它为 `double(&c)`，因为 `b::example` 没有定义 `Cup`，但我们可以使用 `use fun` 别名

```move
fun double_double(c: Cup<u64>): (Cup<u64>, Cup<u64>) {
    use fun b::example::double as Cup.dub;
    (c.dub(), c.dub()) // 在两个调用中都解析为 b::example::double
}
```

虽然 `use fun` 可以在任何范围内进行，但 `use fun` 的目标 `<function>` 必须有一个与 `<type>` 相同的第一个参数。

```move
public struct X() has copy, drop, store;

fun new(): X { X() }
fun flag(flag: bool): u8 { if (flag) 1 else 0 }

use fun new as X.new; // 错误！
use fun flag as X.flag; // 错误！
// `new` 和 `flag` 都没有类型 `X` 的第一个参数
```

但可以使用 `<type>` 的任何第一个参数，包括引用和可变引用

```move
public struct X() has copy, drop, store;

public fun by_val(_: X) {}
public fun by_ref(_: &X) {}
public fun by_mut(_: &mut X) {}

// 所有 3 个都有效，在任何范围内
use fun by_val as X.v;
use fun by_ref as X.r;
use fun by_mut as X.m;
```

注意对于泛型，方法与泛型类型的*所有*实例相关联。您不能重载方法以根据实例化解析为不同的函数。

```move
public struct Cup<T>(T) has copy, drop, store;

public fun value<T: copy>(c: &Cup<T>): T {
    c.0
}

use fun value as Cup<bool>.flag; // 错误！
use fun value as Cup<u64>.num; // 错误！
// 在这两种情况下，`use fun` 别名不能是泛型的，它们必须对类型的所有实例都有效
```

### `public use fun` 别名

与传统的 [`use`](uses) 不同，`use fun` 语句可以设为 `public`，这允许它在其声明范围之外使用。如果在定义接收者类型的模块中声明，`use fun` 可以设为 `public`，就像为定义模块中的函数[自动创建](#functions-in-the-defining-module)的方法别名一样。或者相反，可以认为为定义模块中每个具有接收者类型第一个参数的函数（如果在该模块中定义）自动创建隐式 `public use fun`。这两种观点是等价的。

```move
module a::cup;

public struct Cup<T>(T) has copy, drop, store;

public use fun cup_borrow as Cup.borrow;
public fun cup_borrow<T>(c: &Cup<T>): &T {
    &c.0
}
```

在这个例子中，为 `a::cup::Cup.borrow` 和 `a::cup::Cup.cup_borrow` 创建了公共方法别名。两者都解析为 `a::cup::cup_borrow`。并且两者都是"公共的"，意思是它们可以在 `a::cup` 之外使用，而无需额外的 `use` 或 `use fun`。

```move
module b::example;

fun example<T: drop>(c: a::cup::Cup<u64>) {
    c.borrow(); // 解析为 a::cup::cup_borrow
    c.cup_borrow(); // 解析为 a::cup::cup_borrow
}
```

因此，`public use fun` 声明用作重命名函数的方式，如果您想为其在方法语法中使用提供更清晰的名称。如果您有一个具有多种类型的模块，并且每种类型都有类似命名的函数，这特别有用。

```move
module a::shapes;

public struct Rectangle { base: u64, height: u64 }
public struct Box { base: u64, height: u64, depth: u64 }

// Rectangle 和 Box 可以有同名的方法

public use fun rectangle_base as Rectangle.base;
public fun rectangle_base(rectangle: &Rectangle): u64 {
    rectangle.base
}

public use fun box_base as Box.base;
public fun box_base(box: &Box): u64 {
    box.base
}
```

`public use fun` 的另一个用途是向其他模块的类型添加方法。这与分布在单个包中的函数结合使用时很有用。

```move
module a::cup {
    public struct Cup<T>(T) has copy, drop, store;

    public fun new<T>(t: T): Cup<T> { Cup(t) }
    public fun borrow<T>(c: &Cup<T>): &T {
        &c.0
    }
    // `public use fun` 到在另一个模块中定义的函数
    public use fun a::utils::split as Cup.split;
}

module a::utils {
    use a::m::{Self, Cup};

    public fun split<u64>(c: Cup<u64>): (Cup<u64>, Cup<u64>) {
        let Cup(t) = c;
        let half = t / 2;
        let rem = if (t > 0) t - half else 0;
        (cup::new(half), cup::new(rem))
    }

}
```

注意这个 `public use fun` 不会创建循环依赖，因为 `use fun` 在模块编译后不存在——所有方法都是静态解析的。

### 与 `use` 别名的交互

需要注意的一个小细节是方法别名尊重正常的 `use` 别名。

```move
module a::cup {
    public struct Cup<T>(T) has copy, drop, store;

    public fun cup_borrow<T>(c: &Cup<T>): &T {
        &c.0
    }
}

module b::other {
    use a::cup::{Cup, cup_borrow as borrow};

    fun example(c: &Cup<u64>) {
        c.borrow(); // 解析为 a::cup::cup_borrow
    }
}
```

思考这个问题的有用方式是 `use` 在可能的情况下为函数创建隐式 `use fun` 别名。在这种情况下，`use a::cup::cup_borrow as borrow` 创建隐式 `use fun a::cup::cup_borrow as Cup.borrow`，因为它将是有效的 `use fun` 别名。这两种观点是等价的。这种推理可以告知特定方法如何在遮蔽下解析。更多详细信息请参见[作用域](#scoping)中的情况。

### 作用域

如果不是 `public`，`use fun` 别名局限于其范围，就像正常的 [`use`](uses) 一样。例如

```move
module a::m {
    public struct X() has copy, drop, store;
    public fun foo(_: &X) {}
    public fun bar(_: &X) {}
}

module b::other {
    use a::m::X;

    use fun a::m::foo as X.f;

    fun example(x: &X) {
        x.f(); // 解析为 a::m::foo
        {
            use a::m::bar as f;
            x.f(); // 解析为 a::m::bar
        };
        x.f(); // 仍然解析为 a::m::foo
        {
            use fun a::m::bar as X.f;
            x.f(); // 解析为 a::m::bar
        }
    }
```

## 自动借用

解析方法时，如果函数期望引用，编译器将自动借用接收者。例如

```move
module a::m;

public struct X() has copy, drop;
public fun by_val(_: X) {}
public fun by_ref(_: &X) {}
public fun by_mut(_: &mut X) {}

fun example(mut x: X) {
    x.by_ref(); // 解析为 a::m::by_ref(&x)
    x.by_mut(); // 解析为 a::m::by_mut(&mut x)
}
```

在这些例子中，`x` 分别自动借用为 `&x` 和 `&mut x`。这也适用于字段访问

```move
module a::m;

public struct X() has copy, drop;
public fun by_val(_: X) {}
public fun by_ref(_: &X) {}
public fun by_mut(_: &mut X) {}

public struct Y has drop { x: X }

fun example(mut y: Y) {
    y.x.by_ref(); // 解析为 a::m::by_ref(&y.x)
    y.x.by_mut(); // 解析为 a::m::by_mut(&mut y.x)
}
```

注意在两个例子中，局部变量必须标记为 [`mut`](./variables) 以允许 `&mut` 借用。如果没有这个，会出现错误，说 `x`（或第二个例子中的 `y`）不是可变的。

请记住，没有引用时，变量和字段访问的正常规则会发挥作用。意思是如果值没有被借用，它可能会被移动或复制。

```move
module a::m;

public struct X() has copy, drop;
public fun by_val(_: X) {}
public fun by_ref(_: &X) {}
public fun by_mut(_: &mut X) {}

public struct Y has drop { x: X }
public fun drop_y(y: Y) { y }

fun example(y: Y) {
    y.x.by_val(); // 复制 `y.x`，因为 `by_val` 是按值的，且 `X` 有 `copy`
    y.drop_y(); // 移动 `y`，因为 `drop_y` 是按值的，且 `Y` 没有 `copy`
}
```

## 链式调用

方法调用可以链式进行，因为任何表达式都可以是方法的接收者。

```move
module a::shapes {
    public struct Point has copy, drop, store { x: u64, y: u64 }
    public struct Line has copy, drop, store { start: Point, end: Point }

    public fun x(p: &Point): u64 { p.x }
    public fun y(p: &Point): u64 { p.y }

    public fun start(l: &Line): &Point { &l.start }
    public fun end(l: &Line): &Point { &l.end }

}

module b::example {
    use a::shapes::Line;

    public fun x_values(l: Line): (u64, u64) {
        (l.start().x(), l.end().x())
    }

}
```

在这个例子中，对于 `l.start().x()`，编译器首先将 `l.start()` 解析为 `a::shapes::start(&l)`。然后 `.x()` 解析为 `a::shapes::x(a::shapes::start(&l))`。`l.end().x()` 类似。请记住，这个功能不是"特殊的"——`.` 的左侧可以是任何表达式，编译器将正常解析方法调用。我们只是提请注意这种"链式"调用，因为这是增加可读性的常见做法。