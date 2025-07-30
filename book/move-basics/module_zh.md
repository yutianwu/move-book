# 模块

<!--

Chapter: Base Syntax
Goal: Introduce module keyword.
Notes:
    - modules are the base unit of code organization
    - module members are private by default
    - types internal to the module have special access rules
    - only module can pack and unpack its types

 -->

模块是 Move 中代码组织的基本单元。模块用于分组和隔离代码，默认情况下模块的所有成员都是模块私有的。在本节中，你将学习如何定义模块、声明其成员以及从其他模块访问它。

## 模块声明

模块使用 `module` 关键字声明，后跟包地址、模块名、分号和模块体。模块名应该使用 `snake_case` - 全小写字母，单词之间用下划线分隔。模块名在包中必须是唯一的。

通常，`sources/` 文件夹中的单个文件包含单个模块。文件名应该与模块名匹配 - 例如，`donut_shop` 模块应该存储在 `donut_shop.move` 文件中。你可以在[编码约定](./../guides/code-quality-checklist)部分阅读更多关于编码约定的信息。

> 如果你需要在一个文件中声明多个模块，你必须使用[模块块](#模块块)语法。

```move file=packages/samples/sources/move-basics/module-label.move anchor=module

```

结构体、函数、常量和导入都是模块的一部分：

- [结构体](./struct)
- [函数](./function)
- [常量](./constants)
- [导入](./importing-modules)
- [结构体方法](./struct-methods)

## 地址和命名地址

模块地址可以指定为：地址_字面量_（不需要 `@` 前缀）或在[包清单](./../concepts/manifest)中指定的命名地址。在下面的例子中，两者是相同的，因为在 `Move.toml` 的 `[addresses]` 部分有一个 `book = "0x0"` 记录。

```move file=packages/samples/sources/move-basics/module.move anchor=address_literal

```

Move.toml 中的地址部分：

```toml
# Move.toml
[addresses]
book = "0x0"
```

## 模块成员

模块成员在模块体内声明。为了说明这点，让我们定义一个带有结构体、函数和常量的简单模块：

```move file=packages/samples/sources/move-basics/module-members.move anchor=members

```

## 模块块

Move 的 2024 版之前的版本要求模块的主体是一个_模块块_ - 模块的内容需要被大括号 `{}` 包围。使用块语法而不是_标签_语法的主要原因是如果你需要在一个文件中定义多个模块。但是，不推荐使用模块块。

```move file=packages/samples/sources/move-basics/module.move anchor=members

```

## 扩展阅读

- Move 参考中的[模块](./../../reference/modules)。