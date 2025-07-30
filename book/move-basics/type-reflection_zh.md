# 类型反射

在编程语言中，_反射_是程序检查和修改其自身结构和行为的能力。Move 有一种有限形式的反射，允许您在运行时检查值的类型。当您需要在同构集合中存储类型信息，或者当您需要检查类型是否属于某个包时，这很有用。

类型反射在[标准库](./standard-library)模块 [`std::type_name`][type-name-stdlib] 中实现。简单来说，它提供了一个函数 `get<T>()`，返回类型 `T` 的名称。

## 实践中的应用

该模块很简单，对结果允许的操作仅限于获取字符串表示以及提取类型的模块和地址。

```move file=packages/samples/sources/move-basics/type-reflection.move anchor=main

```

## 进一步阅读

- [std::type_name][type-name-stdlib] 模块文档。

[type-name-stdlib]: https://docs.sui.io/references/framework/std/type_name