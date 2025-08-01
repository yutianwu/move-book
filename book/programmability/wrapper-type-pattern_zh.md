# 模式：包装器类型

有时，需要创建一个新类型，它的行为类似于现有类型，但具有某些修改或限制。例如，您可能想创建一个[集合类型](./collections)，它的行为像向量，但不允许在插入元素后修改它们。包装器类型模式是实现这一点的有效方式。

## 定义

包装器类型模式是一种设计模式，其中您创建一个包装现有类型的新类型。包装器类型与原始类型不同，但可以相互转换。

通常，它被实现为具有单个字段的位置结构体。

```move file=packages/samples/sources/programmability/wrapper-type-pattern.move anchor=main

```

## 常见实践

在目标是扩展现有类型行为的情况下，通常为包装类型提供访问器。这种方法允许用户在需要时直接访问底层类型。例如，在以下代码中，我们为Stack类型提供了`inner()`、`inner_mut()`和`into_inner()`方法。

```move file=packages/samples/sources/programmability/wrapper-type-pattern.move anchor=common

```

## 优势

包装器类型模式提供几个好处：

- 自定义函数：它允许您为现有类型定义自定义函数。
- 健壮的函数签名：它将函数签名约束为新类型，从而使代码更健壮。
- 改善可读性：它通过提供更具描述性的类型名称，通常增加代码的可读性。

## 劣势

包装器类型模式在两种场景中很强大 - 当您想要限制现有类型的行为同时为相同数据结构提供自定义接口时，以及当您想要扩展现有类型的行为时。但是，它确实有一些限制：

- 冗长性：实现可能很冗长，特别是如果您想暴露包装类型的所有方法。
- 稀疏实现：实现可能相当最小，因为它通常只是将调用转发给包装类型。

## 下一步

包装器类型模式非常有用，特别是与集合类型结合使用时，如前一节所示。在下一节中，我们将介绍[动态字段](./dynamic-fields) — 一个重要的原语，它使[动态集合](./dynamic-collections)成为可能，这是一种以更灵活但更昂贵的方式存储大型数据集合的方法。