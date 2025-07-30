# 包清单

`Move.toml`是描述[包](./packages)及其依赖关系的清单文件。它采用[TOML](https://toml.io/en/)格式编写，包含多个部分，其中最重要的是`[package]`、`[dependencies]`和`[addresses]`。

```toml
[package]
name = "my_project"
version = "0.0.0"
edition = "2024"

[dependencies]
Example = { git = "https://github.com/example/example.git", subdir = "path/to/package", rev = "framework/testnet" }

[addresses]
std =  "0x1"
alice = "0xA11CE"

[dev-addresses]
alice = "0xB0B"
```

## 部分

### 包

`[package]`部分用于描述包。此部分中的字段都不会发布到链上，但它们在工具和发布管理中使用；它们还为编译器指定Move版本。

- `name` - 导入时包的名称；
- `version` - 包的版本，可用于发布管理；
- `edition` - Move语言的版本；目前，唯一有效值是`2024`。

<!-- published-at -->

### 依赖关系

`[dependencies]`部分用于指定项目的依赖关系。每个依赖关系都指定为键值对，其中键是依赖关系的名称，值是依赖关系规范。依赖关系规范可以是git仓库URL或本地目录路径。

```toml
# git仓库
Example = { git = "https://github.com/example/example.git", subdir = "path/to/package", rev = "framework/testnet" }

# 本地目录
MyPackage = { local = "../my-package" }
```

包还从其他包导入地址。例如，Sui依赖项将`std`和`sui`地址添加到项目中。这些地址可以在代码中用作地址的别名。

从Sui CLI 1.45版本开始，如果没有明确列出任何Sui系统包（`std`、`sui`、`system`、`bridge`和`deepbook`），则会自动添加它们作为依赖项。

### 使用Override解决版本冲突

有时依赖项对同一包有冲突的版本。例如，如果您有两个使用Example包不同版本的依赖项，您可以在`[dependencies]`部分覆盖依赖项。为此，向依赖项添加`override`字段。将使用`[dependencies]`部分中指定的依赖项版本，而不是依赖项本身指定的版本。

```toml
[dependencies]
Example = { override = true, git = "https://github.com/example/example.git", subdir = "crates/sui-framework/packages/sui-framework", rev = "framework/testnet" }
```

### 开发依赖项

可以向清单添加`[dev-dependencies]`部分。它用于在开发和测试模式下覆盖依赖项。例如，如果您想在开发模式下使用不同版本的Sui包，可以向`[dev-dependencies]`部分添加自定义依赖项规范。

### 地址

`[addresses]`部分用于为地址添加别名。可以在此部分中指定任何地址，然后在代码中用作别名。例如，如果您在此部分中添加`alice = "0xA11CE"`，您可以在代码中使用`alice`作为`0xA11CE`。

### 开发地址

`[dev-addresses]`部分与`[addresses]`相同，但仅适用于测试和开发模式。重要的是要注意，无法在此部分中引入新别名，只能覆盖现有别名。因此在上面的示例中，如果您在此部分中添加`alice = "0xB0B"`，`alice`地址在测试和开发模式下将是`0xB0B`，在常规构建中是`0xA11CE`。

## TOML样式

TOML格式支持两种表格样式：内联和多行。上面的示例使用内联样式，但也可以使用多行样式。您不会想要在`[package]`部分使用它，但对于依赖项来说可能很有用。

```toml
# 内联样式
[dependencies]
Example = { override = true, git = "https://github.com/example/example.git", subdir = "crates/sui-framework/packages/sui-framework", rev = "framework/testnet" }
MyPackage = { local = "../my-package" }
```

```toml
# 多行样式
[dependencies.Example]
override = true
git = "https://github.com/example/example.git"
subdir = "crates/sui-framework/packages/sui-framework"
rev = "framework/testnet"

[dependencies.MyPackage]
local = "../my-package"
```

## 延伸阅读

- Move参考中的[包](./../../reference/packages)。