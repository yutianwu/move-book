# 开源库

开源库是为 Move 生态系统做贡献的绝佳方式。本指南将帮助您了解如何开源库、如何编写测试以及如何为您的库编写文档。

## README

TODO: readme

## 命名地址

TODO: 命名地址

## 生成文档

TODO: 文档生成

## 添加示例

在发布用于使用的包（NFT 协议或库）时，展示该包的使用方法非常重要。这就是示例派上用场的地方。Move 中没有针对示例的特殊功能，但是有一些用于标记示例的约定。首先，只有源代码会包含在包的字节码中，因此放置在不同目录中的任何代码都不会被包含，但会被测试！

这就是为什么将示例放在单独的 `examples/` 目录中是个好主意。

```bash
sources/
    protocol.move
    library.move
tests/
    protocol_test.move
examples/
    my_example.move
Move.toml
```

## 标签和发布 (Git)

TODO: 标签和发布

## 允许与闭源兼容的技巧

TODO: 通过带签名的空函数实现兼容性