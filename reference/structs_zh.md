---
title: '结构体 | 参考'
description: ''
---

# 结构体和资源

_结构体_是用户定义的包含类型化字段的数据结构。结构体可以存储任何非引用、非元组类型，包括其他结构体。

结构体可用于定义所有"资产"值或无限制值，其中对这些值执行的操作可以由结构体的[能力](./abilities_zh)控制。默认情况下，结构体是线性的和短暂的。这意味着它们：不能被复制，不能被丢弃，不能存储在存储中。这意味着所有值都必须转移所有权（线性），并且值必须在程序执行结束时处理完毕（短暂）。我们可以通过给结构体[能力](./abilities_zh)来放宽这种行为，这些能力允许值被复制或丢弃，也可以存储在存储中或定义存储模式。

## 定义结构体

结构体必须在模块内部定义，结构体的字段可以是命名的或位置的：

```move
module a::m;

public struct Foo { x: u64, y: bool }
public struct Bar {}
public struct Baz { foo: Foo, }
//                          ^ 注意：可以有尾随逗号

public struct PosFoo(u64, bool)
public struct PosBar()
public struct PosBaz(Foo)
```

结构体不能是递归的，所以以下定义是无效的：

```move
public struct Foo { x: Foo }
//                     ^ 错误！递归定义

public struct A { b: B }
public struct B { a: A }
//                   ^ 错误！递归定义

public struct D(D)
//              ^ 错误！递归定义
```

### 可见性

如您所见，所有结构体都声明为`public`。这意味着结构体的类型可以从任何其他模块引用。但是，结构体的字段以及创建或销毁结构体的能力仍然是定义结构体的模块的内部内容。

