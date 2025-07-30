---
title: 'Generics | Reference'
description: ''
---

# 泛型

泛型可用于定义在不同输入数据类型上的函数和结构体。这种语言特性有时被称为参数多态性。在 Move 中，我们通常将术语泛型与*类型参数*和*类型实参*互换使用。

泛型通常用于库代码中，例如在[向量](./primitive-types/vector)中，声明适用于任何可能类型（满足指定约束）的代码。这种参数化允许您在多种类型和情况下重用相同的实现。

## 声明类型参数

函数和结构体都可以在其签名中采用类型参数列表，由一对尖括号 `<...>` 包围。

### 泛型函数

函数的类型参数放在函数名之后、（值）参数列表之前。以下代码定义了一个泛型恒等函数，它接受任何类型的值并返回该值不变。

```move
fun id<T>(x: T): T {
    // 这个类型注释是不必要的但有效的
    (x: T)
}
```

一旦定义，类型参数 `T` 可以在参数类型、返回类型和函数体内使用。

### 泛型结构体

结构体的类型参数放在结构体名之后，可用于命名字段的类型。

```move
public struct Foo<T> has copy, drop { x: T }

public struct Bar<T1, T2> has copy, drop {
    x: T1,
    y: vector<T2>,
}
```

注意[类型参数不必被使用](#unused-type-parameters)

## 类型实参

### 调用泛型函数

调用泛型函数时，可以在由一对尖括号包围的列表中为函数的类型参数指定类型实参。

```move
fun foo() {
    let x = id<bool>(true);
}
```

如果不指定类型实参，Move 的[类型推断](#type-inference)将为您提供它们。

### 使用泛型结构体

类似地，在构造或解构泛型类型的值时，可以为结构体的类型参数附加类型实参列表。

```move
fun foo() {
    // 构造时的类型实参
    let foo = Foo<bool> { x: true };
    let bar = Bar<u64, u8> { x: 0, y: vector<u8>[] };

    // 解构时的类型实参
    let Foo<bool> { x } = foo;
    let Bar<u64, u8> { x, y } = bar;
}
```

在任何情况下，如果不指定类型实参，Move 的[类型推断](#type-inference)将为您提供它们。

### 类型实参不匹配

如果指定类型实参并且它们与提供的实际值冲突，将给出错误：

```move
fun foo() {
    let x = id<u64>(true); // 错误！true 不是 u64
}
```

类似地：

```move
fun foo() {
    let foo = Foo<bool> { x: 0 }; // 错误！0 不是 bool
    let Foo<address> { x } = foo; // 错误！bool 与 address 不兼容
}
```

## 类型推断

在大多数情况下，Move 编译器能够推断类型实参，因此您不必显式写出它们。如果我们省略类型实参，上面的例子看起来像这样：

```move
fun foo() {
    let x = id(true);
    //        ^ <bool> 被推断

    let foo = Foo { x: true };
    //           ^ <bool> 被推断

    let Foo { x } = foo;
    //     ^ <bool> 被推断
}
```

注意：当编译器无法推断类型时，您需要手动注释它们。一个常见的场景是调用类型参数仅出现在返回位置的函数。

```move
module a::m;

fun foo() {
    let v = vector[]; // 错误！
    //            ^ 编译器无法确定元素类型，因为它从未被使用

    let v = vector<u64>[];
    //            ^~~~~ 在这种情况下必须手动注释。
}
```

注意这些情况有点人为，因为 `vector[]` 从未被使用，因此 Move 的类型推断无法推断类型。

但是，如果该值在函数中稍后使用，编译器将能够推断类型：

```move
module a::m;

fun foo() {
    let v = vector[];
    //            ^ <u64> 被推断
    vector::push_back(&mut v, 42);
    //               ^ <u64> 被推断
}
```

### `_` 类型

在某些情况下，您可能希望显式注释某些类型实参，但让编译器推断其他类型实参。`_` 类型用作编译器推断类型的占位符。

```move
let bar = Bar<u64, _> { x: 0, y: vector[b"hello"] };
//                 ^ vector<u8> 被推断
```

占位符 `_` 只能出现在表达式和宏函数定义中，不能出现在签名中。这意味着您不能将 `_` 用作函数参数、函数返回类型、常量定义类型和数据类型字段的定义的一部分。

## 整数

在 Move 中，整数类型 `u8`、`u16`、`u32`、`u64`、`u128` 和 `u256` 都是不同的类型。但是，每种类型都可以使用相同的数值语法创建。换句话说，如果没有提供类型后缀，编译器将根据值的使用情况推断整数类型。

```move
let x8: u8 = 0;
let x16: u16 = 0;
let x32: u32 = 0;
let x64: u64 = 0;
let x128: u128 = 0;
let x256: u256 = 0;
```

如果值未在需要特定整数类型的上下文中使用，则采用 `u64` 作为默认值。

```move
let x = 0;
//      ^ 默认使用 u64
```

但是，如果值对于推断的类型来说太大，将给出错误

```move
let i: u8 = 256; // 错误！
//          ^^^ 对于 u8 来说太大
let x = 340282366920938463463374607431768211454;
//      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ 对于 u64 来说太大
```

在数字太大的情况下，您可能需要显式注释它

```move
let x = 340282366920938463463374607431768211454u128;
//                                             ^^^^ 有效！
```

## 未使用的类型参数

对于结构体定义，未使用的类型参数是指未出现在结构体中定义的任何字段中，但在编译时进行静态检查的参数。Move 允许未使用的类型参数，因此以下结构体定义是有效的：

```move
public struct Foo<T> {
    foo: u64
}
```

这在建模某些概念时很方便。这里是一个例子：

```move
module a::m;

// 货币指定符
public struct A {}
public struct B {}

// 可以使用货币指定符类型实例化的泛型硬币类型。
//   例如 Coin<A>，Coin<B> 等。
public struct Coin<Currency> has store {
    value: u64
}

// 为所有货币编写通用代码
public fun mint_generic<Currency>(value: u64): Coin<Currency> {
    Coin { value }
}

// 为一种货币编写具体代码
public fun mint_a(value: u64): Coin<A> {
    mint_generic(value)
}
public fun mint_b(value: u64): Coin<B> {
    mint_generic(value)
}
```

在这个例子中，`Coin<Currency>` 在 `Currency` 类型参数上是泛型的，它指定硬币的货币，并允许代码以任何货币的通用方式或特定货币的具体方式编写。即使 `Currency` 类型参数没有出现在 `Coin` 中定义的任何字段中，这种通用性仍然适用。

### 幻影类型参数

在上面的例子中，尽管 `struct Coin` 请求 `store` 能力，但 `Coin<A>` 和 `Coin<B>` 都不会拥有 `store` 能力。这是由于[条件能力和泛型类型](./abilities#conditional-abilities-and-generic-types)的规则以及 `A` 和 `B` 没有 `store` 能力的事实，尽管它们甚至没有在 `struct Coin` 的主体中使用。这可能会导致一些不愉快的后果。例如，我们无法将 `Coin<A>` 放入存储中的钱包。

一个可能的解决方案是向 `A` 和 `B` 添加虚假的能力注释（即 `public struct Currency1 has store {}`）。但是，这可能导致错误或安全漏洞，因为它通过不必要的能力声明削弱了类型。例如，我们永远不会期望存储中的值具有类型 `A` 的字段，但使用虚假的 `store` 能力这将是可能的。此外，虚假注释将是传染性的，需要许多在未使用类型参数上泛型的函数也包含必要的约束。

幻影类型参数解决了这个问题。未使用的类型参数可以标记为*幻影*类型参数，它不参与结构体的能力派生。这样，幻影类型参数的实参在派生泛型类型的能力时不被考虑，从而避免了虚假能力注释的需要。为了使这个宽松的规则是健全的，Move 的类型系统保证声明为 `phantom` 的参数要么在结构体定义中根本不使用，要么仅用作也声明为 `phantom` 的类型参数的实参。

#### 声明

在结构体定义中，可以通过在声明前添加 `phantom` 关键字将类型参数声明为幻影。

```move
public struct Coin<phantom Currency> has store {
    value: u64
}
```

如果一个类型参数被声明为幻影，我们说它是一个幻影类型参数。定义结构体时，Move 的类型检查器确保每个幻影类型参数要么没有在结构体定义内部使用，要么仅用作幻影类型参数的实参。

```move
public struct S1<phantom T1, T2> { f: u64 }
//               ^^^^^^^ 有效，T1 未出现在结构体定义内部

public struct S2<phantom T1, T2> { f: S1<T1, T2> }
//               ^^^^^^^ 有效，T1 出现在幻影位置
```

以下代码显示了违反规则的示例：

```move
public struct S1<phantom T> { f: T }
//               ^^^^^^^ 错误！  ^ 不是幻影位置

public struct S2<T> { f: T }
public struct S3<phantom T> { f: S2<T> }
//               ^^^^^^^ 错误！     ^ 不是幻影位置
```

更正式地说，如果一个类型用作幻影类型参数的实参，我们说该类型出现在*幻影位置*。有了这个定义，幻影参数正确使用的规则可以指定如下：**幻影类型参数只能出现在幻影位置**。

注意指定 `phantom` 不是必需的，但如果类型参数可以是 `phantom` 但没有标记为这样的话，编译器会发出警告。

#### 实例化

实例化结构体时，在派生结构体能力时排除幻影参数的实参。例如，考虑以下代码：

```move
public struct S<T1, phantom T2> has copy { f: T1 }
public struct NoCopy {}
public struct HasCopy has copy {}
```

现在考虑类型 `S<HasCopy, NoCopy>`。由于 `S` 定义了 `copy` 且所有非幻影实参都有 `copy`，那么 `S<HasCopy, NoCopy>` 也有 `copy`。

#### 带有能力约束的幻影类型参数

能力约束和幻影类型参数是正交的特性，因为幻影参数可以用能力约束声明。

```move
public struct S<phantom T: copy> {}
```

用能力约束实例化幻影类型参数时，类型实参必须满足该约束，即使参数是幻影的。通常的限制适用，`T` 只能用具有 `copy` 的实参实例化。

## 约束

在上面的例子中，我们已经演示了如何使用类型参数来定义"未知"类型，调用者可以在稍后插入这些类型。但是，这意味着类型系统对类型的信息很少，必须以非常保守的方式执行检查。在某种意义上，类型系统必须为无约束泛型假设最坏情况——没有[能力](./abilities)的类型。

约束提供了一种指定这些未知类型具有哪些属性的方法，以便类型系统可以允许否则不安全的操作。

### 声明约束

可以使用以下语法对类型参数施加约束。

```move
// T 是类型参数的名称
T: <ability> (+ <ability>)*
```

`<ability>` 可以是四种[能力](./abilities)中的任何一种，类型参数可以同时受到多种能力的约束。因此，以下所有内容都是有效的类型参数声明：

```move
T: copy
T: copy + drop
T: copy + drop + store + key
```

### 验证约束

约束在实例化站点进行检查

```move
public struct Foo<T: copy> { x: T }

public struct Bar { x: Foo<u8> }
//                         ^^ 有效，u8 有 `copy`

public struct Baz<T> { x: Foo<T> }
//                            ^ 错误！T 没有 'copy'
```

函数也类似

```move
fun unsafe_consume<T>(x: T) {
    // 错误！x 没有 'drop'
}

fun consume<T: drop>(x: T) {
    // 有效，x 将自动被丢弃
}

public struct NoAbilities {}

fun foo() {
    let r = NoAbilities {};
    consume<NoAbilities>(NoAbilities);
    //      ^^^^^^^^^^^ 错误！NoAbilities 没有 'drop'
}
```

一些类似的例子，但使用 `copy`

```move
fun unsafe_double<T>(x: T) {
    (copy x, x)
    // 错误！T 没有 'copy'
}

fun double<T: copy>(x: T) {
    (copy x, x) // 有效，T 有 'copy'
}

public struct NoAbilities {}

fun foo(): (NoAbilities, NoAbilities) {
    let r = NoAbilities {};
    double<NoAbilities>(r)
    //     ^ 错误！NoAbilities 没有 'copy'
}
```

更多信息，请参见能力部分关于[条件能力和泛型类型](./abilities#conditional-abilities-and-generic-types)。

## 递归限制

### 递归结构体

泛型结构体不能包含相同类型的字段，无论是直接还是间接，即使使用不同的类型实参。以下所有结构体定义都是无效的：

```move
public struct Foo<T> {
    x: Foo<u64> // 错误！'Foo' 包含 'Foo'
}

public struct Bar<T> {
    x: Bar<T> // 错误！'Bar' 包含 'Bar'
}

// 错误！'A' 和 'B' 形成循环，这也是不允许的。
public struct A<T> {
    x: B<T, u64>
}

public struct B<T1, T2> {
    x: A<T1>
    y: A<T2>
}
```

### 高级主题：类型级递归

Move 允许泛型函数被递归调用。但是，当与泛型结构体结合使用时，在某些情况下这可能创建无限数量的类型，允许这样做意味着向编译器、vm 和其他语言组件添加不必要的复杂性。因此，此类递归是被禁止的。

这个限制将来可能会放宽，但现在，以下示例应该让您了解什么是允许的，什么是不允许的。

```move
module a::m;

public struct A<T> {}

// 有限多种类型 -- 允许。
// foo<T> -> foo<T> -> foo<T> -> ... 是有效的
fun foo<T>() {
    foo<T>();
}

// 有限多种类型 -- 允许。
// foo<T> -> foo<A<u64>> -> foo<A<u64>> -> ... 是有效的
fun foo<T>() {
    foo<A<u64>>();
}
```

不允许：

```move
module a::m;

public struct A<T> {}

// 无限多种类型 -- 不允许。
// 错误！
// foo<T> -> foo<A<T>> -> foo<A<A<T>>> -> ...
fun foo<T>() {
    foo<Foo<T>>();
}
```

类似地，不允许：

```move
module a::n;

public struct A<T> {}

// 无限多种类型 -- 不允许。
// 错误！
// foo<T1, T2> -> bar<T2, T1> -> foo<T2, A<T1>>
//   -> bar<A<T1>, T2> -> foo<A<T1>, A<T2>>
//   -> bar<A<T2>, A<T1>> -> foo<A<T2>, A<A<T1>>>
//   -> ...
fun foo<T1, T2>() {
    bar<T2, T1>();
}

fun bar<T1, T2> {
    foo<T1, A<T2>>();
}
```

注意，类型级递归的检查基于对调用站点的保守分析，不考虑控制流或运行时值。

```move
module a::m;

public struct A<T> {}

// 无限多种类型 -- 不允许。
// 错误！
fun foo<T>(n: u64) {
    if (n > 0) foo<A<T>>(n - 1);
}
```

上面示例中的函数在技术上对于任何给定的输入都将终止，因此只创建有限多种类型，但它仍然被 Move 的类型系统认为是无效的。