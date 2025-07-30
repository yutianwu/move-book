# 表达式

在编程语言中，表达式是返回值的代码单元。在 Move 中，几乎所有内容都是表达式，唯一的例外是声明语句的 `let` 语句。在本节中，我们将介绍表达式的类型并引入作用域的概念。

> 表达式用分号 `;` 分隔。如果分号后"没有表达式"，编译器将插入一个 `unit ()`，它表示一个空表达式。

## 字面量

在[基本类型](./primitive-types)章节中，我们介绍了 Move 的基本类型。为了说明它们，我们使用了字面量。字面量是在源代码中表示固定值的记法。字面量可以用于初始化变量或直接将固定值作为参数传递给函数。Move 有以下字面量：

- 布尔值：`true` 和 `false`
- 整数值：`0`、`1`、`123123`
- 十六进制值：以 0x 为前缀的数字来表示整数，如 `0x0`、`0x1`、`0x123`
- 字节向量值：以 `b` 为前缀，如 `b"bytes_vector"`
- 字节值：以 `x` 为前缀的十六进制字面量，如 `x"0A"`

```move file=packages/samples/sources/move-basics/expression.move anchor=literals

```

## 运算符

算术、逻辑和位运算符用于对值执行操作。由于这些操作产生值，它们被视为表达式。

```move file=packages/samples/sources/move-basics/expression.move anchor=operators

```

## 代码块

代码块是用花括号 `{}` 括起来的语句和表达式序列。它返回块中最后一个表达式的值（注意最后一个表达式不能有结尾分号）。代码块是一个表达式，因此可以在任何需要表达式的地方使用。

```move file=packages/samples/sources/move-basics/expression.move anchor=block

```

## 函数调用

我们在[函数](./function)章节中详细介绍了函数。但是，我们在前面的章节中已经使用了函数调用，所以值得在这里提及。函数调用是调用函数并返回函数体中最后一个表达式的值的表达式，前提是最后一个表达式没有结尾分号。

```move file=packages/samples/sources/move-basics/expression.move anchor=fun_call

```

## 控制流表达式

控制流表达式用于控制程序的流程。它们也是表达式，因此返回一个值。我们在[控制流](./control-flow)章节中介绍控制流表达式。这里是一个非常简要的概述：

```move file=packages/samples/sources/move-basics/expression.move anchor=control_flow

```