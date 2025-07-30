# 包

<!--

    - packages and how they're stored
        - overview of packages and their contents (use a diagram)
        - how a package is created, and what it consists of
        - what is the package manifest
        - describe how "name" field is used
        - mention the "edition" field
        - what are the folders in a package and what are they for
        - how packages are imported (give Sui as an example)
        - what are addresses, and how they identify packages
        - how packages are published
        - leave a note that packages are also *upgradable*

-->

Move是一种用于编写智能合约的语言——存储和运行在区块链上的程序。单个程序被组织成一个包。包发布在区块链上，由[地址](./address)标识。已发布的包可以通过发送调用其函数的[交易](./what-is-a-transaction)进行交互。它也可以作为其他包的依赖项。

> 要创建新包，请使用`sui move new`命令。要了解有关该命令的更多信息，请运行`sui move new --help`。

包由模块组成——包含函数、类型和其他项目的独立作用域。

```
package 0x...
    module a
        struct A1
        fun hello_world()
    module b
        struct B1
        fun hello_package()
```

## 包结构

在本地，包是一个包含`Move.toml`文件和`sources`目录的目录。`Move.toml`文件——称为"包清单"——包含有关包的元数据，`sources`目录包含模块的源代码。包通常如下所示：

```
sources/
    my_module.move
    another_module.move
    ...
tests/
    ...
examples/
    using_my_module.move
Move.toml
```

`tests`目录是可选的，包含包的测试。放置在`tests`目录中的代码不会发布到链上，仅在测试中可用。`examples`目录可用于代码示例，也不会发布到链上。

## 已发布的包

在开发期间，包没有地址，需要设置为`0x0`。一旦包被发布，它在区块链上获得一个唯一的[地址](./address)，其中包含其模块的字节码。已发布的包变为_不可变的_，可以通过发送交易进行交互。

```
0x...
    my_module: <bytecode>
    another_module: <bytecode>
```

## 链接

- [包清单](./manifest)
- [地址](./address)
- Move参考中的[包](./../../reference/packages)。