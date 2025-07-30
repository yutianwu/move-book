# 可见性修饰符

每个模块成员都有可见性。默认情况下，所有模块成员都是_私有的_ - 意味着它们只能在定义它们的模块内访问。然而，您可以添加可见性修饰符来使模块成员_公共的_ - 在模块外部可见，或者_公共(包)_ - 在同一包内的模块中可见，或者_入口_ - 可以从交易中调用但不能从其他模块调用。

## 内部可见性

在模块中定义的没有可见性修饰符的函数或结构体对模块是_私有的_。它不能从其他模块调用。

```move
module book::internal_visibility;

// 这个函数可以从同一模块中的其他函数调用
fun internal() { /* ... */ }

// 同一模块 -> 可以调用 internal()
fun call_internal() {
    internal();
}
```

以下代码将不会编译：

<!-- TODO: add failure flag to example -->

```move
module book::try_calling_internal;

use book::internal_visibility;

// 不同模块 -> 不能调用 internal()
fun try_calling_internal() {
    internal_visibility::internal();
}
```

请注意，仅仅因为结构体字段在 Move 中不可见，并不意味着其值保持机密 —— 总是可以从 Move 外部读取链上对象的内容。您永远不应该在对象内存储未加密的秘密。

## 公共可见性

结构体或函数可以通过在 `fun` 或 `struct` 关键字之前添加 `public` 关键字来变为_公共的_。

```move
module book::public_visibility;

// 这个函数可以从其他模块调用
public fun public_fun() { /* ... */ }
```

公共函数可以从其他模块导入和调用。以下代码将编译：

```move
module book::try_calling_public;

use book::public_visibility;

// 不同模块 -> 可以调用 public_fun()
fun try_calling_public() {
    public_visibility::public_fun();
}
```

与某些语言不同，结构体字段不能设为公共。

## 包可见性

具有_包_可见性的函数可以从同一包内的任何模块调用，但不能从其他包中的模块调用。换句话说，它对包是_内部的_。

```move
module book::package_visibility;

public(package) fun package_only() { /* ... */ }
```

包函数可以从同一包内的任何模块调用：

```move
module book::try_calling_package;

use book::package_visibility;

// 同一包 `book` -> 可以调用 package_only()
fun try_calling_package() {
    package_visibility::package_only();
}
```

## 原生函数

[框架](./../programmability/sui-framework)和[标准库](./standard-library)中的一些函数标有 `native` 修饰符。这些函数由 Move VM 原生提供，在 Move 源代码中没有函数体。要了解更多关于原生修饰符的信息，请参考 [Move 参考手册](./../../reference/functions?highlight=native#native-functions)。

```move
module std::type_name;

public native fun get<T>(): TypeName;
```

这是来自 `std::type_name` 的一个例子，在[反射章节](./type-reflection)中了解更多关于这个模块的信息。

## 进一步阅读

- Move 参考手册中的[可见性](./../../reference/functions#visibility)。