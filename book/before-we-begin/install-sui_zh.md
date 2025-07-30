# 安装 Sui

Move 是一种编译语言，因此你需要安装编译器才能编写和运行 Move 程序。编译器包含在 Sui 二进制文件中，可以使用以下方法之一进行安装或下载。

## 通过 suiup 安装

安装 Sui 的最佳方式是使用 [`suiup`](https://github.com/MystenLabs/suiup)。它提供了一种简单的方式来安装二进制文件并管理不同环境（例如 `testnet` 和 `mainnet`）的不同版本的二进制文件。

`suiup` 的安装说明可以在[仓库 README](https://github.com/MystenLabs/suiup) 中找到。

要安装 Sui，运行以下命令：

```bash
suiup install sui
```

## 下载二进制文件

你可以从[发布页面](https://github.com/MystenLabs/sui/releases)下载最新的 Sui 二进制文件。该二进制文件适用于 macOS、Linux 和 Windows。对于教育目的和开发，我们建议使用 `mainnet` 版本。

## 使用 Homebrew 安装 (MacOS)

你可以使用 [Homebrew](https://brew.sh/) 包管理器安装 Sui。

```bash
brew install sui
```

## 使用 Chocolatey 安装 (Windows)

你可以使用 Windows 的 [Chocolatey](https://chocolatey.org/install) 包管理器安装 Sui。

```bash
choco install sui
```

## 使用 Cargo 构建 (MacOS, Linux)

你可以使用 Cargo 包管理器在本地安装和构建 Sui（需要 Rust）

```bash
cargo install --git https://github.com/MystenLabs/sui.git sui --branch mainnet
```

如果你的目标是其中之一，请将此处的分支目标更改为 `testnet` 或 `devnet`。

确保你的系统具有最新的 Rust 版本，使用以下命令。

```bash
rustup update stable
```

## 故障排除

有关安装过程的故障排除，请参考[安装 Sui](https://docs.sui.io/guides/developer/getting-started/sui-install) 指南。