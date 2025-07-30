# 动态对象字段

> 本节扩展了[动态字段](./dynamic-fields)。请先阅读它以了解动态字段的基础知识。

动态字段的另一种变体是_动态对象字段_，它与常规动态字段有某些差异。在本节中，我们将介绍动态对象字段的特性，并解释它们与常规动态字段的区别。

> 一般建议是避免使用动态对象字段，而倾向于使用（仅仅）动态字段，特别是如果不需要通过ID进行直接发现。动态对象字段的额外成本可能无法通过它们提供的好处来证明。

## 定义

动态对象字段在[Sui框架](./sui-framework)的`sui::dynamic_object_fields`模块中定义。它们在许多方面与动态字段相似，但与它们不同，动态对象字段对`Value`类型有额外的约束。`Value`必须具有`key`和`store`的组合，而不是像动态字段那样只需要`store`。

它们在框架定义中不太明确，因为概念本身更抽象：

```move
module sui::dynamic_object_field;

/// Internal object used for storing the field and the name associated with the
/// value. The separate type is necessary to prevent key collision with direct
/// usage of dynamic_field
public struct Wrapper<Name> has copy, drop, store {
    name: Name,
}
```

与[动态字段](./dynamic-fields#definition)部分中的`Field`类型不同，`Wrapper`类型只存储字段的名称。值是对象本身，并且_不被包装_。

`Value`类型上的约束在动态对象字段可用的方法中变得可见。这是`add`函数的签名：

```move
/// 向对象`object: &mut UID`添加动态对象字段，字段由`name: Name`指定。
/// 如果对象已经有该名称的字段，则使用`EFieldAlreadyExists`中止。
public fun add<Name: copy + drop + store, Value: key + store>(
    // 我们在多个地方使用&mut UID进行访问控制
    object: &mut UID,
    name: Name,
    value: Value,
) { /* implementation omitted */ }
```

与[动态字段](./dynamic-fields#usage)部分中相同的其余方法对`Value`类型具有相同的约束。让我们列出它们供参考：

- `add` - 向对象添加动态对象字段
- `remove` - 从对象中删除动态对象字段
- `borrow` - 从对象中借用动态对象字段
- `borrow_mut` - 从对象中借用动态对象字段的可变引用
- `exists_` - 检查动态对象字段是否存在
- `exists_with_type` - 检查是否存在具有特定类型的动态对象字段

此外，还有一个`id`方法，它返回`Value`对象的`ID`而不指定其类型。

## 使用和与动态字段的差异

动态字段和动态对象字段之间的主要区别是后者只允许将_对象_存储为值。这意味着您不能存储原始类型，如`u64`或`bool`。如果不是因为动态对象字段_不被包装_到单独的对象中，这可能被认为是一个限制。

> 包装的宽松要求使对象可通过其ID进行链下发现。但是，如果实现了包装对象索引，此属性可能不突出，使动态对象字段成为冗余功能。

```move file=packages/samples/sources/programmability/dynamic-object-fields.move anchor=usage

```

## 定价差异

动态对象字段比动态字段稍微贵一些。由于它们的内部结构，它们需要2个对象：Name的Wrapper和Value。因此，添加和访问对象字段的成本（加载2个对象而不是动态字段的1个）更高。

## 下一步

动态字段和动态对象字段都是强大的功能，允许在应用程序中进行创新解决方案。但是，它们相对低级，需要仔细处理以避免孤立字段。在下一节中，我们将介绍更高级的抽象 - [动态集合](./dynamic-collections) - 这可以帮助更有效地管理动态字段和对象。