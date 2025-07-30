# 能力：Copy

在 Move 中，类型的 _copy_ 能力表示该类型的实例或值可以被复制或克隆。虽然在处理数字或其他原始类型时默认提供这种行为，但对于自定义类型来说，这不是默认行为。Move 被设计用来表达数字资产和资源，控制复制资源的能力是资源模型的一个关键原则。然而，Move 类型系统允许你为自定义类型添加 _copy_ 能力：

```move file=packages/samples/sources/move-basics/copy-ability.move anchor=copyable

```

在上面的示例中，我们定义了一个具有 _copy_ 能力的自定义类型 `Copyable`。这意味着 `Copyable` 的实例可以被复制，包括隐式复制和显式复制。

```move file=packages/samples/sources/move-basics/copy-ability.move anchor=copyable_test

```

在上面的示例中，`a` 被隐式复制给 `b`，然后使用解引用操作符显式复制给 `c`。如果 `Copyable` 没有 _copy_ 能力，代码将无法编译，Move 编译器会抛出错误。

> 注意：在 Move 中，使用空括号进行解构通常用于消费未使用的变量，特别是对于没有 drop 能力的类型。这可以防止值超出作用域而没有显式使用时出现编译器错误。此外，Move 在解构时需要类型名称（例如，`let Copyable {} = a;` 中的 `Copyable`），因为它强制执行严格的类型和所有权规则。

## 复制和删除

`copy` 能力与 [`drop` 能力](./drop-ability) 密切相关。如果一个类型具有 _copy_ 能力，它很可能也应该具有 `drop` 能力。这是因为当实例不再需要时，需要 _drop_ 能力来清理资源。如果一个类型只有 _copy_ 能力，管理其实例会变得更加复杂，因为实例必须被显式使用或消费。

```move file=packages/samples/sources/move-basics/copy-ability.move anchor=copy_drop

```

Move 中的所有原始类型表现得好像它们具有 _copy_ 和 _drop_ 能力。这意味着它们可以被复制和删除，Move 编译器将为它们处理内存管理。

## 具有 `copy` 能力的类型

Move 中的所有原生类型都具有 `copy` 能力。这包括：

- [bool](./../move-basics/primitive-types#booleans)
- [无符号整数](./../move-basics/primitive-types#integer-types)
- [vector](./../move-basics/vector)
- [address](./../move-basics/address)

标准库中定义的所有类型也都具有 `copy` 能力。这包括：

- [Option](./../move-basics/option)
- [String](./../move-basics/string)
- [TypeName](./../move-basics/type-reflection)

## 进一步阅读

- Move 参考中的[类型能力](./../../reference/abilities)。