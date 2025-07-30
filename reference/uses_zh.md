---
title: 'Uses和别名 | 参考'
description: ''
---

# Uses和别名

`use`语法可以用来为其他模块中的成员创建别名。`use`可以用来创建在整个模块中或在给定表达式块作用域中持续的别名。

## 语法

`use`有几种不同的语法情况。从最简单的开始，我们有以下为其他模块创建别名的语法

```move
use <address>::<module name>;
use <address>::<module name> as <module alias name>;
```

例如

```move
use std::vector;
use std::option as o;
```

`use std::vector;`为`std::vector`引入了一个别名`vector`。这意味着在您想要使用模块名称`std::vector`的任何地方（假设此`use`在作用域内），您可以使用`vector`代替。`use std::vector;`等价于`use std::vector as vector;`

类似地，`use std::option as o;`让您可以使用`o`代替`std::option`

```move
use std::vector;
use std::option as o;

fun new_vec(): vector<o::Option<u8>> {
    let mut v = vector[];
    vector::push_back(&mut v, o::some(0));
    vector::push_back(&mut v, o::none());
    v
}
```

如果您想导入特定的模块成员（如函数或结构体），您可以使用以下语法。

```move
use <address>::<module name>::<module member>;
use <address>::<module name>::<module member> as <member alias>;
```

例如

```move
use std::vector::push_back;
use std::option::some as s;
```

这将让您使用函数`std::vector::push_back`而无需完全限定。类似地，对于`std::option::some`使用`s`。相反，您可以分别使用`push_back`和`s`。同样，`use std::vector::push_back;`等价于`use std::vector::push_back as push_back;`

```move
use std::vector::push_back;
use std::option::some as s;

fun new_vec(): vector<std::option::Option<u8>> {
    let mut v = vector[];
    vector::push_back(&mut v, s(0));
    vector::push_back(&mut v, std::option::none());
    v
}
```

### 多个别名

如果您想一次为多个模块成员添加别名，可以使用以下语法

```move
use <address>::<module name>::{<module member>, <module member> as <member alias> ... };
```

例如

```move
use std::vector::push_back;
use std::option::{some as s, none as n};

fun new_vec(): vector<std::option::Option<u8>> {
    let mut v = vector[];
    push_back(&mut v, s(0));
    push_back(&mut v, n());
    v
}
```

### Self别名

如果您需要为模块本身以及模块成员添加别名，可以在单个`use`中使用`Self`。`Self`是一种指向模块的成员。

```move
use std::option::{Self, some, none};
```

为了清楚起见，以下都是等价的：

```move
use std::option;
use std::option as option;
use std::option::Self;
use std::option::Self as option;
use std::option::{Self};
use std::option::{Self as option};
```

### 同一定义的多个别名

如果需要，您可以为任何项目拥有任意数量的别名

```move
use std::vector::push_back;
use std::option::{Option, some, none};

fun new_vec(): vector<Option<u8>> {
    let mut v = vector[];
    push_back(&mut v, some(0));
    push_back(&mut v, none());
    v
}
```

### 嵌套导入

在Move中，您还可以使用同一个`use`声明导入多个名称。这将所有提供的名称带入作用域：

```move
use std::{
    vector::{Self as vec, push_back},
    string::{String, Self as str}
};

fun example(s: &mut String) {
    let mut v = vec::empty();
    push_back(&mut v, 0);
    push_back(&mut v, 10);
    str::append_utf8(s, v);
}
```

## 在`module`内部

在`module`内部，所有`use`声明都可用，无论声明顺序如何。

```move
module a::example;

use std::vector;

fun new_vec(): vector<Option<u8>> {
    let mut v = vector[];
    vector::push_back(&mut v, 0);
    vector::push_back(&mut v, 10);
    v
}

use std::option::{Option, some, none};
```

模块中`use`声明的别名在该模块内可用。

