# Option

`Option` 是表示可能存在也可能不存在的可选值的类型。Move 中的 `Option` 概念借鉴自 Rust，它是 Move 中非常有用的原语。`Option` 在[标准库](./standard-library)中定义，定义如下：

```move
module std::option;

/// Abstraction of a value that may or may not be present.
public struct Option<Element> has copy, drop, store {
    vec: vector<Element>
}
```

_参见 [std::option][option-stdlib] 模块的完整文档。_

> 'std::option' 模块在每个模块中都是隐式导入的，所以你不需要添加显式导入。

`Option` 类型是带有 `Element` 类型参数的泛型类型。它包含一个字段 `vec`，这是一个 `Element` 的 `vector`。向量的长度可以是 0 或 1，分别表示值的缺失或存在。

> 注意：你可能会惊讶于 `Option` 是包含 `vector` 的 `struct` 而不是[枚举][enum-reference]。这是出于历史原因：`Option` 在 Move 支持枚举之前就被添加到 Move 中了。

`Option` 类型有两个变体：`Some` 和 `None`。`Some` 变体包含一个值，而 `None` 变体表示值的缺失。`Option` 类型用于以类型安全的方式表示值的缺失，避免需要空或 `undefined` 值。

## 实际应用

为了展示为什么需要 `Option` 类型，让我们看一个例子。考虑一个应用程序，它接受用户输入并将其存储在变量中。某些字段是必需的，某些是可选的。例如，用户的中间名是可选的。虽然我们可以使用空字符串来表示中间名的缺失，但这需要额外的检查来区分空字符串和缺失的中间名。相反，我们可以使用 `Option` 类型来表示中间名。

```move file=packages/samples/sources/move-basics/option.move anchor=registry

```

在前面的例子中，`middle_name` 字段的类型是 `Option<String>`。这意味着 `middle_name` 字段可以包含一个 String 值（包装在 Some 中），或者是显式为空（由 None 表示）。使用 `Option` 类型使字段的可选性质变得清晰，避免了歧义和需要额外检查来区分空字符串和缺失中间名的需要。

## 创建和使用 Option 值

`Option` 类型以及 `std::option` 模块在 Move 中是隐式导入的。这意味着你可以直接使用 `Option` 类型而不需要 `use` 语句。

要创建 `Option` 类型的值，你可以使用 `option::some` 或 `option::none` 方法。`Option` 值还支持几种操作（借用将在[引用](references#references-1)章节中讨论）：

```move file=packages/samples/sources/move-basics/option.move anchor=usage

```

## 扩展阅读

- 标准库中的 [std::option][option-stdlib]

[enum-reference]: ./../../reference/enums
[option-stdlib]: https://docs.sui.io/references/framework/std/option