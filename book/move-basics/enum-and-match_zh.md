# 枚举和匹配

枚举是一种用户定义的数据结构，与[结构体](./struct)不同，它可以表示多个变体。每个变体可以包含原始类型、结构体或其他枚举。然而，递归枚举定义——类似于递归结构体定义——是不被允许的。

## 定义

枚举使用 `enum` 关键字定义，后跟可选的能力和变体定义块。每个变体都有一个标签名称，可以选择包含位置值或命名字段。枚举必须至少有一个变体。每个变体的结构不是灵活的，变体的总数可以相对较大——最多 100 个。

```move file=packages/samples/sources/move-basics/enum-and-match.move anchor=definition

```

在上面的代码示例中，我们定义了一个公共的 `Segment` 枚举，它具有 `drop` 和 `copy` 能力，以及 3 个变体：

- `Empty`，没有字段。
- `String`，包含一个类型为 `String` 的单个位置字段。
- `Special`，使用命名字段：类型为 `vector<u8>` 的 `content` 和类型为 `u8` 的 `encoding`。

## 实例化

枚举对于定义它们的模块是 _内部的_。这意味着枚举只能在同一个模块内构造、读取和解包。

[类似于结构体](./struct#create-and-use-an-instance)，枚举通过指定类型、变体以及该变体中定义的任何字段的值来实例化。

```move file=packages/samples/sources/move-basics/enum-and-match.move anchor=constructors

```

根据使用情况，您可能想要提供公共构造函数，或者将枚举作为应用程序逻辑的一部分进行内部实例化。

## 在类型定义中使用

使用枚举的最大好处是能够在单一类型下表示不同的数据结构。为了演示这一点，让我们定义一个包含 `Segment` 值向量的结构体：

```move file=packages/samples/sources/move-basics/enum-and-match.move anchor=struct

```

Segment 枚举的所有变体都共享相同的类型——`Segment`——这允许我们创建包含不同变体实例的同构向量。这种灵活性是结构体无法实现的，因为每个结构体定义一个单一的、固定的形状。

## 模式匹配

与结构体不同，枚举在访问内部值或检查变体时需要特殊处理。我们不能简单地使用 `.`（点）语法读取枚举的内部字段，因为我们需要确保我们试图访问的值是正确的。为此，Move 提供了 _模式匹配_ 语法。

> 本章不打算涵盖 Move 中模式匹配的所有特性。请参考 Move 参考书中的[模式匹配](./../../reference/control-flow/pattern-matching)部分。

模式匹配允许根据值的 _模式_ 进行逻辑条件判断。它使用 `match` 表达式执行，后跟括号中的匹配值和 _匹配分支_ 块，定义模式和在模式正确时要执行的表达式。

让我们通过添加一组类似 `is_variant` 的函数来扩展我们的示例，以便外部包可以检查变体。从 `is_empty` 开始。

```move file=packages/samples/sources/move-basics/enum-and-match.move anchor=is_empty

```

`match` 关键字开始表达式，`s` 是被测试的值。每个匹配分支检查 `Segment` 枚举的特定变体。如果 `s` 匹配 `Segment::Empty`，函数返回 `true`；否则返回 `false`。

对于有字段的变体，我们需要将内部结构绑定到局部变量（即使我们不使用它们，也要用 `_` 标记未使用的值以避免编译器警告）。

### 技巧 #1 - _任意_ 条件

Move 编译器推断 `match` 表达式中使用的值的类型，并确保 _匹配分支_ 是穷尽的——也就是说，必须涵盖所有可能的变体或值。

然而，在某些情况下，比如匹配原始值或像向量这样的集合，列出每种可能的情况是不可行的。对于这些情况，匹配支持通配符模式（`_`），它作为默认分支。当没有其他模式匹配时，执行此分支。

我们可以通过简化我们的 `is_empty` 函数并用通配符替换非 `Empty` 变体来演示这一点：

```move file=packages/samples/sources/move-basics/enum-and-match.move anchor=is_empty_2
public fun is_empty(s: &Segment): bool {

```

同样，我们可以使用相同的方法来定义 `is_special` 和 `is_string`：

```move file=packages/samples/sources/move-basics/enum-and-match.move anchor=accessors

```

### 技巧 #2 - `try_into` 辅助函数

随着 `is_variant` 函数的添加，我们使外部模块能够检查枚举实例表示的变体。然而，这通常是不够的——由于枚举对其模块是内部的，外部代码仍然无法访问变体的内部值。

解决这个问题的常见模式是定义 `try_into` 函数。这些函数对值进行匹配，如果 `match` 成功，则返回包含内部内容的 `Option`。

```move file=packages/samples/sources/move-basics/enum-and-match.move anchor=try_into_inner_string

```

这种模式以受控方式安全地暴露内部数据，避免中止。

### 技巧 #3 - 匹配原始值

Move 中的 `match` 表达式可以用于任何类型的值——枚举、结构体或原始类型。为了演示这一点，让我们实现一个 `to_string` 函数，从 `Segment` 创建新的 `String`。在 `Special` 变体的情况下，我们将匹配 `encoding` 字段来确定如何解码内容。

```move file=packages/samples/sources/move-basics/enum-and-match.move anchor=to_string

```

这个函数演示了两个关键点：

- 嵌套的 `match` 表达式可以用于更深层的逻辑分支。
- 通配符对于覆盖原始类型（如 `u8`）的所有可能值是必需的。

## 最终测试

现在我们可以使用我们添加的功能完成之前开始的测试。让我们创建一个场景，将枚举构建到向量中。

```move file=packages/samples/sources/move-basics/enum-and-match-2.move anchor=enum_test

```

这个测试演示了完整的枚举工作流程：实例化不同的变体、使用公共访问器以及使用模式匹配执行逻辑。这应该足以让您开始！

要了解有关枚举和模式匹配的更多信息，请参考[进一步阅读](#further-reading)部分列出的资源。

## 总结

- 枚举是用户定义的类型，可以在单一类型下表示多个变体。
- 每个变体可以包含不同类型的数据（原始类型、结构体或其他枚举）。
- 枚举对其定义模块是内部的，需要模式匹配来访问。
- 模式匹配使用 `match` 表达式完成，它：
  - 适用于枚举、结构体和原始值；
  - 必须处理所有可能的情况（穷尽）；
  - 支持 `_` 通配符模式用于剩余情况；
  - 可以返回值并在表达式中使用；
- 枚举的常见模式包括 `is_variant` 检查和 `try_into` 辅助函数。

## 进一步阅读

- Move 参考中的[枚举](./../../reference/enums)
- Move 参考中的[模式匹配](/reference/control-flow/pattern-matching)