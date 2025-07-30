# 向量

向量是在 Move 中存储元素集合的原生方式。它们类似于其他编程语言中的数组，但有一些不同之处。在本节中，我们介绍 `vector` 类型及其操作。

## 向量语法

`vector` 类型使用 `vector` 关键字编写，后跟尖括号中元素的类型。元素的类型可以是任何有效的 Move 类型，包括其他向量。

Move 有一个向量字面量语法，允许您使用 `vector` 关键字后跟包含元素的方括号来创建向量（或者对于空向量不包含元素）。

```move file=packages/samples/sources/move-basics/vector.move anchor=literal

```

`vector` 类型是 Move 中的内置类型，不需要从模块导入。向量操作在 `std::vector` 模块中定义，该模块被隐式导入，可以直接使用而无需显式的 `use` 导入。

## 向量操作

标准库提供操作向量的方法。以下是一些最常用的操作：

- `push_back`：在向量末尾添加一个元素。
- `pop_back`：从向量中移除最后一个元素。
- `length`：返回向量中元素的数量。
- `is_empty`：如果向量为空则返回 true。
- `remove`：移除给定索引处的元素。

```move file=packages/samples/sources/move-basics/vector.move anchor=methods

```

## 销毁不可丢弃类型的向量

不可丢弃类型的向量不能被丢弃。如果您定义了没有 `drop` 能力的类型的向量，向量值不能被忽略。如果向量为空，编译器需要显式调用 `destroy_empty` 函数。

```move file=packages/samples/sources/move-basics/vector.move anchor=no_drop

```

如果您对非空向量调用 `destroy_empty` 函数，该函数将在运行时失败。

## 进一步阅读

- Move 参考手册中的[向量](./../../reference/primitive-types/vector)。
- [std::vector](https://docs.sui.io/references/framework/std/vector) 模块文档。