# 更好的错误处理

每当执行遇到中止时，交易失败并将中止代码返回给调用者。Move VM返回中止交易的模块名称和中止代码。这种行为对交易的调用者来说不是完全透明的，特别是当单个函数包含对可能中止的同一函数的多次调用时。在这种情况下，调用者将不知道哪个调用中止了交易，这将使调试问题或向用户提供有意义的错误消息变得困难。

```move
module book::module_a;

use book::module_b;

public fun do_something() {
    let field_1 = module_b::get_field(1); // 可能以0中止
    /* ... 大量逻辑 ... */
    let field_2 = module_b::get_field(2); // 可能以0中止
    /* ... 更多逻辑 ... */
    let field_3 = module_b::get_field(3); // 可能以0中止
}
```

上面的示例说明了单个函数包含多个可能中止的调用的情况。如果`do_something`函数的调用者收到中止代码`0`，将很难理解对`module_b::get_field`的哪个调用中止了交易。为了解决这个问题，有一些常见模式可以用来改进错误处理。

## 规则1：处理所有可能的场景

提供一个安全的"检查"函数被认为是良好的实践，该函数返回一个布尔值，指示是否可以安全地执行操作。如果`module_b`提供了一个返回布尔值指示字段是否存在的函数`has_field`，`do_something`函数可以重写如下：

```move
module book::module_a;

use book::module_b;

const ENoField: u64 = 0;

public fun do_something() {
    assert!(module_b::has_field(1), ENoField);
    let field_1 = module_b::get_field(1);
    /* ... */
    assert!(module_b::has_field(2), ENoField);
    let field_2 = module_b::get_field(2);
    /* ... */
    assert!(module_b::has_field(3), ENoField);
    let field_3 = module_b::get_field(3);
}
```

通过在每次调用`module_b::get_field`之前添加自定义检查，`module_a`的开发者控制了错误处理。这允许实现第二条规则。

## 规则2：使用不同的代码中止

第二个技巧，一旦中止代码由调用者模块处理，就是为不同场景使用不同的中止代码。这样，调用者模块可以向用户提供有意义的错误消息。`module_a`可以重写如下：

```move
module book::module_a;

use book::module_b;

const ENoFieldA: u64 = 0;
const ENoFieldB: u64 = 1;
const ENoFieldC: u64 = 2;

public fun do_something() {
    assert!(module_b::has_field(1), ENoFieldA);
    let field_1 = module_b::get_field(1);
    /* ... */
    assert!(module_b::has_field(2), ENoFieldB);
    let field_2 = module_b::get_field(2);
    /* ... */
    assert!(module_b::has_field(3), ENoFieldC);
    let field_3 = module_b::get_field(3);
}
```

现在，调用者模块可以向用户提供有意义的错误消息。如果调用者收到中止代码`0`，它可以翻译为"字段1不存在"。如果调用者收到中止代码`1`，它可以翻译为"字段2不存在"。依此类推。

## 规则3：返回`bool`而不是`assert`

开发者经常被诱惑添加一个公共函数来断言所有条件并中止执行。但是，创建一个返回布尔值的函数是更好的实践。这样，调用者模块可以处理错误并向用户提供有意义的错误消息。

```move
module book::some_app_assert;

const ENotAuthorized: u64 = 0;

public fun do_a() {
    assert_is_authorized();
    // ...
}

public fun do_b() {
    assert_is_authorized();
    // ...
}

/// 不要这样做
public fun assert_is_authorized() {
    assert!(/* 某些条件 */ true, ENotAuthorized);
}
```

这个模块可以重写如下：

```move
module book::some_app;

const ENotAuthorized: u64 = 0;

public fun do_a() {
    assert!(is_authorized(), ENotAuthorized);
    // ...
}

public fun do_b() {
    assert!(is_authorized(), ENotAuthorized);
    // ...
}

public fun is_authorized(): bool {
    /* 某些条件 */ true
}

// 当在多个地方使用相同条件和相同中止代码时，
// 私有函数仍可用来避免代码重复
fun assert_is_authorized() {
    assert!(is_authorized(), ENotAuthorized);
}
```

使用这三条规则将使错误处理对交易调用者更加透明，并允许其他开发者在其模块中使用自定义中止代码。