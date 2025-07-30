# 中止执行

<!-- Consider "aborting execution" -->

<!--

Chapter: Basic Syntax
Goal: Introduce abort keyword and `assert!` macro.
Notes:
    - previous chapter mentions constants
    - error constants standard ECamelCase
    - `assert!` macro
    - asserts should go before the main logic
    - Move has no catch mechanism
    - abort codes are local to the module
    - there are no error messages emitted
    - error codes should handle all possible scenarios in this module

Links:
    - constants (previous section)
 -->

一个交易要么成功要么失败。成功执行会应用对对象和链上数据所做的所有更改，交易被提交到区块链。相反，如果交易中止，更改不会被应用。使用 `abort` 关键字中止交易并回滚已经做出的任何更改。

> 重要的是要注意，Move 中没有捕获机制。如果交易中止，到目前为止所做的更改将被回滚，交易被认为是失败的。

## 中止

`abort` 关键字用于中止交易的执行。它与中止代码结合使用，中止代码返回给交易的调用者。中止代码是类型为 `u64` 的[整数](./primitive-types)。

```move file=packages/samples/sources/move-basics/assert-and-abort.move anchor=abort

```

上面的代码当然会以中止代码 `1` 中止。

## assert!

`assert!` 宏是一个内置宏，可用于断言条件。如果条件为假，交易将以给定的中止代码中止。`assert!` 宏是在不满足条件时中止交易的便捷方式。该宏简化了原本需要用 `if` 表达式 + `abort` 编写的代码。`code` 参数是可选的，但必须是 `u64` 值或 `#[error]`（更多信息见下文）。

```move file=packages/samples/sources/move-basics/assert-and-abort.move anchor=assert

```

## 错误常量

为了使错误代码更具描述性，定义[错误常量](./constants)是一个好习惯。错误常量定义为 `const` 声明，通常以 `E` 为前缀，后跟驼峰命名。错误常量与其他常量类似，没有任何特殊处理。但是，它们通常用于提高代码可读性，使中止场景更容易理解。

```move file=packages/samples/sources/move-basics/assert-and-abort.move anchor=error_const

```

## 错误消息

Move 2024 引入了一种特殊类型的错误常量，用 `#[error]` 属性标记。此属性允许错误常量的类型为 `vector<u8>`，可用于存储错误消息。

```move file=packages/samples/sources/move-basics/assert-and-abort.move anchor=error_attribute

```

## 进一步阅读

- Move 参考中的[中止和断言](./../../reference/abort-and-assert)。
- 我们建议阅读[更好的错误处理](./../guides/better-error-handling)指南，了解 Move 中错误处理的最佳实践。