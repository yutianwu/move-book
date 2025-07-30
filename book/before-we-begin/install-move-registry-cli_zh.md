# 安装 MVR

[Move Registry (MVR)](https://moveregistry.com) 是 Move 的包管理器。它允许任何人发布和使用已发布的包来开发用 Move 编写的新应用程序。本地二进制文件允许在注册表中搜索包，并将它们作为 Sui CLI 构建过程的一部分进行安装。

## 通过 suiup 安装

安装 MVR 的最佳方式是使用 [`suiup`](https://github.com/MystenLabs/suiup)。Suiup 提供了一种简单的方式来更新和管理不同版本的二进制文件。

`suiup` 的安装说明可以在[仓库 README](https://github.com/MystenLabs/suiup) 中找到。

要安装 Move Registry CLI，运行以下命令：

```bash
suiup install mvr
```

安装后，Move Registry 将以 `mvr` 的形式可用。

## 下载二进制文件

你可以从[发布页面](https://github.com/MystenLabs/mvr/releases)下载最新的 MVR 二进制文件。该二进制文件适用于 macOS、Linux 和 Windows。与 [Sui](./install-sui.md) 不同，MVR 二进制文件在不同环境间不会改变，同时支持 `testnet` 和 `mainnet`。

## 使用 Cargo 安装

你可以使用 Cargo 在本地安装和构建 MVR（需要 Rust）

```bash
cargo install --locked --git https://github.com/mystenlabs/mvr --branch release mvr
```

## 故障排除

有关安装过程的故障排除，请参考[安装 MVR](https://docs.suins.io/move-registry/tooling/mvr-cli#installation) 指南。