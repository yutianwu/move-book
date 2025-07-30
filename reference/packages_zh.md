---
title: 'Packages | Reference'
description: ''
---

# 包

包允许 Move 程序员更轻松地重用代码并在项目间共享。Move 包系统允许程序员轻松地：

- 定义包含 Move 代码的包；
- 通过[命名地址](./primitive-types/address)参数化包；
- 在其他 Move 代码中导入和使用包并实例化命名地址；
- 构建包并从包生成相关的编译产物；以及
- 围绕编译的 Move 产物使用通用接口。

## 包布局和清单语法

Move 包源目录包含 `Move.toml` 包清单文件、生成的 `Move.lock` 文件以及一组子目录：

```plaintext
a_move_package
├── Move.toml      (必需)
├── Move.lock      (生成)
├── sources        (必需)
├── doc_templates  (可选)
├── examples       (可选，测试和开发模式)
└── tests          (可选，测试模式)
```

标记为"必需"的目录和文件必须存在，目录才能被视为 Move 包并被构建。可选目录可能存在，如果存在，它们将根据用于构建包的模式包含在编译过程中。例如，在"开发"或"测试"模式下构建时，`tests` 和 `examples` 目录也将被包含。

逐一介绍这些：

1. [`Move.toml`](#movetoml) 文件是包清单，是目录被视为 Move 包所必需的。此文件包含有关包的元数据，例如名称、依赖项等。
2. [`Move.lock`](#movelock) 文件由 Move CLI 生成，包含包及其依赖项的固定构建版本。它用于确保在不同构建中使用一致的版本，并且依赖项中的更改在此文件中的更改是明显的。
3. `sources` 目录是必需的，包含构成包的 Move 模块。此目录中的模块将始终包含在编译过程中。
4. `doc_templates` 目录可以包含在为包生成文档时将使用的文档模板。
5. `examples` 目录可以保存仅用于开发和/或教程的其他代码，这在 `test` 或 `dev` 模式之外编译时不会包含。
6. `tests` 目录可以包含仅在 `test` 模式下编译或运行 [Move 单元测试](./unit-testing)时包含的 Move 模块。

### Move.toml

Move 包清单在 `Move.toml` 文件中定义，具有以下语法。可选字段用 `*` 标记，`+` 表示一个或多个元素：

```toml
[package]
name = <string>
edition* = <string>      # 例如，"2024.alpha" 使用 Move 2024 版本，
                         # 当前为 alpha 版。如果未指定，将默认为最新稳定版本。
license* = <string>              # 例如，"MIT"、"GPL"、"Apache 2.0"
authors* = [<string>,+]  # 例如，["Joe Smith (joesmith@noemail.com)", "John Snow (johnsnow@noemail.com)"]

# 外部工具可以向此部分添加其他字段。例如，在 Sui 上添加了以下部分：
published-at* = "<hex-address>" # 包发布的地址。应在首次发布后设置。

[dependencies] # （可选部分）依赖项路径
# 声明依赖项的一行或多行，格式如下

# ##### 本地依赖项 #####
# 对于本地依赖项，使用 `local = path`。路径相对于包根目录
# Local = { local = "../path/to" }
# 要解决版本冲突并强制依赖项使用特定版本
# 覆盖，您可以使用 `override = true`
# Override = { local = "../conflicting/version", override = true }
# 要在依赖项中实例化地址值，使用 `addr_subst`
<string> = {
    local = <string>,
    override* = <bool>,
    addr_subst* = { (<string> = (<string> | "<hex_address>"))+ }
}

# ##### Git 依赖项 #####
# 对于远程导入，使用 `{ git = "...", subdir = "...", rev = "..." }`。
# 必须提供修订版本，它可以是分支、标签或提交哈希。
# 如果未指定 `subdir`，则使用存储库的根目录。
# MyRemotePackage = { git = "https://some.remote/host.git", subdir = "remote/path", rev = "main" }
<string> = {
    git = <URL ending in .git>,
    subdir=<path to dir containing Move.toml inside git repo>,
    rev=<git commit hash>,
    override* = <bool>,
    addr_subst* = { (<string> = (<string> | "<hex_address>"))+ }
}

[addresses]  # （可选部分）在此包中声明命名地址
# 声明命名地址的一行或多行，格式如下
# 与包名称匹配的地址必须设置为 `"0x0"`，否则将无法发布。
<addr_name> = "_" | "<hex_address>" # 例如，std = "_" 或 my_addr = "0xC0FFEECAFE"

# 命名地址在 Move 中可作为 `@name` 访问。它们也被导出：
# 例如，`std = "0x1"` 由标准库导出。
# alice = "0xA11CE"

[dev-dependencies] # （可选部分）与 [dependencies] 部分相同，但仅在"dev"和"test"模式下包含
# dev-dependencies 部分允许在 `--test` 和
# `--dev` 模式下覆盖依赖项。您可以例如在此处引入仅用于测试的依赖项。
# Local = { local = "../path/to/dev-build" }
<string> = {
    local = <string>,
    override* = <bool>,
    addr_subst* = { (<string> = (<string> | "<hex_address>"))+ }
}
<string> = {
    git = <URL ending in .git>,
    subdir=<path to dir containing Move.toml inside git repo>,
    rev=<git commit hash>,
    override* = <bool>,
    addr_subst* = { (<string> = (<string> | "<hex_address>"))+ }
}

[dev-addresses] # （可选部分）与 [addresses] 部分相同，但仅在"dev"和"test"模式下包含
# dev-addresses 部分允许在 `--test`
# 和 `--dev` 模式下覆盖命名地址。
<addr_name> = "<hex_address>" # 例如，alice = "0xB0B"
```

最小包清单示例：

```toml
[package]
name = "AName"
```

更标准的包清单示例，还包括 Move 标准库并使用地址值 `0x1` 实例化 `LocalDep` 包中的命名地址 `std`：

```toml
[package]
name = "AName"
license = "Apache 2.0"

[addresses]
address_to_be_filled_in = "_"
specified_address = "0xB0B"

[dependencies]
# 本地依赖项
LocalDep = { local = "projects/move-awesomeness", addr_subst = { "std" = "0x1" } }
# Git 依赖项
MoveStdlib = { git = "https://github.com/MystenLabs/sui.git", subdir = "crates/sui-framework/packages/move-stdlib", rev = "framework/mainnet" }

[dev-addresses] # 开发此模块时使用
address_to_be_filled_in = "0x101010101"
```

包清单中的大多数部分都是不言自明的，但命名地址可能有点难以理解，所以我们在[编译期间的命名地址](#named-addresses-during-compilation)中更详细地检查它们。

## 编译期间的命名地址

回想一下，Move 有[命名地址](./primitive-types/address)，命名地址不能在 Move 中声明。相反，它们在包级别声明：在 Move 包的清单文件（`Move.toml`）中，您在包中声明命名地址、实例化其他命名地址，并在 Move 包系统中重命名来自其他包的命名地址。

让我们逐一介绍这些操作，以及它们在包清单中的执行方式：

### 声明命名地址

假设我们在 `example_pkg/sources/A.move` 中有一个 Move 模块，如下所示：

```move
module named_addr::a {
    public fun x(): address { @named_addr }
}
```

我们可以在 `example_pkg/Move.toml` 中以两种不同的方式声明命名地址 `named_addr`。第一种：

```toml
[package]
name = "example_pkg"
...
[addresses]
named_addr = "_"
```

在包 `example_pkg` 中将 `named_addr` 声明为命名地址，并且*此地址可以是任何有效的地址值*。特别是，导入包可以将命名地址 `named_addr` 的值选择为它希望的任何地址。直观地，您可以将此视为通过命名地址 `named_addr` 参数化包 `example_pkg`，然后包可以稍后由导入包实例化。

`named_addr` 也可以声明为：

```toml
[package]
name = "example_pkg"
...
[addresses]
named_addr = "0xCAFE"
```

这表明命名地址 `named_addr` 正好是 `0xCAFE` 且不能更改。这很有用，因此其他导入包可以使用此命名地址而无需担心分配给它的确切值。

通过这两种不同的声明方法，有两种方式可以在包图中流动有关命名地址的信息：

- 前者（"未分配的命名地址"）允许命名地址值从导入站点流向声明站点。
- 后者（"已分配的命名地址"）允许命名地址值从声明站点向上流向包图中的使用站点。

通过这两种在包图中流动命名地址信息的方法，理解作用域和重命名的规则变得重要。

## 命名地址的作用域和重命名

包 `P` 中的命名地址 `N` 在以下情况下在作用域内：

1. `P` 声明命名地址 `N`；或
2. `P` 的传递依赖项中的包声明命名地址 `N`，并且在包图中 `P` 和 `N` 的声明包之间有依赖路径，没有重命名 `N`。

此外，包中的每个命名地址都被导出。由于这一点和上述作用域规则，每个包都可以被视为带有一组命名地址，这些地址在导入包时将被引入作用域，例如，如果您导入 `example_pkg`，该导入也将把 `named_addr` 命名地址引入作用域。由于这一点，如果 `P` 导入两个包 `P1` 和 `P2`，两者都声明命名地址 `N`，则在 `P` 中出现问题：当在 `P` 中引用 `N` 时，意思是哪个"`N`"？来自 `P1` 还是 `P2` 的？为了防止关于命名地址来自哪个包的歧义，我们强制包中所有依赖项引入的作用域集合是不相交的，并提供一种在导入引入它们的包时*重命名命名地址*的方法。

在我们上面的 `P`、`P1` 和 `P2` 示例中，可以在导入时重命名命名地址，如下所示：

```toml
[package]
name = "P"
...
[dependencies]
P1 = { local = "some_path_to_P1", addr_subst = { "P1N" = "N" } }
P2 = { local = "some_path_to_P2"  }
```

通过此重命名，`N` 指的是来自 `P2` 的 `N`，`P1N` 将指的是来自 `P1` 的 `N`：

```move
module N::A {
    public fun x(): address { @P1N }
}
```

重要的是要注意*重命名不是本地的*：一旦命名地址 `N` 在包 `P` 中被重命名为 `N2`，所有导入 `P` 的包将看不到 `N` 而只能看到 `N2`，除非 `N` 从 `P` 之外重新引入。这就是为什么本节开头的作用域规则中的规则 (2) 指定了"包图中 `P` 和 `N` 的声明包之间没有重命名 `N` 的依赖路径"。

### 实例化命名地址

命名地址可以在包图中多次实例化，只要始终使用相同的值。如果相同的命名地址（无论重命名如何）在包图中使用不同的值实例化，这是错误的。

只有当所有命名地址都解析为值时，Move 包才能编译。如果包希望公开未实例化的命名地址，这会带来问题。这就是 `[dev-addresses]` 部分部分解决的问题。此部分可以为命名地址设置值，但不能引入任何命名地址。此外，只有根包中的 `[dev-addresses]` 在 `dev` 模式下包含。例如，具有以下清单的根包在 `dev` 模式之外不会编译，因为 `named_addr` 将是未实例化的：

```toml
[package]
name = "example_pkg"
...
[addresses]
named_addr = "_"

[dev-addresses]
named_addr = "0xC0FFEE"
```

## 用法和产物

Move 包系统附带命令行选项作为 CLI 的一部分：`sui move <command> <command_flags>`。除非提供特定路径，否则所有包命令都将在当前封闭的 Move 包中运行。通过运行 `sui move --help` 可以找到 Move CLI 的命令和标志的完整列表。

### 产物

可以使用 CLI 命令编译包。这将创建一个包含构建相关产物（包括字节码二进制文件、源映射和文档）的 `build` 目录。`build` 目录的一般布局如下：

```plaintext
a_move_package
├── BuildInfo.yaml
├── bytecode_modules
│   ├── dependencies
│   │   ├── <dep_pkg_name>
│   │   │   └── *.mv
│   │   ...
│   │   └──  <dep_pkg_name>
│   │       └── *.mv
│   ...
│   └── *.mv
├── docs
│   ├── dependencies
│   │   ├── <dep_pkg_name>
│   │   │   └── *.md
│   │   ...
│   │   └──  <dep_pkg_name>
│   │       └── *.md
│   ...
│   └── *.md
├── source_maps
│   ├── dependencies
│   │   ├── <dep_pkg_name>
│   │   │   └── *.mvsm
│   │   ...
│   │   └──  <dep_pkg_name>
│   │       └── *.mvsm
│   ...
│   └── *.mvsm
└── sources
    ...
    └── *.move
    ├── dependencies
    │   ├── <dep_pkg_name>
    │   │   └── *.move
    │   ...
    │   └──  <dep_pkg_name>
    │       └── *.move
    ...
    └── *.move
```

## Move.lock

`Move.lock` 文件在构建包时在 Move 包的根目录生成。`Move.lock` 文件包含有关您的包及其构建配置的信息，并充当 Move 编译器和其他工具（如特定于链的命令行界面和第三方包管理器）之间的通信层。

像 `Move.toml` 文件一样，`Move.lock` 文件是基于文本的 TOML 文件。但是，与包清单不同，`Move.lock` 文件不是供您直接编辑的。工具链上的进程，如 Move 编译器，访问和编辑文件以读取和向其附加相关信息。您也不能从根目录移动文件，因为它需要与包中的 `Move.toml` 清单在同一级别。

如果您对包使用源代码控制，建议的做法是检入与您所需的构建或发布包对应的 `Move.lock` 文件。这确保包的每次构建都是原始的精确副本，并且构建的更改将作为 `Move.lock` 文件的更改而明显。

`Move.lock` 文件是当前包含以下字段的 TOML 文件。

**注意**：将来可能会向锁定文件添加其他字段，或者第三方包管理器也可能添加其他字段。

### `[move]` 部分

此部分包含锁定文件中所需的核心信息：

- 锁定文件的版本（需要向后兼容性检查，以及将来锁定文件更改的版本控制）。
- 用于生成此锁定文件的 `Move.toml` 文件的哈希。
- 所有依赖项的 `Move.lock` 文件的哈希。如果没有依赖项，这将是空字符串。
- 依赖项列表。

```toml
[move]
version = <string> # 锁定文件版本，用于向后兼容性检查。
manifest_digest = <hash> # 用于生成此锁定文件的 Move.toml 文件的 Sha3-256 哈希。
deps_digest = <hash> # 所有依赖项的 Move.lock 文件的 Sha3-256 哈希。如果没有依赖项，这将是空字符串。
dependencies = { (name = <string>)* } # 依赖项列表。如果没有依赖项则不存在。
```

### `[move.package]` 部分

Move 编译器为包解析每个依赖项后，将依赖项的位置写入 `Move.lock` 文件。如果依赖项无法解析，编译器将不会写入 `Move.lock` 文件，构建失败。如果所有依赖项都解析，`Move.lock` 文件包含包的所有传递依赖项的位置（本地和远程）。这些将以以下格式存储在 `Move.lock` 文件中：

```toml
# ...

[[move.package]]
name = "A"
source = { git = "https://github.com/b/c.git", subdir = "e/f", rev = "a1b2c3" }

[[move.package]]
name = "B"
source = { local = "../local-dep" }
```

### `[move.toolchain-version]` 部分

如上所述，外部工具可能会向锁定文件添加其他字段。例如，Sui 包管理器将工具链版本信息添加到锁定文件中，然后可用于链上源验证：

```toml
# ...

[move.toolchain-version]
compiler-version = <string> # 用于构建包的 Move 编译器版本，例如 "1.21.0"
edition = <string> # 用于构建包的 Move 语言版本，例如 "2024.alpha"
flavor = <string> # 用于构建包的 Move 编译器版本，例如 "sui"
```