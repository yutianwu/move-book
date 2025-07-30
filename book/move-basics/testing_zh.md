# 测试

测试是软件开发的一个关键方面，特别是在区块链应用程序中，安全性和正确性至关重要。在本节中，我们将介绍 Move 中测试的基础知识，包括如何有效地编写和组织测试。

## `#[test]` 属性

Move 中的测试是用 `#[test]` 属性标记的函数。这个属性告诉编译器该函数是测试函数，应该在执行测试时运行。测试函数是常规函数，但它们必须不接受参数且没有返回值。它们被排除在字节码之外，永远不会被发布。

```move
module book::testing;

// test 属性放在 `fun` 关键字之前（可以在上面或
// 紧靠 `fun` 关键字之前，如 `#[test] fun my_test() { ... }`）
// 在这种情况下，测试的名称将是 `book::testing::simple_test`。
#[test]
fun simple_test() {
    let sum = 2 + 2;
    assert!(sum == 4);
}

// 这个测试的名称将是 `book::testing::more_advanced_test`。
#[test] fun more_advanced_test() {
    let sum = 2 + 2 + 2;
    assert!(sum == 4);
}
```

## 运行测试

要运行测试，您可以使用 `sui move test` 命令。这个命令将首先在_测试模式_下构建包，然后运行包中找到的所有测试。在测试模式下，来自 `sources/` 和 `tests/` 目录的模块都会被处理并执行它们的测试。

```bash
$ sui move test
> UPDATING GIT DEPENDENCY https://github.com/MystenLabs/sui.git
> INCLUDING DEPENDENCY Bridge
> INCLUDING DEPENDENCY DeepBook
> INCLUDING DEPENDENCY SuiSystem
> INCLUDING DEPENDENCY Sui
> INCLUDING DEPENDENCY MoveStdlib
> BUILDING book
> Running Move unit tests
> ...
```

<!-- TODO: fill output -->

## 使用 `#[expected_failure]` 测试失败情况

失败情况的测试可以用 `#[expected_failure]` 标记。当添加到 `#[test]` 函数时，这个属性告诉编译器该测试预期会失败。当您想要测试函数在满足某个条件时会失败，这很有用。

> 注意：这个属性只能添加到 `#[test]` 函数。

该属性可以接受一个参数，指定如果测试失败应该返回的预期中止代码。如果测试返回的中止代码与参数中指定的不同，它将失败。同样，如果执行没有导致中止，测试也会失败。

```move
module book::testing_failure;

const EInvalidArgument: u64 = 1;

#[test]
#[expected_failure(abort_code = 0)]
fun test_fail() {
    abort 0 // 以 0 中止
}

// 属性可以组合在一起
#[test, expected_failure(abort_code = EInvalidArgument)]
fun test_fail_1() {
    abort 1 // 以 1 中止
}
```

`abort_code` 参数可以使用在测试模块中定义的常量，以及从其他模块导入的常量。这是常量可以在其他模块中使用和"访问"的唯一情况。

## 使用 `#[test_only]` 的实用程序

在某些情况下，给测试环境访问一些内部函数或功能是有帮助的。这简化了测试过程，并允许进行更彻底的测试。然而，重要的是要记住这些函数不应该包含在最终包中。这就是 `#[test_only]` 属性派上用场的地方。

```move
module book::testing;

// 使用 `secret` 函数的公共函数。
public fun multiply_by_secret(x: u64): u64 {
    x * secret()
}

/// 不对公众开放的私有函数。
fun secret(): u64 { 100 }

#[test_only]
/// 此函数仅在测试和其他仅用于测试的函数中可用于测试目的。
/// 注意可见性 - 对于 `#[test_only]`，通常使用 `public` 可见性。
public fun secret_for_testing(): u64 {
    secret()
}

#[test]
// 在测试环境中我们可以访问 `secret_for_testing` 函数。
fun test_multiply_by_secret() {
    let expected = secret_for_testing() * 2;
    assert!(multiply_by_secret(2) == expected);
}
```

用 `#[test_only]` 标记的函数将在测试环境中可用，如果它们的可见性设置为 `public`，其他模块也可以使用。

## 进一步阅读

- Move 参考手册中的[单元测试](/reference/unit-testing)。