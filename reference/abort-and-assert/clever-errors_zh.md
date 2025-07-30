---
title: '智能错误 | 参考'
description:
  智能错误是一个功能，当断言失败或引发中止时，允许更多信息性的错误消息
---

# 智能错误

智能错误是一个功能，当断言失败或引发中止时，允许更多信息性的错误消息。它们是源代码功能，会编译为包含访问行号、常量名称和常量值所需信息的`u64`中止代码值，给定智能错误代码和声明智能错误常量的模块。由于这种编译，需要后处理才能从`u64`中止代码值转换为人类可读的错误消息。后处理由Sui GraphQL服务器以及Sui CLI自动执行。如果您想手动解码智能中止代码，可以使用[展开智能中止代码](#inflating-clever-abort-codes)中概述的过程来执行。

> 智能错误包括源代码行信息以及其他数据。因此，它们的值可能会因源文件中的任何更改而更改（例如，由于自动格式化、添加新的模块成员或添加换行符）。

## 智能中止代码

智能中止代码允许您使用非u64常量作为中止代码，只要常量用`#[error]`属性注解。它们可以在断言中使用，也可以作为`abort`的代码。

```move
module 0x42::a_module;

#[error]
const EIsThree: vector<u8> = b"The value is three";

// 如果`x`是3，将以`EIsThree`中止
public fun double_except_three(x: u64): u64 {
    assert!(x != 3, EIsThree);
    x * x
}

// 将始终以`EIsThree`中止
public fun clever_abort() {
    abort EIsThree
}
```

在此示例中，`EIsThree`常量是`vector<u8>`，而不是`u64`。但是，`#[error]`属性允许常量用作中止代码，并将在运行时产生一个`u64`中止代码值，该值包含：

1. 指示中止代码是智能中止代码的设置标记位。
2. 源文件中发生中止的行号（例如，7）。
3. 模块标识符表中常量名称的索引（例如，`EIsThree`）。
4. 模块常量表中常量值的索引（例如，`b"The value is three"`）。

在十六进制中，如果调用`double_except_three(3)`，它将以如下`u64`中止代码中止：

```
0x8000_0007_0001_0000
  ^       ^    ^    ^
  |       |    |    |
  |       |    |    |
  |       |    |    +-- 常量值索引 = 0 (b"The value is three")
  |       |    +-- 常量名称索引 = 1 (EIsThree)
  |       +-- 行号 = 7 (断言的行)
  +-- 标记位 = 0b1000_0000_0000_0000
```

可以呈现为人类可读的错误消息（例如）

```
Error from '0x42::a_module::double_except_three' (line 7), abort 'EIsThree': "The value is three"
```

此消息的确切格式可能因用于解码智能错误的工具而异，但是，当与发生错误的模块结合时，在`u64`中止代码中存在生成如上所示的人类可读错误消息所需的所有信息。

> 智能中止代码值_不_需要是`vector<u8>` —— 它可以是Move中的任何有效常量类型。

## 没有中止代码的断言

没有中止代码的断言和`abort`语句将自动从源行号派生中止代码，并将以智能错误格式编码，常量名称和常量值信息将填充为每个`0xffff`的哨兵值。例如，

```move
module 0x42::a_module;

#[test]
fun assert_false(x: bool) {
    assert!(false);
}

#[test]
fun abort_no_code() {
    abort
}
```

这两个都将产生一个`u64`中止代码值，该值包含：

1. 指示中止代码是智能中止代码的设置标记位。
2. 源文件中发生中止的行号（例如，6）。
3. 模块标识符表中常量名称索引的哨兵值`0xffff`。
4. 模块常量表中常量值索引的哨兵值`0xffff`。

在十六进制中，如果调用`assert_false(3)`，它将以如下`u64`中止代码中止：

```
0x8000_0004_ffff_ffff
  ^       ^    ^    ^
  |       |    |    |
  |       |    |    |
  |       |    |    +-- 常量值索引 = 0xffff (哨兵值)
  |       |    +-- 常量名称索引 = 0xffff (哨兵值)
  |       +-- 行号 = 4 (断言的行)
  +-- 标记位 = 0b1000_0000_0000_0000
```

## 智能错误和宏

智能中止代码中的行号信息是从发生中止的位置的源文件派生的。特别是，对于函数，这将是函数内的行号，但是对于宏，这将是调用宏的位置。这在编写宏时非常有用，因为它为用户提供了一种使用可能引发中止条件的宏并仍然获得有用错误消息的方法。

```move
module 0x42::macro_exporter;

public macro fun assert_false() {
    assert!(false);
}

public macro fun abort_always() {
    abort
}

public fun assert_false_fun() {
    assert!(false); // 将始终以此调用的行号中止
}

public fun abort_always_fun() {
    abort // 将始终以此调用的行号中止
}
```

然后在使用这些宏的模块中：

```move
module 0x42::user_module;

use 0x42::macro_exporter::{
    assert_false,
    abort_always,
    assert_false_fun,
    abort_always_fun
};

fun invoke_assert_false() {
    assert_false!(); // 将以此调用的行号中止
}

fun invoke_abort_always() {
    abort_always!(); // 将以此调用的行号中止
}

fun invoke_assert_false_fun() {
    assert_false_fun(); // 将以`assert_false_fun`中断言的行号中止
}

fun invoke_abort_always_fun() {
    abort_always_fun(); // 将以`abort_always_fun`中`abort`的行号中止
}
```

## 展开智能中止代码

确切地说，智能中止代码的布局如下：

```

|<tagbit>|<reserved>|<source line number>|<module identifier index>|<module constant index>|
+--------+----------+--------------------+-------------------------+-----------------------+
| 1-bit  | 15-bits  |       16-bits      |     16-bits             |        16-bits        |

```

注意Move中止将带有一些附加信息 —— 在我们的情况下重要的是发生错误的模块。这很重要，因为标识符索引和常量索引是相对于模块的标识符和常量表的（如果没有设置哨兵值）。

> 要解码智能中止代码，如果标识符索引或常量索引未设置为哨兵值`0xffff`，您需要知道发生错误的模块。

在伪代码中，您可以如下解码智能中止代码：

```rust
// MoveAbort中可用的信息
let clever_abort_code: u64 = ...;
let (package_id, module_name): (PackageStorageId, ModuleName) = ...;

let is_clever_abort = (clever_abort_code & 0x8000_0000_0000_0000) != 0;

if is_clever_abort {
    // 获取行号、标识符索引和常量索引
    // 如果设置为'0xffff'，标识符和常量索引是哨兵值
    let line_number = ((clever_abort_code & 0x0000_ffff_0000_0000) >> 32) as u16;
    let identifier_index = ((clever_abort_code & 0x0000_0000_ffff_0000) >> 16) as u16;
    let constant_index = ((clever_abort_code & 0x0000_0000_0000_ffff)) as u16;

    // 打印行错误消息
    print!("Error from '{}::{}' (line {})", package_id, module_name, line_number);

    // 如果两者都是哨兵值，则无需打印任何内容或加载模块
    if identifier_index == 0xffff && constant_index == 0xffff {
        return;
    }

    // 只有当常量名称和值不是0xffff时才需要
    let module: CompiledModule = fetch_module(package_id, module_name);

    // 打印常量名称（如果有）
    if identifier_index != 0xffff {
        let constant_name = module.get_identifier_at_table_index(identifier_index);
        print!(", '{}'", constant_name);
    }

    // 打印常量值（如果有）
    if constant_index != 0xffff {
        let constant_value = module
            .get_constant_at_table_index(constant_index)
            .deserialize_on_constant_type()
            .to_string();

        print!(": {}", constant_value);
    }

    return;
}
```