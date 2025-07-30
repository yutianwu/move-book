---
title: '局部变量和作用域 | 参考'
description: ''
---

# 局部变量和作用域

Move中的局部变量是词法（静态）作用域的。新变量通过关键字`let`引入，它将遮蔽任何具有相同名称的先前局部变量。标记为`mut`的局部变量是可变的，可以直接更新，也可以通过可变引用更新。

## 声明局部变量

### `let`绑定

Move程序使用`let`将变量名绑定到值：

```move
let x = 1;
let y = x + x;
```

`let`也可以在不将值绑定到局部变量的情况下使用。

```move
let x;
```

然后可以稍后为局部变量分配值。

```move
let x;
if (cond) {
  x = 1
} else {
  x = 0
}
```

当试图从循环中提取值而无法提供默认值时，这非常有用。

```move
let x;
let mut i = 0;
loop {
    let (res, cond) = foo(i);
    if (!cond) {
        x = res;
        break
    };
    i = i + 1;
}
```

要在分配_后_修改局部变量，或可变借用它`&mut`，必须将其声明为`mut`。

```move
let mut x = 0;
if (cond) x = x + 1;
foo(&mut x);
```

有关更多详细信息，请参阅下面的[赋值](#assignments)部分。

### 变量必须在使用前赋值

Move的类型系统防止局部变量在被赋值之前使用。

```move
let x;
// highlight-error
x + x // 错误！x在被赋值前使用
```

```move
let x;
if (cond) x = 0;
// highlight-error
x + x // 错误！x在所有情况下都没有值
```

```move
let x;
while (cond) x = 0;
// highlight-error
x + x // 错误！x在所有情况下都没有值
```

### 有效的变量名

变量名可以包含下划线`_`、字母`a`到`z`、字母`A`到`Z`和数字`0`到`9`。变量名必须以下划线`_`或字母`a`到`z`开头。它们_不能_以大写字母开头。

```move
// 全部有效
let x = e;
let _x = e;
let _A = e;
let x0 = e;
let xA = e;
let foobar_123 = e;

// 全部无效
// highlight-error-start
let X = e; // 错误！
let Foo = e; // 错误！
// highlight-error-end
```

### 类型注解

局部变量的类型几乎总是可以被Move的类型系统推断出来。但是，Move允许显式类型注解，这对于可读性、清晰性或可调试性很有用。添加类型注解的语法是：

```move
let x: T = e; // "类型T的变量x初始化为表达式e"
```

显式类型注解的一些示例：

```move
module 0::example;

public struct S { f: u64, g: u64 }

fun annotated() {
    let u: u8 = 0;
    let b: vector<u8> = b"hello";
    let a: address = @0x0;
    let (x, y): (&u64, &mut u64) = (&0, &mut 1);
    let S { f, g: f2 }: S = S { f: 0, g: 1 };
}
```

注意类型注解必须始终在模式的右侧：

```move
// highlight-error-start
// 错误！应该是let (x, y): (&u64, &mut u64) = ...
let (x: &u64, y: &mut u64) = (&0, &mut 1);
// highlight-error-end
```

### 何时需要注解

在某些情况下，如果类型系统无法推断类型，则需要局部类型注解。这通常发生在无法推断泛型类型的类型参数时。例如：

```move
// highlight-error-start
let _v1 = vector[]; // 错误！
//        ^^^^^^^^ 无法推断此类型。尝试添加注解
// highlight-error-end
let v2: vector<u64> = vector[]; // 没有错误
```

在更罕见的情况下，类型系统可能无法为发散代码（其中所有后续代码都不可达）推断类型。[`return`](./functions_zh#return-expression)和[`abort`](./abort-and-assert_zh)都是表达式，可以有任何类型。[`loop`](./control-flow/loops_zh)如果有`break`则类型为`()`（或者如果有`break e`其中`e: T`则为`T`），但如果没有从`loop`中跳出，它可以有任何类型。如果无法推断这些类型，则需要类型注解。例如，此代码：

```move
let a: u8 = return ();
let b: bool = abort 0;
let c: signer = loop ();

// highlight-error-start
let x = return (); // 错误！
//  ^ 无法推断此类型。尝试添加注解
let y = abort 0; // 错误！
//  ^ 无法推断此类型。尝试添加注解
let z = loop (); // 错误！
//  ^ 无法推断此类型。尝试添加注解
// highlight-error-end
```

为此代码添加类型注解将暴露关于死代码或未使用局部变量的其他错误，但该示例仍然有助于理解这个问题。

### 使用元组的多个声明

`let`可以使用元组一次引入多个局部变量。在括号内声明的局部变量被初始化为元组中相应的值。

```move
let () = ();
let (x0, x1) = (0, 1);
let (y0, y1, y2) = (0, 1, 2);
let (z0, z1, z2, z3) = (0, 1, 2, 3);
```

表达式的类型必须与元组模式的元数完全匹配。

```move
// highlight-error
let (x, y) = (0, 1, 2); // 错误！
// highlight-error
let (x, y, z, q) = (0, 1, 2); // 错误！
```

您不能在单个`let`中声明多个具有相同名称的局部变量。

```move
// highlight-error
let (x, x) = 0; // 错误！
```

声明的局部变量的可变性可以混合。

```move
let (mut x, y) = (0, 1);
x = 1;
```

### 使用结构体的多个声明

当解构（或匹配）结构体时，`let`也可以一次引入多个局部变量。在这种形式中，`let`创建一组局部变量，这些变量被初始化为结构体字段的值。语法如下：

```move
public struct T { f1: u64, f2: u64 }
```

```move
let T { f1: local1, f2: local2 } = T { f1: 1, f2: 2 };
// local1: u64
// local2: u64
```

对于位置结构体也是如此

```move
public struct P(u64, u64)
```

和

```move
let P (local1, local2) = P ( 1, 2 );
// local1: u64
// local2: u64
```

这是一个更复杂的示例：

```move
module 0::example;

public struct X(u64)
public struct Y { x1: X, x2: X }

fun new_x(): X {
    X(1)
}

fun example() {
    let Y { x1: X(f), x2 } = Y { x1: new_x(), x2: new_x() };
    assert!(f + x2.0 == 2, 42);

    let Y { x1: X(f1), x2: X(f2) } = Y { x1: new_x(), x2: new_x() };
    assert!(f1 + f2 == 2, 42);

    // `struct X`没有`drop`能力，需要手动销毁
    let X(_) = x2;
}
```

结构体的字段可以起双重作用，既标识要绑定的字段_又_标识变量的名称。这有时被称为双关语。

```move
let Y { x1, x2 } = e;
```

等价于：

```move
let Y { x1: x1, x2: x2 } = e;
```

如元组所示，您不能在单个`let`中声明多个具有相同名称的局部变量。

```move
// highlight-error
let Y { x1: x, x2: x } = e; // 错误！
```

与元组一样，声明的局部变量的可变性可以混合。

```move
let Y { x1: mut x1, x2 } = e;
```

此外，可变性注解可以应用于双关语字段。给出等价的示例

```move
let Y { mut x1, x2 } = e;
```

### 对引用的解构

在上面结构体的示例中，let中绑定的值被移动，销毁结构体值并绑定其字段。

```move
public struct T { f1: u64, f2: u64 }
```

```move
let T { f1: local1, f2: local2 } = T { f1: 1, f2: 2 };
// local1: u64
// local2: u64
```

在这种情况下，结构体值`T { f1: 1, f2: 2 }`在`let`之后不再存在。

如果您希望不移动和销毁结构体值，可以借用其每个字段。例如：

```move
let t = T { f1: 1, f2: 2 };
let T { f1: local1, f2: local2 } = &t;
// local1: &u64
// local2: &u64
```

对于可变引用也是如此：

```move
let mut t = T { f1: 1, f2: 2 };
let T { f1: local1, f2: local2 } = &mut t;
// local1: &mut u64
// local2: &mut u64
```

这种行为也适用于嵌套结构体。

```move
module 0::example;

public struct X(u64)
public struct Y { x1: X, x2: X }

fun new_x(): X {
    X(1)
}

fun example() {
    let mut y = Y { x1: new_x(), x2: new_x() };

    let Y { x1: X(f), x2 } = &y;
    assert!(*f + x2.0 == 2, 42);

    let Y { x1: X(f1), x2: X(f2) } = &mut y;
    *f1 = *f1 + 1;
    *f2 = *f2 + 1;
    assert!(*f1 + *f2 == 4, 42);

    // `struct X和struct Y`没有`drop`能力，需要手动销毁
    let Y { x1: X(_), x2: X(_) } = y;
}
```

### 忽略值

在`let`绑定中，忽略某些值通常很有用。以`_`开头的局部变量将被忽略且不引入新变量

```move
fun three(): (u64, u64, u64) {
    (0, 1, 2)
}
```

```move
let (x1, _, z1) = three();
let (x2, _y, z2) = three();
assert!(x1 + z1 == x2 + z2, 42);
```

这有时是必需的，因为编译器会对未使用的局部变量发出警告

```move
let (x1, y, z1) = three(); // 警告！
//       ^ 未使用的局部变量'y'
```

### 通用`let`语法

`let`中的所有不同结构都可以组合！这样我们得到了`let`语句的通用语法：

> _let-binding_ → **let** _pattern-or-list_ _type-annotation_<sub>_opt_</sub> >
> _initializer_<sub>_opt_</sub> > _pattern-or-list_ → _pattern_ | **(** _pattern-list_ **)** >
> _pattern-list_ → _pattern_ **,**<sub>_opt_</sub> | _pattern_ **,** _pattern-list_ >
> _type-annotation_ → **:** _type_ _initializer_ → **=** _expression_

引入绑定的项目的通用术语是_模式_。模式用于解构数据（可能递归地）并引入绑定。模式语法如下：

> _pattern_ -> _local-variable_ | _struct-type_ **\{** _field-binding-list_ **\}** >
> _field-binding-list_ → _field-binding_ **,**<sub>_opt_</sub> | _field-binding_ **,** >
> _field-binding-list_ > _field-binding_ → _field_ | _field_ **:** _pattern_

应用此语法的几个具体示例：

```move
    let (x, y): (u64, u64) = (0, 1);
//       ^                           local-variable
//       ^                           pattern
//          ^                        local-variable
//          ^                        pattern
//          ^                        pattern-list
//       ^^^^                        pattern-list
//      ^^^^^^                       pattern-or-list
//            ^^^^^^^^^^^^           type-annotation
//                         ^^^^^^^^  initializer
//  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ let-binding

    let Foo { f, g: x } = Foo { f: 0, g: 1 };
//      ^^^                                    struct-type
//            ^                                field
//            ^                                field-binding
//               ^                             field
//                  ^                          local-variable
//                  ^                          pattern
//               ^^^^                          field-binding
//            ^^^^^^^                          field-binding-list
//      ^^^^^^^^^^^^^^^                        pattern
//      ^^^^^^^^^^^^^^^                        pattern-or-list
//                      ^^^^^^^^^^^^^^^^^^^^   initializer
//  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ let-binding
```

## 变更

### 赋值

在引入局部变量之后（通过`let`或作为函数参数），可以通过赋值修改`mut`局部变量：

```move
x = e
```

与`let`绑定不同，赋值是表达式。在某些语言中，赋值返回被赋值的值，但在Move中，任何赋值的类型始终是`()`。

```move
(x = e: ())
```

实际上，赋值是表达式意味着它们可以在不添加带有大括号的新表达式块（`{`...`}`）的情况下使用。

```move
let x;
if (cond) x = 1 else x = 2;
```

赋值使用与`let`绑定类似的模式语法方案，但没有`mut`：

```move
module 0::example;

public struct X { f: u64 }

fun new_x(): X {
    X { f: 1 }
}

// 注意：此示例将抱怨未使用的变量和赋值。
fun example() {
    let (mut x, mut y, mut f, mut g) = (0, 0, 0, 0);

    (X { f }, X { f: x }) = (new_x(), new_x());
    assert!(f + x == 2, 42);

    (x, y, f, _, g) = (0, 0, 0, 0, 0);
}
```

注意局部变量只能有一种类型，因此局部变量的类型不能在赋值之间改变。

```move
let mut x;
x = 0;
// highlight-error
x = false; // 错误！
```

### 通过引用变更

除了直接通过赋值修改局部变量外，还可以通过可变引用`&mut`修改`mut`局部变量。

```move
let mut x = 0;
let r = &mut x;
*r = 1;
assert!(x == 1, 42);
```

这在以下情况下特别有用：

(1) 您想根据某些条件修改不同的变量。

```move
let mut x = 0;
let mut y = 1;
let r = if (cond) &mut x else &mut y;
*r = *r + 1;
```

(2) 您希望另一个函数修改您的局部值。

```move
let mut x = 0;
modify_ref(&mut x);
```

这种修改方式是您修改结构体和向量的方式！

```move
let mut v = vector[];
vector::push_back(&mut v, 100);
assert!(*vector::borrow(&v, 0) == 100, 42);
```

有关更多详细信息，请参阅[Move引用](./primitive-types/references_zh)。

## 作用域

用`let`声明的任何局部变量都可用于任何后续表达式，_在该作用域内_。作用域用表达式块`{`...`}`声明。

局部变量不能在声明的作用域之外使用。

```move
let x = 0;
{
    let y = 1;
};
// highlight-error-start
x + y // 错误！
//  ^ 未绑定的局部变量'y'
// highlight-error-end
```

但是，来自外部作用域的局部变量_可以_在嵌套作用域中使用。

```move
{
    let x = 0;
    {
        let y = x + 1; // 有效
    }
}
```

局部变量可以在任何可访问的作用域中变更。该变更与局部变量一起存在，无论执行变更的作用域如何。

```move
let mut x = 0;
x = x + 1;
assert!(x == 1, 42);
{
    x = x + 1;
    assert!(x == 2, 42);
};
assert!(x == 2, 42);
```

### 表达式块

表达式块是由分号（`;`）分隔的一系列语句。表达式块的结果值是块中最后一个表达式的值。

```move
{ let x = 1; let y = 1; x + y }
```

在此示例中，块的结果是`x + y`。

语句可以是`let`声明或表达式。记住赋值（`x = e`）是类型为`()`的表达式。

```move
{ let x; let y = 1; x = 1; x + y }
```

函数调用是另一种类型为`()`的常见表达式。修改数据的函数调用通常用作语句。

```move
{ let v = vector[]; vector::push_back(&mut v, 1); v }
```

这不仅限于`()`类型---任何表达式都可以在序列中用作语句！

```move
{
    let x = 0;
    x + 1; // 值被丢弃
    x + 2; // 值被丢弃
    b"hello"; // 值被丢弃
}
```

但是！如果表达式包含资源（没有`drop` [能力](./abilities_zh)的值），您将得到错误。这是因为Move的类型系统保证任何被丢弃的值都具有`drop` [能力](./abilities_zh)。（所有权必须转移或值必须在其声明模块内显式销毁。）

```move
{
    let x = 0;
// highlight-error-start
    Coin { value: x }; // 错误！
//  ^^^^^^^^^^^^^^^^^ 没有`drop`能力的未使用值
// highlight-error-end
    x
}
```

如果块中没有最终表达式---也就是说，如果有尾随分号`;`，则有一个隐式[单元`()`值](https://en.wikipedia.org/wiki/Unit_type)。类似地，如果表达式块为空，则有一个隐式单元`()`值。

两者都是等价的

```move
{ x = x + 1; 1 / x; }
```

```move
{ x = x + 1; 1 / x; () }
```

类似地，两者都是等价的

```move
{ }
```

```move
{ () }
```

表达式块本身是一个表达式，可以在使用表达式的任何地方使用。（注意：函数的主体也是表达式块，但函数主体不能被另一个表达式替换。）

```move
let my_vector: vector<vector<u8>> = {
    let mut v = vector[];
    vector::push_back(&mut v, b"hello");
    vector::push_back(&mut v, b"goodbye");
    v
};
```

（此示例中不需要类型注解，仅为清楚起见而添加。）

### 遮蔽

如果`let`引入的局部变量名称已经在作用域中，则该先前变量在此作用域的其余部分无法再访问。这称为_遮蔽_。

```move
let x = 0;
assert!(x == 0, 42);

let x = 1; // x被遮蔽
assert!(x == 1, 42);
```

当局部变量被遮蔽时，它不需要保持与之前相同的类型。

```move
let x = 0;
assert!(x == 0, 42);

let x = b"hello"; // x被遮蔽
assert!(x == b"hello", 42);
```

局部变量被遮蔽后，存储在局部变量中的值仍然存在，但将不再可访问。对于没有[`drop`能力](./abilities_zh)的类型的值，这一点很重要，因为值的所有权必须在函数结束时转移。

```move
module 0::example;

public struct Coin has store { value: u64 }

fun unused_coin(): Coin {
// highlight-error-start
    let x = Coin { value: 0 }; // 错误！
//      ^ 这个局部变量仍然包含没有`drop`能力的值
    x.value = 1;
    let x = Coin { value: 10 };
    x
//  ^ 无效返回
// highlight-error-end
}
```

当局部变量在作用域内被遮蔽时，遮蔽仅对该作用域保持。一旦该作用域结束，遮蔽就消失了。

```move
let x = 0;
{
    let x = 1;
    assert!(x == 1, 42);
};
assert!(x == 0, 42);
```

请记住，局部变量在被遮蔽时可以改变类型。

```move
let x = 0;
{
    let x = b"hello";
    assert!(x == b"hello", 42);
};
assert!(x == 0, 42);
```

## Move和Copy

Move中的所有局部变量都可以通过两种方式使用：`move`或`copy`。如果没有指定其中一种，Move编译器能够推断应该使用`copy`还是`move`。这意味着在上面的所有示例中，编译器会插入`move`或`copy`。不能在不使用`move`或`copy`的情况下使用局部变量。

来自其他编程语言，`copy`可能会感觉最熟悉，因为它创建变量内值的新副本以在该表达式中使用。使用`copy`，局部变量可以多次使用。

```move
let x = 0;
let y = copy x + 1;
let z = copy x + 2;
```

任何具有`copy` [能力](./abilities_zh)的值都可以以这种方式复制，并且除非指定`move`，否则将隐式复制。

`move`从局部变量中取出值_而不_复制数据。发生`move`后，局部变量不可用，即使值的类型具有`copy` [能力](./abilities_zh)。

```move
let x = 1;
// highlight-error-start
let y = move x + 1;
//      ------ 局部变量在这里被移动
let z = move x + 2; // 错误！
//      ^^^^^^ 局部变量'x'的无效使用
// highlight-error-end
y + z
```

### 安全性

Move的类型系统将防止值在被移动后使用。这与[`let`声明](#let-bindings)中描述的安全检查相同，后者防止局部变量在被分配值之前使用。

<!-- 有关更多信息，请参阅TODO关于所有权和移动语义的未来部分。 -->

### 推断

如上所述，如果没有指示，Move编译器将推断`copy`或`move`。这样做的算法非常简单：

- 任何具有`copy` [能力](./abilities_zh)的值都被给予`copy`。
- 任何引用（可变`&mut`和不可变`&`）都被给予`copy`。
  - 除非在特殊情况下为了可预测的借用检查器错误而使其成为`move`。一旦引用不再被使用，这就会发生。
- 任何其他值都被给予`move`。

给定结构体

```move
public struct Foo has copy, drop, store { f: u64 }
public struct Coin has store { value: u64 }
```

我们有以下示例

```move
let s = b"hello";
let foo = Foo { f: 0 };
let coin = Coin { value: 0 };
let coins = vector[Coin { value: 0 }, Coin { value: 0 }];

let s2 = s; // copy
let foo2 = foo; // copy
let coin2 = coin; // move
let coins2 = coins; // move

let x = 0;
let b = false;
let addr = @0x42;
let x_ref = &x;
let coin_ref = &mut coin2;

let x2 = x; // copy
let b2 = b; // copy
let addr2 = @0x42; // copy
let x_ref2 = x_ref; // copy
let coin_ref2 = coin_ref; // copy
```