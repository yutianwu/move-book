---
title: '条件表达式 | 参考'
description: ''
---

# 条件`if`表达式

`if`表达式指定某些代码只有在某个条件为真时才应被评估。例如：

```move
if (x > 5) x = x - 5
```

条件必须是`bool`类型的表达式。

`if`表达式可以可选地包含`else`子句来指定当条件为假时要评估的另一个表达式。

```move
if (y <= 10) y = y + 1 else y = 10
```

"真"分支或"假"分支中的一个将被评估，但不会两个都评估。任一分支都可以是单个表达式或表达式块。

条件表达式可以产生值，使得`if`表达式有结果。

```move
let z = if (x < 100) x else 100;
```

真分支和假分支中的表达式必须具有兼容的类型。例如：

```move
// x和y必须是u64整数
let maximum: u64 = if (x > y) x else y;

// 错误！分支类型不同
let z = if (maximum < 10) 10u8 else 100u64;

// 错误！分支类型不同，因为默认的假分支是()而不是u64
if (maximum >= 10) maximum;
```

如果没有指定`else`子句，假分支默认为单元值。以下是等价的：

```move
if (condition) true_branch // 隐含默认：else ()
if (condition) true_branch else ()
```

通常，`if`表达式与[表达式块](./../variables_zh#expression-blocks)结合使用。

```move
let maximum = if (x > y) x else y;
if (maximum < 10) {
    x = x + 10;
    y = y + 10;
} else if (x >= 10 && y >= 10) {
    x = x - 10;
    y = y - 10;
}
```

## 条件语句的语法

> _if-expression_ → **if (** _expression_ **)** _expression_ _else-clause_<sub>_opt_</sub> >
> _else-clause_ → **else** _expression_