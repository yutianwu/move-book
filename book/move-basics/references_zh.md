# 引用

<!--

Chapter: Basic Syntax
Goal: Show what the borrow checker is and how it works.
Notes:
    - give the metro pass example
    - show why passing by reference is useful
    - mention that reference comparison is faster
    - references can be both mutable and immutable
    - immutable access to shared objects is faster
    - implicit copy
    - moving the value
    - unpacking a reference (mutable and immutable)

 -->

在[所有权和作用域](./ownership-and-scope)部分，我们解释了当一个值被传递给函数时，它会被_移动_到函数的作用域中。这意味着函数成为该值的所有者，原始作用域（所有者）不再能够使用它。这是 Move 中的一个重要概念，因为它确保了值不会在多个地方同时被使用。然而，在一些用例中，我们希望将值传递给函数但保留所有权。这就是引用发挥作用的地方。

为了说明这一点，让我们考虑一个简单的例子——地铁（地下铁）卡的应用程序。我们将看看卡可能的 4 种不同场景：

1. 在售票机以固定价格购买
2. 向检票员出示以证明乘客持有有效卡
3. 在闸机处使用以进入地铁并购买行程
4. 在卡用完后回收

## 布局

地铁卡应用程序的初始布局很简单。我们定义了 `Card` 类型和 `USES` [常量](./constants)，它表示单张卡上的行程数。我们还添加了[错误常量](./assert-and-abort#error-constants)，用于卡为空和卡不为空的情况。

```move file=packages/samples/sources/move-basics/references.move anchor=header_new
module book::metro_pass;


```

<!-- In [the previous section](./ownership-and-scope) we explained the ownership and scope in Move. We showed how the value is *moved* to a new scope, and how it changes the owner. In this section, we will explain how to *borrow* a reference to a value to avoid moving it, and how Move's *borrow checker* ensures that the references are used correctly. -->

## 引用

引用是一种在不放弃所有权的情况下向函数_展示_值的方式。在我们的例子中，当我们向检票员出示卡片时，我们不想放弃对它的所有权，也不允许检票员使用我们的任何行程。我们只是想允许_读取_我们卡片的值并证明其所有权。

为了做到这一点，在函数签名中，我们使用 `&` 符号来表示我们传递的是值的_引用_，而不是值本身。

```move file=packages/samples/sources/move-basics/references.move anchor=immutable

```

因为函数不获取卡片的所有权，它可以_读取_其数据但不能_写入_，这意味着它不能修改行程数。此外，函数签名确保在没有卡片实例的情况下无法调用它。这是一个重要的属性，它支持[能力模式](./../programmability/capability)，我们将在下一章中介绍。

创建一个值的引用通常被称为"借用"该值。例如，获取 `Option` 包装的值的引用的方法叫做 `borrow`。

## 可变引用

在某些情况下，我们希望允许函数修改卡片。例如，在闸机处使用卡片时，我们需要扣除一次行程。为了实现这一点，我们在函数签名中使用 `&mut` 关键字。

```move file=packages/samples/sources/move-basics/references.move anchor=mutable

```

如您在函数体中所见，`&mut` 引用允许修改值，函数可以消费行程。

## 按值传递

最后，让我们说明当我们将值本身传递给函数时会发生什么。在这种情况下，函数获取值的所有权，使其在原始作用域中无法访问。卡片的所有者可以回收它，从而将所有权让渡给函数。

```move file=packages/samples/sources/move-basics/references.move anchor=move

```

在 `recycle` 函数中，卡片按值传递，将所有权转移给函数。这允许它被解包和销毁。

> 注意：在 Move 中，`_` 是一个用于解构的通配符模式，用来忽略字段但仍然消费值。解构必须匹配结构体类型中的所有字段。如果结构体有字段，您必须明确列出所有字段或使用 `_` 来忽略不需要的字段。

## 完整示例

为了说明应用程序的完整流程，让我们将所有部分放在一个测试中组合起来。

```move file=packages/samples/sources/move-basics/references.move anchor=move_2024

```

## 进一步阅读

- Move 参考手册中的[引用](/reference/primitive-types/references)。

<!-- ## Dereference and Copy -->

<!-- TODO: defer and copy, *& -->

<!-- ## Notes -->

<!--
    Move 2024 is great but it's better to show the example with explicit &t and &mut t
    ...and then say that the example could be rewritten with the new syntax


-->

<!-- ## Move 2024

Here's the test from this page written with the Move 2024 syntax:

```move file=packages/samples/sources/move-basics/references.move anchor=move_2024
```
-->