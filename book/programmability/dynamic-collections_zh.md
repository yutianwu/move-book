# 动态集合

[Sui框架](./sui-framework)提供了多种基于[动态字段](./dynamic-fields)和[动态对象字段](./dynamic-object-fields)概念构建的集合类型。这些集合被设计为存储和管理动态字段和对象的更安全、更易理解的方式。

对于每种集合类型，我们将指定它们使用的原语以及它们提供的特定功能。

> 与在UID上操作的动态（对象）字段不同，集合类型有自己的类型，允许调用[关联函数](./../move-basics/struct-methods)。

## 通用概念

所有集合类型都共享相同的方法集，这些方法是：

- `add` - 向集合添加字段
- `remove` - 从集合中移除字段
- `borrow` - 从集合中借用字段
- `borrow_mut` - 从集合中借用可变引用字段
- `contains` - 检查字段是否存在于集合中
- `length` - 返回集合中字段的数量
- `is_empty` - 检查`length`是否为0

所有集合类型都支持`borrow`和`borrow_mut`方法的索引语法。如果您在示例中看到方括号，它们会被转换为`borrow`和`borrow_mut`调用。

```move
let hat: &Hat = &bag[b"key"];
let hat_mut: &mut Hat = &mut bag[b"key"];

// 等价于
let hat: &Hat = bag.borrow(b"key");
let hat_mut: &mut Hat = bag.borrow_mut(b"key");
```

在示例中，我们不会专注于这些函数，而是关注集合类型之间的差异。

## Bag

Bag，顾名思义，充当异构值的"袋子"。它是一个简单的、非泛型类型，可以存储任何数据。Bag永远不会允许孤立字段，因为它跟踪字段数量，如果不为空则无法销毁。

```move
module sui::bag;

public struct Bag has key, store {
    /// the ID of this bag
    id: UID,
    /// the number of key-value pairs in the bag
    size: u64,
}
```

_查看[sui::bag][bag-framework]模块的完整文档。_

由于Bag存储任何类型，它提供的额外方法是：

- `contains_with_type` - 检查是否存在具有特定类型的字段

用作结构体字段：

```move file=packages/samples/sources/programmability/dynamic-collections.move anchor=bag_struct

```

使用Bag：

```move file=packages/samples/sources/programmability/dynamic-collections.move anchor=bag_usage

```

## ObjectBag

在`sui::object_bag`模块中定义。与[Bag](#bag)相同，但内部使用[动态对象字段](./dynamic-object-fields)。只能存储对象作为值。

_查看[sui::object_bag][object-bag-framework]模块的完整文档。_

## Table

Table是一个类型化的动态集合，具有固定的键和值类型。它在`sui::table`模块中定义。

```move
module sui::table;

public struct Table<phantom K: copy + drop + store, phantom V: store> has key, store {
    /// the ID of this table
    id: UID,
    /// the number of key-value pairs in the table
    size: u64,
}
```

_查看[sui::table][table-framework]模块的完整文档。_

用作结构体字段：

```move file=packages/samples/sources/programmability/dynamic-collections.move anchor=table_struct

```

使用Table：

```move file=packages/samples/sources/programmability/dynamic-collections.move anchor=table_usage

```

## ObjectTable

在`sui::object_table`模块中定义。与[Table](#table)相同，但内部使用[动态对象字段](./dynamic-object-fields)。只能存储对象作为值。

_查看[sui::object_table][object-table-framework]模块的完整文档。_

## LinkedTable

它在`sui::linked_table`模块中定义，类似于[Table](#table)，但值链接在一起，允许有序插入和删除。

```move
module sui::linked_table;

public struct LinkedTable<K: copy + drop + store, phantom V: store> has key, store {
    /// the ID of this table
    id: UID,
    /// the number of key-value pairs in the table
    size: u64,
    /// the front of the table, i.e. the key of the first entry
    head: Option<K>,
    /// the back of the table, i.e. the key of the last entry
    tail: Option<K>,
}
```

_查看[sui::linked_table][linked-table-framework]模块的完整文档。_

由于LinkedTable中存储的值链接在一起，它有独特的添加和删除方法。

- `push_front` - 在表前端插入键值对
- `push_back` - 在表后端插入键值对
- `remove` - 通过键删除键值对并返回值
- `pop_front` - 删除表的前端，返回键和值
- `pop_back` - 删除表的后端，返回键和值

用作结构体字段：

```move file=packages/samples/sources/programmability/dynamic-collections.move anchor=linked_table_struct

```

使用LinkedTable：

```move file=packages/samples/sources/programmability/dynamic-collections.move anchor=linked_table_usage

```

## 总结

- [Bag](#bag) - 一个可以存储任何类型数据的简单集合。
- [ObjectBag](#objectbag) - 一个只能存储对象的集合。
- [Table](#table) - 一个具有固定键和值类型的类型化动态集合。
- [ObjectTable](#objecttable) - 与Table相同，但只能存储对象。
- [LinkedTable](#linkedtable) - 类似于Table，但值链接在一起。

## 进一步阅读

- [sui::table][table-framework]模块文档。
- [sui::object_table][object-table-framework]模块文档。
- [sui::linked_table][linked-table-framework]模块文档。
- [sui::bag][bag-framework]模块文档。
- [sui::object_bag][object-bag-framework]模块文档。

[table-framework]: https://docs.sui.io/references/framework/sui/table
[object-table-framework]: https://docs.sui.io/references/framework/sui/object_table
[linked-table-framework]: https://docs.sui.io/references/framework/sui/linked_table
[bag-framework]: https://docs.sui.io/references/framework/sui/bag
[object-bag-framework]: https://docs.sui.io/references/framework/sui/object_bag

<!-- TODO! -->

<!-- ## Choosing a Collection Type

Depending on the needs of your project, you may choose to -->