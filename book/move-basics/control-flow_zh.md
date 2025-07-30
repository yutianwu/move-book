# 控制流

<!--

Chapter: Basic Syntax
Goal: Introduce control flow statements.
Notes:
    - if/else is an expression
    - while () {} loop
    - continue and break
    - loop {}
    - infinite loop is possible but will lead to gas exhaustion
    - return keyword
    - if is an expression and as such requires a semicolon (!!!)

Links:
    - reference (control flow)
    - coding conventions (control flow)

 -->

控制流语句用于控制程序中执行的流程。它们用于做决定、重复代码块或提早退出代码块。Move 包括以下控制流语句（下面详细解释）：

- [`if` 和 `if-else`](#conditional-statements) - 决定是否执行代码块
- [`loop` 和 `while` 循环](#repeating-statements-with-loops) - 重复代码块
- [`break` 和 `continue` 语句](#exiting-a-loop-early) - 提早退出循环
- [`return`](#early-return) 语句 - 提早退出函数

## 条件语句

`if` 表达式用于在程序中做决定。它计算一个[布尔表达式](./expression#literals)，如果表达式为真，则执行代码块。与 `else` 配对，如果表达式为假，它可以执行不同的代码块。

`if` 表达式的语法是：

```move
if (<bool_expression>) <expression>;
if (<bool_expression>) <expression> else <expression>;
```

就像任何其他表达式一样，如果后面还有其他表达式，`if` 需要分号。`else` 关键字是可选的，除非将结果值赋给变量，因为所有分支都必须返回一个值以确保类型安全。让我们通过以下示例来检查 `if` 表达式在 Move 中如何工作：

```move file=packages/samples/sources/move-basics/control-flow.move anchor=if_condition

```

让我们看看如何使用 `if` 和 `else` 为变量赋值：

```move file=packages/samples/sources/move-basics/control-flow.move anchor=if_else

```

在这个例子中，`if` 表达式的值被赋给变量 `y`。如果 `x` 大于 0，`y` 被赋予值 1；否则，它被赋予 0。需要 `else` 块，因为 `if` 表达式的两个分支都必须返回相同类型的值。省略 `else` 块会导致编译器错误，因为它确保所有可能的分支都被考虑并维持类型安全。

<!-- TODO: add an error -->

条件表达式是 Move 中最重要的控制流语句之一。它们评估用户提供的输入或存储的数据以做出决定。一个关键用例是在 [`assert!` 宏](./assert-and-abort)中，它检查条件是否为真，如果不是则中止执行。我们将在稍后详细探讨这一点。

## 用循环重复语句

循环用于多次执行代码块。Move 有两种内置类型的循环：`loop` 和 `while`。在许多情况下，它们可以互换使用，但通常当迭代次数事先知道时使用 `while`，当迭代次数事先不知道或有多个退出点时使用 `loop`。

循环对于处理集合（如向量）或重复代码块直到满足特定条件很有用。但是，要小心避免无限循环，这可能会耗尽 gas 限制并导致交易中止。

## `while` 循环

`while` 语句只要相关的布尔表达式计算为真，就会重复执行代码块。就像我们在 `if` 中看到的一样，在循环的每次迭代之前都会计算布尔表达式。另外，像条件语句一样，`while` 循环是一个表达式，如果后面还有其他表达式，则需要分号。

`while` 循环的语法是：

```move
while (<bool_expression>) { <expressions>; };
```

这里是一个具有非常简单条件的 `while` 循环示例：

```move file=packages/samples/sources/move-basics/control-flow.move anchor=while_loop

```

## 无限 `loop`

现在让我们想象一个布尔表达式始终为 `true` 的场景。例如，如果我们字面上将 `true` 传递给 `while` 条件。这类似于 `loop` 语句的功能，只是 `while` 计算一个条件。

```move file=packages/samples/sources/move-basics/control-flow.move anchor=infinite_while

```

无限 `while` 循环，或具有始终 `true` 条件的 `while` 循环，等价于 `loop`。创建 `loop` 的语法很简单：

```move
loop { <expressions>; };
```

让我们使用 `loop` 而不是 `while` 来重写前面的示例：

```move file=packages/samples/sources/move-basics/control-flow.move anchor=infinite_loop

```

无限循环在 Move 中很少实用，因为每个操作都消耗 gas，无限循环不可避免地会导致 gas 耗尽。如果你发现自己使用循环，考虑一下是否可能有更好的方法，因为许多用例可以用其他控制流结构更有效地处理。话说回来，当与 `break` 和 `continue` 语句结合使用时，`loop` 可能有用，以创建可控和灵活的循环行为。

## 提早退出循环

正如我们已经提到的，无限循环本身相当无用。这就是我们引入 `break` 和 `continue` 语句的地方。它们分别用于提早退出循环和跳过当前迭代的剩余部分。

`break` 语句的语法是（没有分号）：

```move
break
```

`break` 语句用于停止循环的执行并提早退出。它通常与条件语句结合使用，在满足某个条件时退出循环。为了说明这一点，让我们将前面示例中的无限 `loop` 转换为更像 `while` 循环的东西：

```move file=packages/samples/sources/move-basics/control-flow.move anchor=break_loop

```

几乎与 `while` 循环相同，对吧？当 `x` 为 5 时，`break` 语句用于退出循环。如果我们移除 `break` 语句，循环将永远运行，就像前面的示例一样。

## 跳过迭代

`continue` 语句用于跳过当前迭代的剩余部分并开始下一次迭代。与 `break` 类似，它与条件语句结合使用，在满足某个条件时跳过迭代的剩余部分。

`continue` 语句的语法是（没有分号）：

```move
continue
```

下面的示例跳过奇数，只打印从 0 到 10 的偶数：

```move file=packages/samples/sources/move-basics/control-flow.move anchor=continue_loop

```

`break` 和 `continue` 语句可以在 `while` 和 `loop` 循环中使用。

## 提早返回

`return` 语句用于提早退出[函数](./function)并返回一个值。它通常与条件语句结合使用，在满足某个条件时退出函数。`return` 语句的语法是：

```move
return <expression>
```

这里是一个在满足某个条件时返回值的函数示例：

```move file=packages/samples/sources/move-basics/control-flow.move anchor=return_statement

```

与许多其他语言不同，函数中的最后一个表达式不需要 `return` 语句。函数块中的最后一个表达式会自动返回。但是，当我们想在满足某个条件时提早退出函数时，`return` 语句很有用。

## 进一步阅读

- Move 参考中的[控制流](./../../reference/control-flow)章节。