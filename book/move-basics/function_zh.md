# 函数

函数是 Move 程序的构建块。它们从[用户交易](./../concepts/what-is-a-transaction)和其他函数中调用，将可执行代码分组为可重用的单元。函数可以接受参数并返回值。它们在模块级别用 `fun` 关键字声明。就像任何其他模块成员一样，默认情况下它们是私有的，只能从模块内部访问。

```move file=packages/samples/sources/move-basics/function.move anchor=math

```

在这个例子中，我们定义了一个 `add` 函数，它接受两个 `u64` 类型的参数并返回它们的和。位于同一模块中的 `test_add` 函数是一个调用 `add` 的测试函数。测试使用 `assert!` 宏来比较 `add` 的结果与期望值。如果 `assert!` 中的条件评估为 false，执行将自动中止。

## 函数声明

> 在 Move 中，函数通常使用 `snake_case` 约定命名。这意味着函数名应该全部小写，单词之间用下划线分隔。例如 `do_something`、`add`、`get_balance`、`is_authorized` 等。

函数用 `fun` 关键字声明，后跟函数名（有效的 Move 标识符）、括号中的参数列表和返回类型。函数体是包含语句和表达式序列的代码块。函数体中的最后一个表达式是函数的返回值。

```move file=packages/samples/sources/move-basics/function.move anchor=return_nothing

```

## 访问函数

就像其他模块成员一样，函数可以通过路径导入和访问。路径由模块路径和函数名组成，用 `::` 分隔。例如，如果你在 `book` 包的 `math` 模块中有一个名为 `add` 的函数，它的完整路径将是 `book::math::add`。如果模块已经被导入，你可以直接访问它作为 `math::add`，如下例所示：

```move file=packages/samples/sources/move-basics/function_use.move anchor=use_math

```

## 多返回值

Move 函数可以返回多个值，当你需要从函数返回多个数据时这特别有用。返回类型指定为类型元组，返回值作为表达式元组提供：

```move file=packages/samples/sources/move-basics/function.move anchor=tuple_return

```

具有元组返回的函数调用结果必须通过 `let (tuple)` 语法解包到变量中：

```move file=packages/samples/sources/move-basics/function.move anchor=tuple_return_imm

```

如果任何声明的值需要声明为可变的，`mut` 关键字放在变量名之前：

```move file=packages/samples/sources/move-basics/function.move anchor=tuple_return_mut

```

如果某些参数未使用，可以用 `_` 符号忽略：

```move file=packages/samples/sources/move-basics/function.move anchor=tuple_return_ignore

```

## 扩展阅读

- Move 参考中的[函数](./../../reference/functions)。