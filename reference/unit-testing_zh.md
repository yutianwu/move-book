---
title: '单元测试 | 参考'
description: ''
---

# 单元测试

Move的单元测试使用Move源语言中的三个注解：

- `#[test]` 将函数标记为测试；
- `#[expected_failure]` 标记测试预期失败；
- `#[test_only]` 将模块或模块成员（[`use`](./uses_zh)、[函数](./functions_zh)、[结构体](./structs_zh)或[常量](./constants_zh)）标记为仅在测试时包含的代码。

这些注解可以放置在任何具有任何可见性的适当形式上。每当模块或模块成员被注解为`#[test_only]`或`#[test]`时，除非为测试编译，否则它不会包含在编译的字节码中。

## 测试注解

`#[test]`注解只能放置在没有参数的函数上。此注解将函数标记为要由单元测试工具运行的测试。

```move
#[test] // OK
fun this_is_a_test() { ... }

#[test] // 编译失败，因为测试接受参数
fun this_is_not_correct(arg: u64) { ... }
```

测试也可以被注解为`#[expected_failure]`。此注解标记测试预期会引发错误。有许多选项可以与`#[expected_failure]`注解一起使用，以确保只有具有指定条件的失败才被标记为通过，这些选项在[预期失败](#expected-failures)中详细说明。只有具有`#[test]`注解的函数才能也被注解为`#[expected_failure]`。

使用`#[expected_failure]`注解的一些简单示例如下所示：

```move
#[test, expected_failure]
public fun this_test_will_abort_and_pass() { abort 1 }

#[test, expected_failure]
public fun test_will_error_and_pass() { 1/0; }

// 将通过，因为测试失败时带有预期的中止代码常量。
// ENotFound是模块中定义的常量。
#[test, expected_failure(abort_code = ENotFound)]
public fun test_will_error_and_pass_abort_code() { abort ENotFound }

// 将失败，因为测试失败时带有与预期不同的错误。
#[test, expected_failure(abort_code = my_module::ENotFound)]
public fun test_will_error_and_fail() { 1/0; }

#[test, expected_failure] // 一个属性中可以有多个。这个测试将通过。
public fun this_other_test_will_abort_and_pass() { abort 1 }
```

> **注意**: `#[test]`和`#[test_only]`函数也可以调用[`entry`](./functions_zh#entry-modifier)函数，无论其可见性如何。

## 预期失败

有许多不同的方式可以使用`#[expected_failure]`注解来指定不同类型的错误条件。这些是：

### 1. `#[expected_failure(abort_code = <constant>)]`

如果测试以定义常量的模块中指定的常量值中止，则此方法将通过，否则失败。这是测试预期测试失败的推荐方法。

> **注意**: 您可以在`expected_failure`注解中引用当前模块或包之外的常量。

```move
module pkg_addr::other_module {
    const ENotFound: u64 = 1;
    public fun will_abort() {
        abort ENotFound
    }
}

module pkg_addr::my_module {
    use pkg_addr::other_module;
    const ENotFound: u64 = 1;

    #[test, expected_failure(abort_code = ENotFound)]
    fun test_will_abort_and_pass() { abort ENotFound }

    #[test, expected_failure(abort_code = other_module::ENotFound)]
    fun test_will_abort_and_pass() { other_module::will_abort() }

    // 失败：不会通过，因为我们期望来自错误模块的常量。
    #[test, expected_failure(abort_code = ENotFound)]
    fun test_will_abort_and_pass() { other_module::will_abort() }
}
```

### 2. `#[expected_failure(arithmetic_error, location = <location>)]`

这指定测试预期在指定位置失败并出现算术错误（例如，整数溢出、除零等）。`<location>`必须是模块位置的有效路径，例如`Self`或`my_package::my_module`。

```move
module pkg_addr::other_module {
    public fun will_arith_error() { 1/0; }
}

module pkg_addr::my_module {
    use pkg_addr::other_module;
    
    #[test, expected_failure(arithmetic_error, location = Self)]
    fun test_will_arith_error_and_pass1() { 1/0; }

    #[test, expected_failure(arithmetic_error, location = pkg_addr::other_module)]
    fun test_will_arith_error_and_pass2() { other_module::will_arith_error() }

    // 失败：将失败，因为我们期望它失败的位置与测试实际失败的位置不同。
    #[test, expected_failure(arithmetic_error, location = Self)]
    fun test_will_arith_error_and_fail() { other_module::will_arith_error() }
}
```

### 3. `#[expected_failure(out_of_gas, location = <location>)]`

这指定测试预期在指定位置失败并出现gas耗尽错误。`<location>`必须是模块位置的有效路径，例如`Self`或`my_package::my_module`。

```move
module pkg_addr::other_module {
    public fun will_oog() { loop {} }
}

module pkg_addr::my_module {
    use pkg_addr::other_module;
    
    #[test, expected_failure(out_of_gas, location = Self)]
    fun test_will_oog_and_pass1() { loop {} }

    #[test, expected_failure(arithmetic_error, location = pkg_addr::other_module)]
    fun test_will_oog_and_pass2() { other_module::will_oog() }

    // 失败：将失败，因为我们期望它失败的位置与测试实际失败的位置不同。
    #[test, expected_failure(out_of_gas, location = Self)]
    fun test_will_oog_and_fail() { other_module::will_oog() }
}
```

### 4. `#[expected_failure(vector_error, minor_status = <u64_opt>, location = <location>)]`

这指定测试预期在指定位置失败并出现向量错误，具有给定的`minor_status`（如果提供）。`<location>`必须是模块位置的有效路径，例如`Self`或`my_package::my_module`。`<u64_opt>`是一个可选参数，指定向量错误的次要状态。如果未指定，如果测试失败时带有任何次要状态，测试将通过。如果指定，测试只有在失败时带有指定次要状态的向量错误才会通过。

```move
module pkg_addr::other_module {
    public fun vector_borrow_empty() {
        &vector<u64>[][1];
    }
}

module pkg_addr::my_module {
    #[test, expected_failure(vector_error, location = Self)]
    fun vector_abort_same_module() {
        vector::borrow(&vector<u64>[], 1);
    }

    #[test, expected_failure(vector_error, location = pkg_addr::other_module)]
    fun vector_abort_same_module() {
        other_module::vector_borrow_empty();
    }

    // 可以指定要期望的次要状态（即，向量特定的错误代码）。
    #[test, expected_failure(vector_error, minor_status = 1, location = Self)]
    fun native_abort_good_right_code() {
        vector::borrow(&vector<u64>[], 1);
    }

    // 失败：正确的错误，但位置错误。
    #[test, expected_failure(vector_error, location = pkg_addr::other_module)]
    fun vector_abort_same_module() {
        other_module::vector_borrow_empty();
    }

    // 失败：正确的错误和位置，但次要状态不同，所以这个测试将失败。
    #[test, expected_failure(vector_error, minor_status = 0, location = Self)]
    fun vector_abort_wrong_minor_code() {
        vector::borrow(&vector<u64>[], 1);
    }
}
```

### 5. `#[expected_failure]`

如果测试以_任何_错误代码中止，这将通过。您应该**_非常小心_**使用这个来注解预期的测试失败，并始终优先选择上面描述的方式之一。这些类型的注解示例是：

```move
#[test, expected_failure]
fun test_will_abort_and_pass1() { abort 1 }

#[test, expected_failure]
fun test_will_arith_error_and_pass2() { 1/0; }
```

## 仅测试注解

模块及其任何成员都可以声明为仅测试。如果项目被注解为`#[test_only]`，该项目只会在测试模式下编译时包含在编译的Move字节码中。此外，当在测试模式之外编译时，任何非测试的`#[test_only]`模块的使用都会在编译期间引发错误。

> **注意**: 被注解为`#[test_only]`的函数只能从测试代码中调用，但它们本身不是测试，不会被单元测试框架作为测试运行。

```move
#[test_only] // 仅测试属性可以附加到模块
module abc { ... }

#[test_only] // 仅测试属性可以附加到常量
const MY_ADDR: address = @0x1;

#[test_only] // .. 到使用
use pkg_addr::some_other_module;

#[test_only] // .. 到结构体
public struct SomeStruct { ... }

#[test_only] // .. 和函数。只能从测试代码调用，但这_不是_测试！
fun test_only_function(...) { ... }
```

## 运行单元测试

使用`sui move test`命令为[Move包](./packages_zh)运行单元测试。

运行测试时，每个测试都会`PASS`、`FAIL`或`TIMEOUT`。如果测试用例失败，如果可能的话，会报告失败的位置以及导致失败的函数名称。您可以在下面看到一个示例。

如果测试超过可以为任何单个测试执行的最大指令数，测试将被标记为超时。此边界可以使用下面的选项更改。此外，虽然测试的结果总是确定性的，但默认情况下测试并行运行，因此测试运行中测试结果的顺序是非确定性的，除非只使用一个线程运行，这可以通过选项配置。

上述选项是可以微调测试并帮助调试失败测试的许多选项中的两个。要查看所有可用选项以及每个选项的描述，请将`--help`标志传递给`sui move test`命令：

```
$ sui move test --help
```

## 示例

使用一些单元测试功能的简单模块在以下示例中显示：

首先创建一个空包并切换到该目录：

```bash
$ sui move new test_example; cd test_example
```

接下来在`sources`目录下添加以下模块：

```move
// filename: sources/my_module.move
module test_example::my_module;

public struct Wrapper(u64)

const ECoinIsZero: u64 = 0;

public fun make_sure_non_zero_coin(coin: Wrapper): Wrapper {
    assert!(coin.0 > 0, ECoinIsZero);
    coin
}

#[test]
fun make_sure_non_zero_coin_passes() {
    let coin = Wrapper(1);
    let Wrapper(_) = make_sure_non_zero_coin(coin);
}

#[test, expected_failure(abort_code = ECoinIsZero)]
// 或者 #[test, expected_failure] 如果我们不关心中止代码
fun make_sure_zero_coin_fails() {
    let coin = Wrapper(0);
    let Wrapper(_) = make_sure_non_zero_coin(coin);
}

#[test_only] // 仅测试辅助函数
fun make_coin_zero(coin: &mut Wrapper) {
    coin.0 = 0;
}

#[test, expected_failure(abort_code = ECoinIsZero)]
fun make_sure_zero_coin_fails2() {
    let mut coin = Wrapper(10);
    coin.make_coin_zero();
    let Wrapper(_) = make_sure_non_zero_coin(coin);
}
```

### 运行测试

然后可以使用`move test`命令运行这些测试：

```bash
$ sui move test
INCLUDING DEPENDENCY Bridge
INCLUDING DEPENDENCY DeepBook
INCLUDING DEPENDENCY SuiSystem
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING test_example
Running Move unit tests
[ PASS    ] 0x0::my_module::make_sure_non_zero_coin_passes
[ PASS    ] 0x0::my_module::make_sure_zero_coin_fails
[ PASS    ] 0x0::my_module::make_sure_zero_coin_fails2
Test result: OK. Total tests: 3; passed: 3; failed: 0
```

### 使用测试标志

#### 传递要运行的特定测试

您可以使用`sui move test <str>`运行特定测试或一组测试。这只会运行完全限定名称包含`<str>`的测试。例如，如果我们只想运行名称中包含`"non_zero"`的测试：

```bash
$ sui move test non_zero
INCLUDING DEPENDENCY Bridge
INCLUDING DEPENDENCY DeepBook
INCLUDING DEPENDENCY SuiSystem
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING test_example
Running Move unit tests
[ PASS    ] 0x0::my_module::make_sure_non_zero_coin_passes
Test result: OK. Total tests: 1; passed: 1; failed: 0
```

#### `-i <bound>` 或 `--gas_used <bound>`

这将任何一个测试可以消耗的gas量限制为`<bound>`：

```bash
$ sui move test -i 0
INCLUDING DEPENDENCY Bridge
INCLUDING DEPENDENCY DeepBook
INCLUDING DEPENDENCY SuiSystem
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING test_example
Running Move unit tests
[ TIMEOUT ] 0x0::my_module::make_sure_non_zero_coin_passes
[ FAIL    ] 0x0::my_module::make_sure_zero_coin_fails
[ FAIL    ] 0x0::my_module::make_sure_zero_coin_fails2

Test failures:

Failures in 0x0::my_module:

┌── make_sure_non_zero_coin_passes ──────
│ Test timed out
└──────────────────

┌── make_sure_zero_coin_fails ──────
│ error[E11001]: test failure
│    ┌─ ./sources/my_module.move:22:27
│    │
│ 21 │     fun make_sure_zero_coin_fails() {
│    │         ------------------------- In this function in 0x0::my_module
│ 22 │         let coin = MyCoin(0);
│    │                           ^ Test did not error as expected. Expected test to abort with code 0 <SNIP>
│
│
└──────────────────

┌── make_sure_zero_coin_fails2 ──────
│ error[E11001]: test failure
│    ┌─ ./sources/my_module.move:34:31
│    │
│ 33 │     fun make_sure_zero_coin_fails2() {
│    │         -------------------------- In this function in 0x0::my_module
│ 34 │         let mut coin = MyCoin(10);
│    │                               ^^ Test did not error as expected. Expected test to abort with code 0 <SNIP>
│
│
└──────────────────

Test result: FAILED. Total tests: 3; passed: 0; failed: 3
```

#### `-s` 或 `--statistics`

使用这些标志，您可以收集有关运行测试的统计信息，并报告每个测试的运行时和使用的gas。您还可以添加`csv`（`sui move test -s csv`）以获得csv输出格式的gas使用情况。例如，如果我们想查看上面示例中测试的统计信息：

```bash
$ sui move test -s
INCLUDING DEPENDENCY Bridge
INCLUDING DEPENDENCY DeepBook
INCLUDING DEPENDENCY SuiSystem
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING test_example
Running Move unit tests
[ PASS    ] 0x0::my_module::make_sure_non_zero_coin_passes
[ PASS    ] 0x0::my_module::make_sure_zero_coin_fails
[ PASS    ] 0x0::my_module::make_sure_zero_coin_fails2

Test Statistics:

┌────────────────────────────────────────────────┬────────────┬───────────────────────────┐
│                   Test Name                    │    Time    │         Gas Used          │
├────────────────────────────────────────────────┼────────────┼───────────────────────────┤
│ 0x0::my_module::make_sure_non_zero_coin_passes │   0.001    │             1             │
├────────────────────────────────────────────────┼────────────┼───────────────────────────┤
│ 0x0::my_module::make_sure_zero_coin_fails      │   0.001    │             1             │
├────────────────────────────────────────────────┼────────────┼───────────────────────────┤
│ 0x0::my_module::make_sure_zero_coin_fails2     │   0.001    │             1             │
└────────────────────────────────────────────────┴────────────┴───────────────────────────┘

Test result: OK. Total tests: 3; passed: 3; failed: 0
```