---
title: '宏函数 | 参考'
description: ''
---

# 宏函数

宏函数是一种定义函数的方式，在编译时在每个调用位置展开。宏的参数不像普通函数那样急切求值，而是通过表达式替换。此外，调用者可以通过[lambda](#lambdas)向宏提供代码。

这些表达式替换机制使`macro`函数类似于[其他编程语言中的宏](<https://en.wikipedia.org/wiki/Macro_(computer_science)>)；但是，它们在Move中比您在其他语言中期望的更受限制。`macro`函数的参数和返回值仍然是类型化的——尽管这可以通过[`_`类型](./../generics_zh#_-type)部分放松。然而，这种限制的好处是，`macro`函数可以在普通函数可以使用的任何地方使用，这对[方法语法](./../method-syntax_zh)特别有用。

未来可能会有更广泛的[语法宏](<https://en.wikipedia.org/wiki/Macro_(computer_science)#Syntactic_macros>)系统。

## 语法

`macro`函数具有与普通函数相似的语法。但是，所有类型参数名称和所有参数名称都必须以`$`开头。注意`_`仍然可以单独使用，但不能作为前缀，必须使用`$_`代替。

```text
<visibility>? macro fun <identifier><[$type_parameters: constraint],*>([$identifier: type],*): <return_type> <function_body>
```

例如，以下`macro`函数接受一个向量和一个lambda，并将lambda应用于向量的每个元素以构造一个新向量。

```move
macro fun map<$T, $U>($v: vector<$T>, $f: |$T| -> $U): vector<$U> {
    let mut v = $v;
    v.reverse();
    let mut i = 0;
    let mut result = vector[];
    while (!v.is_empty()) {
        result.push_back($f(v.pop_back()));
        i = i + 1;
    };
    result
}
```

`$`用来表示参数（类型和值参数）的行为与它们的普通非宏对应物不同。对于类型参数，它们可以用任何类型（甚至引用类型`&`或`&mut`）实例化，并且它们将满足任何约束。类似地，对于参数，它们不会被急切求值，而是在每次使用时替换参数表达式。

## Lambda

Lambda是一种新的表达式类型，只能与`macro`一起使用。这些用于将代码从调用者传递到`macro`的主体中。虽然替换在编译时完成，但它们的使用类似于其他语言中的[匿名函数](https://en.wikipedia.org/wiki/Anonymous_function)、[lambda](https://en.wikipedia.org/wiki/Lambda_calculus)或[闭包](<https://en.wikipedia.org/wiki/Closure_(computer_programming)>)。

如上面的示例所示（`$f: |$T| -> $U`），lambda类型的定义语法为

```text
|<type>,*| (-> <type>)?
```

一些示例

```move
|u64, u64| -> u128 // 接受两个u64并返回u128的lambda
|&mut vector<u8>| -> &mut u8 // 接受&mut vector<u8>并返回&mut u8的lambda
```

如果没有注释返回类型，默认为单元`()`。

```move
// 以下是等价的
|&mut vector<u8>, u64|
|&mut vector<u8>, u64| -> ()
```

然后在`macro`的调用点使用以下语法定义Lambda表达式

```text
|(<identifier> (: <type>)?),*| <expression>
|(<identifier> (: <type>)?),*| -> <type> { <expression> }
```

注意如果注释了返回类型，lambda的主体必须用`{}`包围。

使用上面定义的`map`宏

```move
let v = vector[1, 2, 3];
let doubled: vector<u64> = map!(v, |x| 2 * x);
let bytes: vector<vector<u8>> = map!(v, |x| std::bcs::to_bytes(&x));
```

带类型注解

```move
let doubled: vector<u64> = map!(v, |x: u64| 2 * x); // 返回类型注解可选
let bytes: vector<vector<u8>> = map!(v, |x: u64| -> vector<u8> { std::bcs::to_bytes(&x) });
```

### 捕获

Lambda表达式也可以引用定义lambda的作用域中的变量。这有时称为"捕获"。

```move
let res = foo();
let incremented = map!(vector[1, 2, 3], |x| x + res);
```

任何变量都可以被捕获，包括可变和不可变引用。

有关更复杂的用法，请参阅[示例](#iterating-over-a-vector)部分。

### 限制

目前，lambda只能直接在`macro`函数的调用中使用。它们不能绑定到变量。例如，以下代码将产生错误：

```move
let f = |x| 2 * x;
//      ^^^^^^^^^ 错误！Lambda必须直接在'macro'调用中使用
let doubled: vector<u64> = map!(vector[1, 2, 3], f);
```

## 类型

像普通函数一样，`macro`函数是类型化的——参数和返回值的类型必须被注解。但是，函数的主体在宏展开之前不会进行类型检查。这意味着给定宏的并非所有用法都可能是有效的。例如

```move
macro fun add_one<$T>($x: $T): $T {
    $x + 1
}
```

如果`$T`不是原始整数类型，上面的宏将不会通过类型检查。

这与[方法语法](./../method-syntax_zh)结合使用时特别有用，函数在宏展开后才解析。

```move
macro fun call_foo<$T, $U>($x: $T): &$U {
    $x.foo()
}
```

只有当`$T`有一个返回引用`&$U`的方法`foo`时，这个宏才会成功展开。如[卫生性](#hygiene)部分所述，`foo`将基于定义`call_foo`的作用域解析——而不是展开的地方。

### 类型参数

类型参数可以用任何类型实例化，包括引用类型`&`和`&mut`。它们也可以用[元组类型](./../primitive-types/tuples_zh)实例化，尽管目前的实用性有限，因为元组不能绑定到变量。

这种放松强制类型参数的约束在调用点以通常不会发生的方式得到满足。但是，通常建议为类型参数添加所有必要的约束。例如

```move
public struct NoAbilities()
public struct CopyBox<T: copy> has copy, drop { value: T }
macro fun make_box<$T>($x: $T): CopyBox<$T> {
    CopyBox { value: $x }
}
```

只有当`$T`用具有`copy`能力的类型实例化时，这个宏才会展开。

```move
make_box!(1); // 有效！
make_box!(NoAbilities()); // 错误！'NoAbilities'没有copy能力
```

建议的`make_box`声明是为类型参数添加`copy`约束。这样就向调用者传达了类型必须具有`copy`能力。

```move
macro fun make_box<$T: copy>($x: $T): CopyBox<$T> {
    CopyBox { value: $x }
}
```

那么您可能合理地问，如果建议不使用这种放松，为什么要有这种放松？类型参数的约束在所有情况下都无法强制执行，因为主体在展开之前不会被检查。在以下示例中，`$T`上的`copy`约束在签名中不是必需的，但在主体中是必需的。

```move
macro fun read_ref<$T>($r: &$T): $T {
    *$r
}
```

但是，如果您想要一个极其宽松的类型签名，建议使用[`_`类型](#_-type)。

### `_`类型

通常，[`_`占位符类型](./../generics_zh#_-type)在表达式中用于允许类型参数的部分注解。但是，对于`macro`函数，`_`类型可以用来代替类型参数，为任何类型放松签名。这应该增加声明"泛型"`macro`函数的人体工程学。

例如，我们可以接受任何整数组合并将它们相加。

```move
macro fun add($x: _, $y: _, $z: _): u256 {
    ($x as u256) + ($y as u256) + ($z as u256)
}
```

此外，`_`类型可以用不同的类型_多次_实例化。例如

```move
public struct Box<T> has copy, drop, store { value: T }
macro fun create_two($f: |_| -> Box<_>): (Box<u8>, Box<u16>) {
    ($f(0u8), $f(0u16))
}
```

如果我们用类型参数声明函数，类型必须统一为公共类型，在这种情况下是不可能的。

```move
macro fun create_two<$T>($f: |$T| -> Box<$T>): (Box<u8>, Box<u16>) {
    ($f(0u8), $f(0u16))
    //           ^^^^ 错误！期望`u8`但发现`u16`
}
...
let (a, b) = create_two!(|value| Box { value });
```

在这种情况下，`$T`必须用单一类型实例化，但推断发现`$T`必须绑定到`u8`和`u16`。

但是有一个权衡，因为`_`类型对调用者传达的意义和意图较少。考虑上面重新声明的`map`宏，使用`_`而不是`$T`和`$U`。

```move
macro fun map($v: vector<_>, $f: |_| -> _): vector<_> {
```

在类型级别不再有`$f`行为的指示。调用者必须从注释或宏的主体中获得理解。

## 展开和替换

`macro`的主体在编译时被替换到调用点。每个参数都被其参数的_表达式_（而不是值）替换。对于lambda，可以在`macro`主体的上下文中绑定额外的局部变量值。

以一个非常简单的示例

```move
macro fun apply($f: |u64| -> u64, $x: u64): u64 {
    $f($x)
}
```

调用点

```move
let incremented = apply!(|x| x + 1, 5);
```

这将大致展开为

```move
let incremented = {
    let x = { 5 };
    { x + 1 }
};
```

同样，替换的不是`x`的值，而是表达式`5`。这可能意味着参数被多次求值，或者根本不求值，取决于`macro`的主体。

```move
macro fun dup($f: |u64, u64| -> u64, $x: u64): u64 {
    $f($x, $x)
}
```

```move
let sum = dup!(|x, y| x + y, foo());
```

展开为

```move
let sum = {
    let x = { foo() };
    let y = { foo() };
    { x + y }
};
```

注意`foo()`将被调用两次。如果`dup`是普通函数则不会发生这种情况。

通常建议通过将参数绑定到局部变量来创建可预测的求值行为。

```move
macro fun dup($f: |u64, u64| -> u64, $x: u64): u64 {
    let a = $x;
    $f(a, a)
}
```

现在同一个调用点将展开为

```move
let sum = {
    let a = { foo() };
    {
        let x = { a };
        let y = { a };
        { x + y }
    }
};
```

### 卫生性

在上面的示例中，`dup`宏有一个局部变量`a`，用于绑定参数`$x`。您可能会问，如果变量命名为`x`会发生什么？这会与lambda中的`x`冲突吗？

简短的答案是，不会。`macro`函数是[卫生的](https://en.wikipedia.org/wiki/Hygienic_macro)，这意味着`macro`和lambda的展开不会意外地从另一个作用域捕获变量。

编译器通过为每个作用域关联一个唯一数字来做到这一点。当`macro`展开时，宏主体获得自己的作用域。此外，参数在每次使用时都重新作用域化。

修改`dup`宏以使用`x`而不是`a`

```move
macro fun dup($f: |u64, u64| -> u64, $x: u64): u64 {
    let a = $x;
    $f(a, a)
}
```

调用点的展开

```move
// let sum = dup!(|x, y| x + y, foo());
let sum = {
    let x#1 = { foo() };
    {
        let x#2 = { x#1 };
        let y#2 = { x#1 };
        { x#2 + y#2 }
    }
};
```

这是编译器内部表示的近似，为了这个示例的简单性省略了一些细节。

每次使用参数都会重新作用域化，以便不同的使用不会冲突。

```move
macro fun apply_twice($f: |u64| -> u64, $x: u64): u64 {
    $f($x) + $f($x)
}
```

```move
let result = apply_twice!(|x| x + 1, { let x = 5; x });
```

展开为

```move
let result = {
    {
        let x#1 = { let x#2 = { 5 }; x#2 };
        { x#1 + x#1 }
    }
    +
    {
        let x#3 = { let x#4 = { 5 }; x#4 };
        { x#3 + x#3 }
    }
};
```

类似于变量卫生性，[方法解析](./../method-syntax_zh)也作用域化到宏定义。例如

```move
public struct S { f: u64, g: u64 }

fun f(s: &S): u64 {
    s.f
}
fun g(s: &S): u64 {
    s.g
}

use fun f as foo;
macro fun call_foo($s: &S): u64 {
    let s = $s;
    s.foo()
}
```

在这种情况下，方法调用`foo`将始终解析为函数`f`，即使`call_foo`在`foo`绑定到不同函数（如`g`）的作用域中使用。

```move
fun example(s: &S): u64 {
    use fun g as foo;
    call_foo!(s) // 展开为'f(s)'，不是'g(s)'
}
```

因此，具有`macro`函数的模块中未使用的`use fun`声明可能不会收到警告。

### 控制流

类似于变量卫生性，控制流结构也始终作用域化到它们定义的地方，而不是展开的地方。

```move
macro fun maybe_div($x: u64, $y: u64): u64 {
    let x = $x;
    let y = $y;
    if (y == 0) return 0;
    x / y
}
```

在调用点，`return`将始终从`macro`主体返回，而不是从调用者返回。

```move
let result: vector<u64> = vector[maybe_div!(10, 0)];
```

将展开为

```move
let result: vector<u64> = vector['a: {
    let x = { 10 };
    let y = { 0 };
    if (y == 0) return 'a 0;
    x / y
}];
```

其中`return 'a 0`将返回到块`'a: { ... }`而不是调用者的主体。有关更多详细信息，请参阅[标记控制流](./../control-flow/labeled-control-flow_zh)部分。

类似地，lambda中的`return`将从lambda返回，而不是从`macro`主体返回，也不是从外部函数返回。

```move
macro fun apply($f: |u64| -> u64, $x: u64): u64 {
    $f($x)
}
```

和

```move
let result = apply!(|x| { if (x == 0) return 0; x + 1 }, 100);
```

将展开为

```move
let result = {
    let x = { 100 };
    'a: {
        if (x == 0) return 'a 0;
        x + 1
    }
};
```

除了从lambda返回，还可以使用标签返回到外部函数。在`vector::any`宏中，使用带标签的`return`提前从整个`macro`返回

```move
public macro fun any<$T>($v: &vector<$T>, $f: |&$T| -> bool): bool {
    let v = $v;
    'any: {
        v.do_ref!(|e| if ($f(e)) return 'any true);
        false
    }
}
```

当条件满足时，`return 'any true`提前退出"循环"。否则，宏"返回"`false`。

### 方法语法

当适用时，`macro`函数可以使用[方法语法](./../method-syntax_zh)调用。使用方法语法时，参数的求值将发生变化，第一个参数（方法的"接收器"）将在宏展开之外求值。这个示例是人为的，但将简洁地演示行为。

```move
public struct S() has copy, drop;
public fun foo(): S { abort 0 }
public macro fun maybe_s($s: S, $cond: bool): S {
    if ($cond) $s
    else S()
}
```

即使`foo()`会中止，其返回类型可以用来开始方法调用。

如果`$cond`为`false`，`$s`将不会被求值，在正常的非方法调用下，`foo()`参数不会被求值且不会中止。以下示例演示了带有`foo()`参数的`$s`不被求值。

```move
maybe_s!(foo(), false) // 不会中止
```

当查看展开形式时，为什么它不会中止变得更清楚

```move
if (false) foo()
else S()
```

但是，使用方法语法时，第一个参数在宏展开之前被求值。所以`$s`的相同`foo()`参数现在将被求值并将中止。

```move
foo().maybe_s!(false) // 中止
```

当查看展开形式时我们可以更清楚地看到这一点

```move
let tmp = foo(); // 中止
if (false) tmp
else S()
```

从概念上讲，方法调用的接收器在宏展开之前绑定到临时变量，这强制求值从而中止。

### 参数限制

`macro`函数的参数必须始终用作表达式。它们不能在可能重新解释参数的情况下使用。例如，以下是不允许的

```move
macro fun no($x: _): _ {
    $x.f
}
```

原因是如果参数`$x`不是引用，它会先被借用，这可能会重新解释参数。要绕过此限制，您应该将参数绑定到局部变量。

```move
macro fun yes($x: _): _ {
    let x = $x;
    x.f
}
```

## 示例

### 懒惰参数：assert_eq

```move
macro fun assert_eq<$T>($left: $T, $right: $T, $code: u64) {
    let left = $left;
    let right = $right;
    if (left != right) {
        std::debug::print(&b"assertion failed.\n left: ");
        std::debug::print(&left);
        std::debug::print(&b"\n does not equal right: ");
        std::debug::print(&right);
        abort $code;
    }
}
```

在这种情况下，`$code`的参数只有在断言失败时才会被求值。

```move
assert_eq!(vector[true, false], vector[true, false], 1 / 0); // 除零不会被求值
```

### 任意整数平方根

这个宏计算除`u256`之外任何整数类型的整数平方根。

`$T`是输入的类型，`$bitsize`是该类型的位数，例如`u8`有8位。`$U`应该设置为下一个更大的整数类型，例如`u8`对应`u16`。

在这个`macro`中，整数字面量`1`和`0`的类型被注解，例如`(1: $U)`，允许字面量的类型在每次调用时不同。类似地，`as`可以与类型参数`$T`和`$U`一起使用。只有当`$T`和`$U`用整数类型实例化时，这个宏才会成功展开。

```move
macro fun num_sqrt<$T, $U>($x: $T, $bitsize: u8): $T {
    let x = $x;
    let mut bit = (1: $U) << $bitsize;
    let mut res = (0: $U);
    let mut x = x as $U;

    while (bit != 0) {
        if (x >= res + bit) {
            x = x - (res + bit);
            res = (res >> 1) + bit;
        } else {
            res = res >> 1;
        };
        bit = bit >> 2;
    };

    res as $T
}
```

### 遍历向量

这两个`macro`分别不可变地和可变地遍历向量。

```move
macro fun for_imm<$T>($v: &vector<$T>, $f: |&$T|) {
    let v = $v;
    let n = v.length();
    let mut i = 0;
    while (i < n) {
        $f(&v[i]);
        i = i + 1;
    }
}

macro fun for_mut<$T>($v: &mut vector<$T>, $f: |&mut $T|) {
    let v = $v;
    let n = v.length();
    let mut i = 0;
    while (i < n) {
        $f(&mut v[i]);
        i = i + 1;
    }
}
```

一些使用示例

```move
fun imm_examples(v: &vector<u64>) {
    // 打印所有元素
    for_imm!(v, |x| std::debug::print(x));

    // 求和所有元素
    let mut sum = 0;
    for_imm!(v, |x| sum = sum + x);

    // 找到最大元素
    let mut max = 0;
    for_imm!(v, |x| if (x > max) max = x);
}

fun mut_examples(v: &mut vector<u64>) {
    // 增加每个元素
    for_mut!(v, |x| *x = *x + 1);

    // 将每个元素设置为前一个值，第一个设置为最后一个值
    let mut prev = v[v.length() - 1];
    for_mut!(v, |x| {
        let tmp = *x;
        *x = prev;
        prev = tmp;
    });

    // 将最大元素设置为0
    let mut max = &mut 0;
    for_mut!(v, |x| if (*x > *max) max = x);
    *max = 0;
}
```

### 非循环lambda用法

Lambda不需要在循环中使用，通常对条件性应用代码很有用。

```move
macro fun inspect<$T>($opt: &Option<$T>, $f: |&$T|) {
    let opt = $opt;
    if (opt.is_some()) $f(opt.borrow())
}

macro fun is_some_and<$T>($opt: &Option<$T>, $f: |&$T| -> bool): bool {
    let opt = $opt;
    if (opt.is_some()) $f(opt.borrow())
    else false
}

macro fun map<$T, $U>($opt: Option<$T>, $f: |$T| -> $U): Option<$U> {
    let opt = $opt;
    if (opt.is_some()) {
        option::some($f(opt.destroy_some()))
    } else {
        opt.destroy_none();
        option::none()
    }
}
```

一些使用示例

```move
fun examples(opt: Option<u64>) {
    // 如果值存在则打印
    inspect!(&opt, |x| std::debug::print(x));

    // 检查值是否为0
    let is_zero = is_some_and!(&opt, |x| *x == 0);

    // 将u64上转换为u256
    let str_opt = map!(opt, |x| x as u256);
}
```