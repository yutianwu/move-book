# 导入模块

<!--
    TODO: create a better example for:
        1. Importing a module in general
        2. Importing a member
        3. Importing multiple members
        4. Grouping imports
        5. Self keyword for groups
-->

<!--

Goals:
    - Show the import syntax
    - Local dependencies
    - External dependencies
    - Importing modules from other packages

 -->

Move 通过允许模块导入实现了高度的模块化和代码重用。同一包内的模块可以相互导入，新包也可以依赖于已存在的包并使用它们的模块。本节将介绍导入模块的基础知识以及如何在你自己的代码中使用它们。

## 导入模块

在同一包中定义的模块可以相互导入。`use` 关键字后跟模块路径，模块路径由包地址（或别名）和模块名组成，用 `::` 分隔。

```move title="File: sources/module_one.move" file=packages/samples/sources/move-basics/importing-modules.move anchor=module_one

```

在同一包中定义的另一个模块可以使用 `use` 关键字导入第一个模块。

```move title="File: sources/module_two.move" file=packages/samples/sources/move-basics/importing-modules-two.move anchor=module_two

```

> 注意：你想从另一个模块导入的任何项目（结构体、函数、常量等）都必须用 `public`（或 `public(package)` - 参见[可见性修饰符](./visibility)）关键字标记，以使其在定义模块外部可访问。例如，`module_one` 中的 `Character` 结构体和 `new` 函数被标记为 public，以便它们可以在 `module_two` 中使用。

## 导入成员

你也可以从模块导入特定成员。当你只需要模块中的单个函数或单个类型时，这很有用。语法与导入模块相同，但你在模块路径后添加成员名。

```move file=packages/samples/sources/move-basics/importing-modules-members.move anchor=members

```

## 分组导入

导入可以使用大括号 `{}` 分组到单个 `use` 语句中。这允许在从同一模块或包导入多个成员时编写更干净、更有组织的代码。

```move file=packages/samples/sources/move-basics/importing-modules-grouped.move anchor=grouped

```

在 Move 中导入函数名不太常见，因为函数名可能重叠并造成混淆。推荐的做法是导入整个模块并使用模块路径访问函数。类型具有唯一名称，应该单独导入。

要在分组导入中导入成员和模块本身，你可以使用 `Self` 关键字。`Self` 关键字指代模块本身，可以用于导入模块及其成员。

```move file=packages/samples/sources/move-basics/importing-modules-self.move anchor=self

```

## 解决名称冲突

当从不同模块导入多个成员时，可能会出现名称冲突。例如，如果你导入两个都有同名函数的模块，你需要使用模块路径访问函数。在不同包中也可能有同名的模块。为了解决冲突并避免歧义，Move 提供了 `as` 关键字来重命名导入的成员。

```move file=packages/samples/sources/move-basics/importing-modules-conflict-resolution.move anchor=conflict

```

## 添加外部依赖

Move 包可以依赖于其他包；依赖项在名为 `Move.toml` 的[包清单](./../concepts/manifest)文件中列出。

包依赖项在[包清单](./../concepts/manifest)中定义如下：

```ini title="Move.toml"
[dependencies]
Example = { git = "https://github.com/Example/example.git", subdir = "path/to/package", rev = "v1.2.3" }
Local = { local = "../my_other_package" }
```

`dependencies` 部分包含每个包依赖项的条目。条目的键是包的名称（示例中的 `Example` 或 `Local`），值是 git 导入表或本地路径。git 导入包含包的 URL、包所在的子目录和包的版本。本地路径是包目录的相对路径。

如果你添加一个依赖项，它的所有依赖项也对你的包可用。

如果依赖项被添加到 `Move.toml` 文件中，编译器在构建包时会自动获取（并在稍后重新获取）依赖项。

> 从 sui CLI 版本 1.45 开始，如果系统包不在 `Move.toml` 中，它们会自动作为所有包的依赖项包含在内。因此，`MoveStdlib`、`Sui`、`System`、`Bridge` 和 `Deepbook` 都可以在没有显式导入的情况下使用。

## 从另一个包导入模块

通常，包在 `[addresses]` 部分定义它们的地址。你可以使用别名而不是完整地址。例如，不使用 `0x2::coin` 来引用 Sui `coin` 模块，你可以使用 `sui::coin`。`sui` 别名在 Sui Framework 包的清单中定义。类似地，`std` 别名在标准库包中定义，可以用来代替 `0x1` 访问标准库模块。

要从另一个包导入模块，使用 `use` 关键字后跟模块路径。模块路径由包地址（或别名）和模块名组成，用 `::` 分隔。

```move file=packages/samples/sources/move-basics/importing-modules-external.move anchor=external

```

> 注意：模块地址名来自清单文件（`Move.toml`）的 `[addresses]` 部分，而不是 `[dependencies]` 部分中使用的名称。