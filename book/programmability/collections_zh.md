# 集合

集合类型是任何编程语言的基本组成部分。它们用于存储数据集合，例如项目列表。`vector`类型已在[向量部分](./../move-basics/vector)中介绍过，在本章中我们将介绍[Sui框架](./sui-framework)提供的基于向量的集合类型。

## 向量

虽然我们之前在[向量部分](./../move-basics/vector)中介绍了`vector`类型，但在新的上下文中再次介绍它是值得的。这次我们将介绍`vector`类型在对象中的使用以及它如何在应用程序中使用。

```move file=packages/samples/sources/programmability/collections.move anchor=vector

```

## VecSet

`VecSet`是一种存储一组唯一项目的集合类型。它类似于`vector`，但不允许重复项目。这个属性使其对于存储唯一项目的集合很有用，例如ID或地址列表。

```move file=packages/samples/sources/programmability/collections-2.move anchor=vec_set

```

VecSet在尝试插入集合中已存在的项目时会失败。

## VecMap

`VecMap`是一种存储键值对映射的集合类型。它类似于`VecSet`，但允许您将值与集合中的每个项目关联。这使其对于存储键值对的集合很有用，例如地址及其余额的列表，或用户ID及其关联数据的列表。

`VecMap`中的键是唯一的，每个键只能与单个值关联。如果您尝试插入一个键值对，其中键在映射中已存在，旧值将被新值替换。

```move file=packages/samples/sources/programmability/collections-3.move anchor=vec_map

```

## 限制

标准集合类型是存储具有保证安全性和一致性的类型化数据的好方法。但是，它们受到它们可以存储的数据类型的限制 - 类型系统不允许您在集合中存储错误的类型；并且它们的大小受到限制 - 受对象大小限制。它们适用于相对小型的集合和列表，但对于更大的集合，您可能需要使用不同的方法。

集合类型的另一个限制是无法比较它们。由于插入顺序不能保证，尝试将一个`VecSet`与另一个`VecSet`进行比较可能不会产生预期的结果。

> 这种行为被linter捕获并会发出警告：_比较类型为'sui::vec_set::VecSet'的集合可能产生意外结果_

```move file=packages/samples/sources/programmability/collections-4.move anchor=vec_set_comparison

```

在上面的示例中，比较将失败，因为插入顺序不能保证，两个`VecSet`实例可能有不同的元素顺序。即使两个`VecSet`实例包含相同的元素，比较也会失败。

## 总结

- Vector是一种原生类型，允许存储项目列表。
- VecSet建立在vector之上，允许存储唯一项目的集合。
- VecMap用于在类似映射的结构中存储键值对。
- 基于向量的集合是严格类型化的，受对象大小限制限制，最适合小型集合和列表。

## 下一步

在下一节中，我们将介绍[包装器类型模式](./wrapper-type-pattern) - 一种通常与集合类型一起使用的设计模式，用于扩展或限制它们的行为。