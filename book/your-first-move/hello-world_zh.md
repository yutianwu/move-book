# Hello, World!

在本章中，您将学习如何创建新包、编写简单模块、编译它并使用Move CLI运行测试。确保您已[安装Sui](./../before-we-begin/install-sui)并设置好您的[IDE环境](./../before-we-begin/ide-support)。运行下面的命令来测试Sui是否已正确安装。

```bash
# 它应该打印客户端版本。例如 sui-client 1.22.0-036299745。
sui client --version
```

> Move CLI是Move语言的命令行界面；它内置于Sui二进制文件中，提供一组管理包、编译和测试代码的命令。

本章的结构如下：

- [创建新包](#create-a-new-package)
- [目录结构](#directory-structure)
- [编译包](#compiling-the-package)
- [运行测试](#running-tests)

## 创建新包

要创建新程序，我们将使用`sui move new`命令，后跟应用程序的名称。我们的第一个程序将被称为`hello_world`。

> 注意：在本章和其他章节中，如果您看到以`$`（美元符号）开头的代码块，这意味着应该在终端中运行以下命令。不应包含该符号。这是在终端环境中显示命令的常见方式。

```bash
$ sui move new hello_world
```

`sui move`命令提供对Move CLI的访问——一个内置编译器、测试运行器和Move的实用工具。`new`命令后跟包的名称将在新文件夹中创建新包。在我们的例子中，文件夹名称是"hello_world"。

我们可以查看文件夹的内容以确认包创建成功。

```bash
$ ls -l hello_world
Move.toml
sources
tests
```

## 目录结构

Move CLI将创建应用程序的脚手架，并预创建目录结构和所有必要的文件。让我们看看里面有什么。

```plaintext
hello_world
├── Move.toml
├── sources
│   └── hello_world.move
└── tests
    └── hello_world_tests.move
```

### 清单

`Move.toml`文件，被称为[包清单](./../concepts/manifest)，包含包的定义和配置设置。Move编译器使用它来管理包元数据、获取依赖项和注册命名地址。我们将在[概念](./../concepts)章节中详细解释它。

> 默认情况下，包具有一个命名地址——包的名称。

```toml
[addresses]
hello_world = "0x0"
```

### 源文件

`sources/`目录包含源文件。Move源文件具有_.move_扩展名，通常根据文件中定义的模块命名。例如，在我们的例子中，文件名是_hello_world.move_，Move CLI已经在其中放置了注释掉的代码：

```move
/*
/// Module: hello_world
module hello_world::hello_world;
*/
```

> `/*`和`*/`是Move中的注释分隔符。两者之间的所有内容都被编译器忽略，可用于文档或笔记。我们在[基本语法](./../move-basics/comments)中解释所有注释代码的方式。

注释掉的代码是模块定义，以关键字`module`开始，后跟命名地址（或地址字面量）和模块名。模块名是模块的唯一标识符，在包内必须是唯一的。模块名用于从其他模块或交易中引用模块。

<!-- And the module name has to be a valid Move identifier: alphanumeric with underscores to separate words. A common convention is to call modules (and functions) in snake_case - all lowercase, with underscores. Coding conventions are important for readability and maintainability of the code, we summarize them in the Coding Conventions section. -->

### 测试

`tests/`目录包含包测试。编译器在常规构建过程中排除这些文件，但在_test_和_dev_模式下使用它们。测试用Move编写，并用`#[test]`属性标记。测试可以分组在单独的模块中（通常称为_module_name_tests.move_），或在它们测试的模块内部。

模块、导入、常量和函数可以用`#[test_only]`注解。此属性用于从构建过程中排除模块、函数或导入。当您想为测试添加辅助函数而不将它们包含在将发布到链上的代码中时，这很有用。

_hello_world_tests.move_文件包含注释掉的测试模块模板：

```move
/*
#[test_only]
module hello_world::hello_world_tests;
// uncomment this line to import the module
// use hello_world::hello_world;

const ENotImplemented: u64 = 0;

#[test]
fun test_hello_world() {
    // pass
}

#[test, expected_failure(abort_code = hello_world::hello_world_tests::ENotImplemented)]
fun test_hello_world_fail() {
    abort ENotImplemented
}
*/
```

### 其他文件夹

此外，Move CLI支持`examples/`文件夹。那里的文件与放置在`tests/`文件夹下的文件处理类似——它们只在_test_和_dev_模式下构建。它们是如何使用包或如何将其与其他包集成的示例。最受欢迎的用例是用于文档目的和库包。

## 编译包

Move是编译语言，因此需要将源文件编译为Move字节码。它只包含有关模块、其成员和类型的必要信息，排除注释和一些标识符（例如，常量）。

为了演示这些功能，让我们用以下内容替换_sources/hello_world.move_文件的内容：

```move
/// The module `hello_world` under named address `hello_world`.
/// The named address is set in the `Move.toml`.
module hello_world::hello_world;

// Imports the `String` type from the Standard Library
use std::string::String;

/// Returns the "Hello, World!" as a `String`.
public fun hello_world(): String {
    b"Hello, World!".to_string()
}
```

在编译期间，代码被构建，但不运行。编译的包仅包含可以被其他模块或在交易中调用的函数。我们将在[概念](./../concepts)章节中解释这些概念。但现在，让我们看看运行_sui move build_时会发生什么。

```bash
# 从`hello_world`文件夹运行
$ sui move build

# 或者，如果您没有`cd`进入它
$ sui move build --path hello_world
```

它应该在您的控制台输出以下消息。

```plaintext
UPDATING GIT DEPENDENCY https://github.com/MystenLabs/sui.git
INCLUDING DEPENDENCY Bridge
INCLUDING DEPENDENCY DeepBook
INCLUDING DEPENDENCY SuiSystem
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING hello_world
```

在编译过程中，Move编译器自动创建一个构建文件夹，在其中放置所有获取和编译的依赖项以及当前包模块的字节码。

> 如果您使用版本控制系统，如Git，应忽略构建文件夹。例如，您应该使用`.gitignore`文件并将`build`添加到其中。

## 运行测试

在我们开始测试之前，我们应该添加一个测试。Move编译器支持用Move编写的测试并提供执行环境。测试可以放在源文件和`tests/`文件夹中。测试用`#[test]`属性标记，并由编译器自动发现。我们在[测试](./../move-basics/testing)部分深入解释测试。

用以下内容替换`tests/hello_world_tests.move`的内容：

```move
#[test_only]
module hello_world::hello_world_tests;

use hello_world::hello_world;

#[test]
fun test_hello_world() {
    assert!(hello_world::hello_world() == b"Hello, World!".to_string(), 0);
}
```

在这里我们导入`hello_world`模块，并调用其`hello_world`函数来测试输出确实是字符串"Hello, World!"。现在，我们有了测试，让我们在测试模式下编译包并运行测试。Move CLI有`test`命令用于此目的：

```bash
$ sui move test
```

输出应该类似于以下内容：

```plaintext
INCLUDING DEPENDENCY Bridge
INCLUDING DEPENDENCY DeepBook
INCLUDING DEPENDENCY SuiSystem
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING hello_world
Running Move unit tests
[ PASS    ] 0x0::hello_world_tests::test_hello_world
Test result: OK. Total tests: 1; passed: 1; failed: 0
```

如果您在包文件夹外运行测试，可以指定包的路径：

```bash
$ sui move test --path hello_world
```

您也可以通过指定字符串一次运行单个或多个测试。所有包含该字符串的测试名称都将运行：

```bash
$ sui move test test_hello
```

## 下一步

在本节中，我们解释了Move包的基础：其结构、清单、构建和测试流程。[在下一页](./hello-sui)中，我们将编写一个应用程序并了解代码的结构以及语言能做什么。

## 延伸阅读

- [包清单](./../concepts/manifest)部分
- [Move参考](./../../reference/packages)中的包