此外，引入的别名不能与其他模块成员冲突。有关更多详细信息，请参阅[唯一性](#uniqueness)

## 在表达式内部

您可以在任何表达式块的开头添加`use`声明

```move
module a::example;

fun new_vec(): vector<Option<u8>> {
    use std::vector::push_back;
    use std::option::{Option, some, none};

    let mut v = vector[];
    push_back(&mut v, some(0));
    push_back(&mut v, none());
    v
}
```

与`let`一样，表达式块中`use`引入的别名在该块结束时被移除。

```move
module a::example;

fun new_vec(): vector<Option<u8>> {
    let result = {
        use std::vector::push_back;
        use std::option::{Option, some, none};

        let mut v = vector[];
        push_back(&mut v, some(0));
        push_back(&mut v, none());
        v
    };
    result
}
```

尝试在块结束后使用别名将导致错误

```move
fun new_vec(): vector<Option<u8>> {
    let mut result = {
        use std::vector::push_back;
        use std::option::{Option, some, none};

        let mut v = vector[];
        push_back(&mut v, some(0));
        v
    };
    push_back(&mut result, std::option::none());
    // ^^^^^^ 错误！未绑定的函数'push_back'
    result
}
```

任何`use`都必须是块中的第一项。如果`use`出现在任何表达式或`let`之后，将导致解析错误

```move
{
    let mut v = vector[];
    use std::vector; // 错误！
}
```

这允许您在许多情况下缩短导入块。请注意，这些导入与之前的导入一样，都受以下部分描述的命名和唯一性规则的约束。

## 命名规则

别名必须遵循与其他模块成员相同的规则。这意味着结构体（和常量）的别名必须以`A`到`Z`开头

```move
module a::data {
    public struct S {}
    const FLAG: bool = false;
    public fun foo() {}
}
module a::example {
    use a::data::{
        S as s, // 错误！
        FLAG as fLAG, // 错误！
        foo as FOO,  // 有效
        foo as bar, // 有效
    };
}
```

## 唯一性

在给定的作用域内，所有由`use`声明引入的别名必须是唯一的。

对于模块，这意味着`use`引入的别名不能重叠

```move
module a::example;

use std::option::{none as foo, some as foo}; // 错误！
//                                     ^^^ 重复的'foo'

use std::option::none as bar;

use std::option::some as bar; // 错误！
//                       ^^^ 重复的'bar'
```

并且，它们不能与模块的任何其他成员重叠

```move
module a::data {
    public struct S {}
}

module example {
    use a::data::S;

    public struct S { value: u64 } // 错误！
    //            ^ 与上面的别名'S'冲突
}
```

在表达式块内部，它们不能彼此重叠，但它们可以[遮蔽](#shadowing)来自外部作用域的其他别名或名称

## 遮蔽

表达式块内的`use`别名可以遮蔽来自外部作用域的名称（模块成员或别名）。与局部变量的遮蔽一样，遮蔽在表达式块结束时结束；

```move
module a::example;

public struct WrappedVector { vec: vector<u64> }

public fun empty(): WrappedVector {
    WrappedVector { vec: std::vector::empty() }
}

public fun push_back(v: &mut WrappedVector, value: u64) {
    std::vector::push_back(&mut v.vec, value);
}

fun example1(): WrappedVector {
    use std::vector::push_back;
    // 'push_back'现在指向std::vector::push_back
    let mut vec = vector[];
    push_back(&mut vec, 0);
    push_back(&mut vec, 1);
    push_back(&mut vec, 10);
    WrappedVector { vec }
}

fun example2(): WrappedVector {
    let vec = {
        use std::vector::push_back;
        // 'push_back'现在指向std::vector::push_back

        let mut v = vector[];
        push_back(&mut v, 0);
        push_back(&mut v, 1);
        v
    };
    // 'push_back'现在指向Self::push_back
    let mut res = WrappedVector { vec };
    push_back(&mut res, 10);
    res
}
```

## 未使用的Use或别名

未使用的`use`会导致警告

```move
module a::example;

use std::option::{some, none}; // 警告！
//                      ^^^^ 未使用的别名'none'

public fun example(): std::option::Option<u8> {
    some(0)
}
```