# 原始类型

<!-- TODO: Shall we split this into two pages? Maybe give an overview and focus more on specifics? -->

对于简单的值，Move 提供了许多内置的原始类型。它们是所有其他类型的基础。原始类型包括：

- [布尔值](#booleans)
- [无符号整数](#integer-types)
- [地址](./address) - 将在下一节中介绍

在深入了解原始类型之前，让我们先看看如何在 Move 中声明和分配变量。

## 变量和赋值

变量使用 `let` 关键字声明。默认情况下它们是不可变的，但可以通过添加 `mut` 关键字使其可变：

```
let <variable_name>[: <type>]  = <expression>;
let mut <variable_name>[: <type>] = <expression>;
```

其中：

- `<variable_name>` - 变量的名称
- `<type>` - 变量的类型，可选
- `<expression>` - 要赋给变量的值

```move file=packages/samples/sources/move-basics/primitive-types.move anchor=variables_and_assignment

```

可变变量可以使用 `=` 操作符重新赋值。

```move
y = 43;
```

变量也可以通过重新声明来进行遮蔽。

```move file=packages/samples/sources/move-basics/primitive-types.move anchor=shadowing

```

## 布尔值

`bool` 类型表示一个布尔值——是或否，真或假。它有两个可能的值：`true` 和 `false`，这些是 Move 中的关键字。对于布尔值，编译器总是可以从值推断类型，所以不需要显式指定。

```move file=packages/samples/sources/move-basics/primitive-types.move anchor=boolean

```

布尔值经常用于存储标志和控制程序流程。更多信息请参考[控制流](./control-flow)部分。

## 整数类型

Move 支持各种大小的无符号整数，从 8 位到 256 位。整数类型包括：

- `u8` - 8 位
- `u16` - 16 位
- `u32` - 32 位
- `u64` - 64 位
- `u128` - 128 位
- `u256` - 256 位

```move file=packages/samples/sources/move-basics/primitive-types.move anchor=integers

```

虽然像 `true` 和 `false` 这样的布尔字面量明确是布尔值，但像 `42` 这样的整数字面量可能是任何整数类型。在大多数情况下，编译器会从值推断类型，通常默认为 `u64`。然而，有时编译器无法推断类型，需要显式的类型注解。它可以在赋值期间提供，也可以使用类型后缀。

```move file=packages/samples/sources/move-basics/primitive-types.move anchor=integer_explicit_type

```

### 操作

Move 支持整数的标准算术操作：加法、减法、乘法、除法和模运算（余数）。这些操作的语法是：

| 语法   | 操作           | 中止条件                      |
| ------ | -------------- | ----------------------------- |
| +      | 加法           | 结果对于整数类型来说太大      |
| -      | 减法           | 结果小于零                    |
| \*     | 乘法           | 结果对于整数类型来说太大      |
| %      | 模运算（余数） | 除数为 0                      |
| /      | 截断除法       | 除数为 0                      |

> 更多操作，包括位运算，请参考
> [Move 参考手册](./../../reference/primitive-types/integers#bitwise)。

操作数的类型_必须匹配_，否则编译器会抛出错误。操作的结果将与操作数的类型相同。要对不同类型执行操作，需要将操作数转换为相同类型。

<!-- TODO: add examples + parentheses for arithmetic operations -->
<!-- TODO: add bitwise operators -->

### 使用 `as` 进行转换

Move 支持在整数类型之间进行显式转换。语法如下：

```move
<expression> as <type>
```

注意，表达式周围可能需要括号来防止歧义：

```move file=packages/samples/sources/move-basics/primitive-types.move anchor=cast_as

```

一个更复杂的例子，防止溢出：

```move file=packages/samples/sources/move-basics/primitive-types.move anchor=overflow

```

### 溢出

Move 不支持溢出/下溢；导致值超出类型范围的操作将引发运行时错误。这是一个安全特性，用于防止意外行为。

```move
let x = 255u8;
let y = 1u8;

// 这将引发错误
let z = x + y;
```

## 进一步阅读

- Move 参考手册中的[布尔](./../../reference/primitive-types/bool)。
- Move 参考手册中的[整数](./../../reference/primitive-types/integers)。