# 常量

<!--

Chapter: Basic Syntax
Goal: Introduce constants.
Notes:
    - constants are immutable
    - constants are private
    - start with a capital letter always
    - stored in the bytecode (but w/o a name)
    - mention standard for naming constants

Links:
    - next section (abort and assert)
    - coding conventions (constants)
    - constants (language reference)

 -->

常量是在模块级别定义的不可变值。它们通常用作为整个模块中使用的静态值命名的方式。例如，如果有产品的默认价格，你可能会为它定义一个常量。常量存储在模块的字节码中，每次使用时，值都会被复制。

```move file=packages/samples/sources/move-basics/constants-shop-price.move anchor=shop_price

```

## 命名约定

常量必须以大写字母开头——这在编译器级别强制执行。对于用作值的常量，约定是使用全大写字母和单词之间的下划线，这使常量在代码中的其他标识符中脱颖而出。[错误常量](./assert-and-abort#error-constants)是一个例外，它们使用 ECamelCase 编写。

```move file=packages/samples/sources/move-basics/constants-naming.move anchor=naming

```

## 常量是不可变的

常量不能被更改和赋予新值。作为包字节码的一部分，它们本质上是不可变的。

```move
module book::immutable_constants;

const ITEM_PRICE: u64 = 100;

// 发出错误
fun change_price() {
    ITEM_PRICE = 200;
}
```

## 使用配置模式

应用程序的一个常见用例是定义在整个代码库中使用的一组常量。但是由于常量对模块是私有的，它们不能从其他模块访问。解决这个问题的一种方法是定义一个导出常量的"配置"模块。

```move file=packages/samples/sources/move-basics/constants-config.move anchor=config

```

这样其他模块可以导入和读取常量，并且更新过程得到简化。如果常量需要更改，在包升级期间只需要更新配置模块。

## 链接

- Move 参考中的[常量](./../../reference/constants)
- [常量的编码约定](./../guides/code-quality-checklist#regular-constant-are-all_caps)