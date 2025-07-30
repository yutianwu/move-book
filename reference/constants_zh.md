---
title: '常量 | 参考手册'
description: ''
---

# 常量

常量是在 `module` 内部为共享的静态值命名的一种方式。

常量的值必须在编译时已知。常量的值存储在编译后的模块中。每次使用常量时，都会创建该值的新副本。

## 声明

常量声明以 `const` 关键字开始，后跟名称、类型和值。

```text
const <name>: <type> = <expression>;
```

例如

```move
module a::example;

const MY_ADDRESS: address = @a;

public fun permissioned(addr: address) {
    assert!(addr == MY_ADDRESS, 0);
}
```

## 命名

常量必须以大写字母 `A` 到 `Z` 开头。在第一个字母之后，常量名称可以包含下划线 `_`、字母 `a` 到 `z`、字母 `A` 到 `Z` 或数字 `0` 到 `9`。

```move
const FLAG: bool = false;
const EMyErrorCode: u64 = 0;
const ADDRESS_42: address = @0x42;
```

即使你可以在常量中使用字母 `a` 到 `z`，[通用风格指南](./coding-conventions)建议只使用大写字母 `A` 到 `Z`，每个单词之间用下划线 `_` 分隔。对于错误代码，我们使用 `E` 作为前缀，然后为名称的其余部分使用大驼峰命名法（也称为帕斯卡命名法），如 `EMyErrorCode` 所示。

当前以 `A` 到 `Z` 开头的命名限制是为了给未来的语言特性留出空间。

## 可见性

目前不支持 `public` 或 `public(package)` 常量。`const` 值只能在声明模块中使用。但是，为了方便起见，它们可以在[单元测试属性](./unit-testing)中跨模块使用。

## 有效表达式

目前，常量仅限于基本类型 `bool`、`u8`、`u16`、`u32`、`u64`、`u128`、`u256`、`address` 和 `vector<T>`，其中 `T` 是常量的有效类型。

### 值

通常，`const` 被分配一个其类型的简单值或字面量。例如

```move
const MY_BOOL: bool = false;
const MY_ADDRESS: address = @0x70DD;
const BYTES: vector<u8> = b"hello world";
const HEX_BYTES: vector<u8> = x"DEADBEEF";
```

### 复杂表达式

除了字面量，常量还可以包含更复杂的表达式，只要编译器能够在编译时将表达式简化为值。

目前，相等操作、所有布尔操作、所有位操作和所有算术操作都可以使用。

```move
const RULE: bool = true && false;
const CAP: u64 = 10 * 100 + 1;
const SHIFTY: u8 = {
    (1 << 1) * (1 << 2) * (1 << 3) * (1 << 4)
};
const HALF_MAX: u128 = 340282366920938463463374607431768211455 / 2;
const REM: u256 =
    57896044618658097711785492504343953926634992332820282019728792003956564819968 % 654321;
const EQUAL: bool = 1 == 1;
```

如果操作会导致运行时异常，编译器将给出错误，指出无法生成常量的值

```move
const DIV_BY_ZERO: u64 = 1 / 0; // 错误!
const SHIFT_BY_A_LOT: u64 = 1 << 100; // 错误!
const NEGATIVE_U64: u64 = 0 - 1; // 错误!
```

此外，常量可以引用同一模块内的其他常量。

```move
const BASE: u8 = 4;
const SQUARE: u8 = BASE * BASE;
```

但是请注意，常量定义中的任何循环都会导致错误。

```move
const A: u16 = B + 1;
const B: u16 = A + 1; // 错误!
```