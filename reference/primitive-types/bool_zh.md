---
title: '布尔值 | 参考'
description: ''
---

# 布尔值

`bool`是Move用于布尔`true`和`false`值的原始类型。

## 字面量

`bool`的字面量是`true`或`false`。

## 操作

### 逻辑运算

`bool`支持三种逻辑运算：

| 语法                    | 描述                  | 等价表达式                                               |
| ------------------------- | ---------------------------- | ------------------------------------------------------------------- |
| `&&`                      | 短路逻辑与 | `p && q`等价于`if (p) q else false`                     |
| <code>&vert;&vert;</code> | 短路逻辑或  | <code>p &vert;&vert; q</code>等价于`if (p) true else q` |
| `!`                       | 逻辑非             | `!p`等价于`if (p) false else true`                      |

### 控制流

`bool`值用于Move的几个控制流构造中：

- [`if (bool) { ... }`](./../control-flow/conditionals_zh)
- [`while (bool) { .. }`](./../control-flow/loops_zh)
- [`assert!(bool, u64)`](./../abort-and-assert_zh)

## 所有权

与语言内置的其他标量值一样，布尔值是隐式可复制的，这意味着它们可以在没有显式指令（如[`copy`](.././variables_zh#move-and-copy)）的情况下被复制。