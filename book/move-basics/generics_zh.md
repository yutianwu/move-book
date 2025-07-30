# 泛型

泛型是定义可以与任何类型一起工作的类型或函数的方法。当你想编写可以与不同类型一起使用的函数，或者当你想定义可以容纳任何其他类型的类型时，这很有用。泛型是 Move 中许多高级功能的基础，包括集合、抽象实现等。

## 在标准库中

在本章中，我们已经提到了[向量](./vector)类型，它是一个可以容纳任何其他类型的泛型类型。标准库中泛型类型的另一个例子是 [Option](./option) 类型，它用于表示可能存在也可能不存在的值。

## 泛型语法

要定义泛型类型或函数，类型签名需要有一个用角括号（`<` 和 `>`）括起来的泛型参数列表。泛型参数用逗号分隔。

```move file=packages/samples/sources/move-basics/generics.move anchor=container

```

在上面的例子中，`Container` 是具有单个类型参数 `T` 的泛型类型，容器的 `value` 字段存储 `T`。`new` 函数是具有单个类型参数 `T` 的泛型函数，它返回一个带有给定值的 `Container`。泛型类型必须用具体类型初始化，泛型函数必须用具体类型调用，尽管在某些情况下 Move 编译器可以推断正确的类型。

```move file=packages/samples/sources/move-basics/generics.move anchor=test_container

```

在测试函数 `test_container` 中，我们演示了三种等效的方法来创建一个带有 `u8` 值的新 `Container`。因为数值常量具有模糊的类型，我们必须在某处指定数字字面量的类型（在容器的类型、`new` 的参数或数字字面量本身中）；一旦我们指定其中一个，编译器就可以推断其他的。

## 多个类型参数

你可以定义具有多个类型参数的类型或函数。类型参数用逗号分隔。

```move file=packages/samples/sources/move-basics/generics.move anchor=pair

```

在上面的例子中，`Pair` 是具有两个类型参数 `T` 和 `U` 的泛型类型，`new_pair` 函数是具有两个类型参数 `T` 和 `U` 的泛型函数。函数返回一个带有给定值的 `Pair`。类型参数的顺序很重要，应该与类型签名中类型参数的顺序匹配。

```move file=packages/samples/sources/move-basics/generics.move anchor=test_pair

```

如果我们添加另一个实例，在 `new_pair` 函数中交换类型参数，并尝试比较两种类型，我们会看到类型签名不同，无法比较。

```move file=packages/samples/sources/move-basics/generics.move anchor=test_pair_swap

```

由于 `pair1` 和 `pair2` 的类型不同，比较 `pair1 == pair2` 将不会编译。

## 为什么使用泛型？

在上面的例子中，我们专注于实例化泛型类型和调用泛型函数来创建这些类型的实例。然而，泛型的真正威力在于它们能够为基础的泛型类型定义共享行为，然后独立于具体类型使用它。这在处理集合、抽象实现和 Move 中的其他高级功能时特别有用。

```move file=packages/samples/sources/move-basics/generics.move anchor=user

```

在上面的例子中，`User` 是具有单个类型参数 `T` 的泛型类型，具有共享字段 `name`、`age` 和泛型 `metadata` 字段，可以存储任何类型。无论 `metadata` 是什么，`User` 的所有实例都将包含相同的字段和方法。

```move file=packages/samples/sources/move-basics/generics.move anchor=update_user

```

## 幻影类型参数

在某些情况下，你可能想定义一个泛型类型，其类型参数在类型的字段或方法中未使用。这称为_幻影类型参数_。幻影类型参数在你想定义可以容纳任何其他类型的类型，但想对类型参数强制执行某些约束时很有用。

```move file=packages/samples/sources/move-basics/generics.move anchor=phantom

```

这里的 `Coin` 类型不包含使用类型参数 `T` 的任何字段或方法。它用于区分不同类型的硬币，并对类型参数 `T` 强制执行某些约束。

```move file=packages/samples/sources/move-basics/generics.move anchor=test_phantom

```

在上面的例子中，我们演示了如何创建具有不同幻影类型参数 `USD` 和 `EUR` 的两个不同的 `Coin` 实例。类型参数 `T` 在 `Coin` 类型的字段或方法中未使用，但它用于区分不同类型的硬币。这有助于确保 `USD` 和 `EUR` 硬币不会被错误地混淆。

## 类型参数的约束

类型参数可以被约束为具有某些能力。当你需要内部类型允许某些行为（如 _copy_ 或 _drop_）时，这很有用。约束类型参数的语法是 `T: <ability> + <ability>`。

```move file=packages/samples/sources/move-basics/generics.move anchor=constraints

```

Move 编译器将强制类型参数 `T` 具有指定的能力。如果类型参数没有指定的能力，代码将不会编译。

<!-- TODO: failure case -->

```move file=packages/samples/sources/move-basics/generics.move anchor=test_constraints

```

## 扩展阅读

- Move 参考中的[泛型](./../../reference/generics)。