将来，我们计划添加将结构体声明为`public(package)`或内部的功能，就像[函数](./functions_zh#visibility)一样。

### 能力

如上所述：默认情况下，结构体声明是线性和短暂的。因此，为了允许值以这些方式使用（例如，复制、丢弃、存储在[对象](./abilities/object_zh)中，或用于定义可存储的[对象](./abilities/object_zh)），结构体可以通过用`has <ability>`注释它们来被授予[能力](./abilities_zh)：

```move
module a::m {
    public struct Foo has copy, drop { x: u64, y: bool }
}
```

能力声明可以出现在结构体字段之前或之后。但是，只能使用其中一种，不能两种都使用。如果在结构体字段之后声明，能力声明必须以分号结尾：

```move
module a::m;

public struct PreNamedAbilities has copy, drop { x: u64, y: bool }
public struct PostNamedAbilities { x: u64, y: bool } has copy, drop;
public struct PostNamedAbilitiesInvalid { x: u64, y: bool } has copy, drop
//                                                                        ^ 错误！缺少分号

public struct NamedInvalidAbilities has copy { x: u64, y: bool } has drop;
//                                                               ^ 错误！重复的能力声明

public struct PrePositionalAbilities has copy, drop (u64, bool)
public struct PostPositionalAbilities (u64, bool) has copy, drop;
public struct PostPositionalAbilitiesInvalid (u64, bool) has copy, drop
//                                                                     ^ 错误！缺少分号
public struct InvalidAbilities has copy (u64, bool) has drop;
//                                                  ^ 错误！重复的能力声明
```

有关更多详细信息，请参阅[注释结构体的能力](./abilities_zh#annotating-structs-and-enums)一节。

### 命名

结构体必须以大写字母`A`到`Z`开头。在第一个字母之后，结构体名称可以包含下划线`_`、字母`a`到`z`、字母`A`到`Z`或数字`0`到`9`。

```move
public struct Foo {}
public struct BAR {}
public struct B_a_z_4_2 {}
public struct P_o_s_Foo()
```

这种以`A`到`Z`开头的命名限制是为了给未来的语言特性留出空间。它可能会在以后被移除，也可能不会。

## 使用结构体

### 创建结构体

结构体类型的值可以通过指示结构体名称，然后为每个字段提供值来创建（或"打包"）。

对于具有命名字段的结构体，字段的顺序不重要，但需要提供字段名称。对于具有位置字段的结构体，字段的顺序必须与结构体定义中字段的顺序匹配，并且必须使用`()`而不是`{}`来包围参数。

```move
module a::m;

public struct Foo has drop { x: u64, y: bool }
public struct Baz has drop { foo: Foo }
public struct Positional(u64, bool) has drop;

fun example() {
    let foo = Foo { x: 0, y: false };
    let baz = Baz { foo: foo };
    // 注意：位置结构体值使用括号创建，并
    // 基于位置而不是名称。
    let pos = Positional(0, false);
    let pos_invalid = Positional(false, 0);
    //                           ^ 错误！字段顺序错误且类型不匹配。
}
```

对于具有命名字段的结构体，如果您有一个与字段同名的局部变量，可以使用以下简写：

```move
let baz = Baz { foo: foo };
// 等价于
let baz = Baz { foo };
```

这有时被称为"字段名称双关语"。

### 通过模式匹配销毁结构体

结构体值可以通过在模式中绑定或分配它们来销毁，使用与构造它们相似的语法。

```move
module a::m;

public struct Foo { x: u64, y: bool }
public struct Bar(Foo)
public struct Baz {}
public struct Qux()

fun example_destroy_foo() {
    let foo = Foo { x: 3, y: false };
    let Foo { x, y: foo_y } = foo;
    //        ^ `x: x`的简写

    // 两个新绑定
    //   x: u64 = 3
    //   foo_y: bool = false
}

fun example_destroy_foo_wildcard() {
    let foo = Foo { x: 3, y: false };
    let Foo { x, y: _ } = foo;

    // 只有一个新绑定，因为y绑定到通配符
    //   x: u64 = 3
}

fun example_destroy_foo_assignment() {
    let x: u64;
    let y: bool;
    Foo { x, y } = Foo { x: 3, y: false };

    // 变更现有变量x和y
    //   x = 3, y = false
}

fun example_foo_ref() {
    let foo = Foo { x: 3, y: false };
    let Foo { x, y } = &foo;

    // 两个新绑定
    //   x: &u64
    //   y: &bool
}

fun example_foo_ref_mut() {
    let foo = Foo { x: 3, y: false };
    let Foo { x, y } = &mut foo;

    // 两个新绑定
    //   x: &mut u64
    //   y: &mut bool
}

fun example_destroy_bar() {
    let bar = Bar(Foo { x: 3, y: false });
    let Bar(Foo { x, y }) = bar;
    //            ^ 嵌套模式

    // 两个新绑定
    //   x: u64 = 3
    //   y: bool = false
}

fun example_destroy_baz() {
    let baz = Baz {};
    let Baz {} = baz;
}

fun example_destroy_qux() {
    let qux = Qux();
    let Qux() = qux;
}
```

### 访问结构体字段

结构体的字段可以使用点操作符`.`访问。

对于具有命名字段的结构体，字段可以通过它们的名称访问：

```move
public struct Foo { x: u64, y: bool }
let foo = Foo { x: 3, y: true };
let x = foo.x;  // x == 3
let y = foo.y;  // y == true
```

对于位置结构体，字段可以通过它们在结构体定义中的位置访问：

```move
public struct PosFoo(u64, bool)
let pos_foo = PosFoo(3, true);
let x = pos_foo.0;  // x == 3
let y = pos_foo.1;  // y == true
```

访问结构体字段而不借用或复制它们受字段能力约束的限制。有关更多详细信息，请参阅[借用结构体和字段](#borrowing-structs-and-fields)和[读取和写入字段](#reading-and-writing-fields)部分。

### 借用结构体和字段

`&`和`&mut`操作符可用于创建对结构体或字段的引用。这些示例包括一些可选的类型注释（例如，`: &Foo`）来演示操作的类型。

```move
let foo = Foo { x: 3, y: true };
let foo_ref: &Foo = &foo;
let y: bool = foo_ref.y;         // 通过结构体的引用读取字段
let x_ref: &u64 = &foo.x;        // 通过扩展结构体的引用来借用字段

let x_ref_mut: &mut u64 = &mut foo.x;
*x_ref_mut = 42;            // 通过可变引用修改字段
```

可以借用嵌套结构体的内部字段：

```move
let foo = Foo { x: 3, y: true };
let bar = Bar(foo);

let x_ref = &bar.0.x;
```

您也可以通过结构体的引用借用字段：

```move
let foo = Foo { x: 3, y: true };
let foo_ref = &foo;
let x_ref = &foo_ref.x;
// 这与let x_ref = &foo.x具有相同的效果
```

### 读取和写入字段

如果您需要读取和复制字段的值，可以解引用借用的字段：

```move
let foo = Foo { x: 3, y: true };
let bar = Bar(copy foo);
let x: u64 = *&foo.x;
let y: bool = *&foo.y;
let foo2: Foo = *&bar.0;
```

更标准的方法是，点操作符可用于读取结构体的字段而无需任何借用。与[解引用](./primitive-types/references_zh#reading-and-writing-through-references)一样，字段类型必须具有`copy`[能力](./abilities_zh)。

```move
let foo = Foo { x: 3, y: true };
let x = foo.x;  // x == 3
let y = foo.y;  // y == true
```

点操作符可以链接以访问嵌套字段：

```move
let bar = Bar(Foo { x: 3, y: true });
let x = baz.0.x; // x = 3;
```

但是，对于包含非原始类型的字段（如向量或另一个结构体），这是不被允许的：

```move
let foo = Foo { x: 3, y: true };
let bar = Bar(foo);
let foo2: Foo = *&bar.0;
let foo3: Foo = bar.0; // 错误！必须添加显式复制*&
```

我们可以可变借用结构体的字段来为其分配新值：

```move
let mut foo = Foo { x: 3, y: true };
*&mut foo.x = 42;     // foo = Foo { x: 42, y: true }
*&mut foo.y = !foo.y; // foo = Foo { x: 42, y: false }
let mut bar = Bar(foo);               // bar = Bar(Foo { x: 42, y: false })
*&mut bar.0.x = 52;                   // bar = Bar(Foo { x: 52, y: false })
*&mut bar.0 = Foo { x: 62, y: true }; // bar = Bar(Foo { x: 62, y: true })
```

与解引用类似，我们可以直接使用点操作符来修改字段。在这两种情况下，字段类型必须具有`drop`[能力](./abilities_zh)。

```move
let mut foo = Foo { x: 3, y: true };
foo.x = 42;     // foo = Foo { x: 42, y: true }
foo.y = !foo.y; // foo = Foo { x: 42, y: false }
let mut bar = Bar(foo);         // bar = Bar(Foo { x: 42, y: false })
bar.0.x = 52;                   // bar = Bar(Foo { x: 52, y: false })
bar.0 = Foo { x: 62, y: true }; // bar = Bar(Foo { x: 62, y: true })
```

赋值的点语法也通过结构体的引用工作：

```move
let mut foo = Foo { x: 3, y: true };
let foo_ref = &mut foo;
foo_ref.x = foo_ref.x + 1;
```

## 特权结构体操作

对结构体类型`T`的大多数结构体操作只能在声明`T`的模块内执行：

- 结构体类型只能在定义结构体的模块内创建（"打包"）、销毁（"解包"）。
- 结构体的字段只能在定义结构体的模块内访问。

遵循这些规则，如果您想在模块外修改您的结构体，您需要为它们提供公共API。本章末尾包含一些这方面的示例。

但是如[上面的可见性部分](#visibility)所述，结构体_类型_对另一个模块总是可见的

```move
module a::m {
    public struct Foo has drop { x: u64 }

    public fun new_foo(): Foo {
        Foo { x: 42 }
    }
}

module a::n {
    use a::m::Foo;

    public struct Wrapper has drop {
        foo: Foo
        //   ^ 有效，类型是公共的

    }

    fun f1(foo: Foo) {
        let x = foo.x;
        //      ^ 错误！不能在`a::m`外访问`Foo`的字段
    }

    fun f2() {
        let foo_wrapper = Wrapper { foo: a::m::new_foo() };
        //                               ^ 有效，函数是公共的
    }
}

```

## 所有权

如上面[定义结构体](#defining-structs)中提到的，结构体默认是线性和短暂的。这意味着它们不能被复制或丢弃。当建模真实世界资产如金钱时，这个属性非常有用，因为您不希望金钱被复制或在流通中丢失。

```move
module a::m;

public struct Foo { x: u64 }

public fun copying() {
    let foo = Foo { x: 100 };
    let foo_copy = copy foo; // 错误！'copy'需要'copy'能力
    let foo_ref = &foo;
    let another_copy = *foo_ref // 错误！解引用需要'copy'能力
}

public fun destroying_1() {
    let foo = Foo { x: 100 };

    // 错误！当函数返回时，foo仍然包含值。
    // 这种销毁需要'drop'能力
}

public fun destroying_2(f: &mut Foo) {
    *f = Foo { x: 100 } // 错误！
                        // 通过写入销毁旧值需要'drop'能力
}
```

要修复示例`fun destroying_1`，您需要手动"解包"值：

```move
module a::m;

public struct Foo { x: u64 }

public fun destroying_1_fixed() {
    let foo = Foo { x: 100 };
    let Foo { x: _ } = foo;
}
```

回想一下，您只能在定义结构体的模块内解构结构体。这可以用来在系统中强制执行某些不变量，例如，金钱的守恒。

另一方面，如果您的结构体不代表有价值的东西，您可以添加`copy`和`drop`能力来获得一个结构体值，这可能感觉更像其他编程语言：

```move
module a::m;

public struct Foo has copy, drop { x: u64 }

public fun run() {
    let foo = Foo { x: 100 };
    let foo_copy = foo;
    //             ^ 这段代码复制foo，
    //             而`let x = move foo`会移动foo

    let x = foo.x;            // x = 100
    let x_copy = foo_copy.x;  // x = 100

    // 当函数返回时，foo和foo_copy都被隐式丢弃
}
```

## 存储

结构体可用于定义存储模式，但详细信息因Move的部署而异。有关更多详细信息，请参阅[`key`能力](./abilities_zh#key)和[Sui对象](./abilities/object_zh)的文档。