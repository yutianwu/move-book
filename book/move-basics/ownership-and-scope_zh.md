# 所有权和作用域

Move 中的每个变量都有作用域和所有者。作用域是变量有效的代码范围，所有者是该变量所属的作用域。一旦所有者作用域结束，变量就会被丢弃。这是 Move 中的基本概念，理解它的工作原理很重要。

<!--

- Borrow Checker
- Mention Rust's borrow checker
- Borrowing / References intro

-->

## 所有权

在函数作用域中定义的变量由该作用域拥有。运行时遍历函数作用域并执行每个表达式和语句。函数作用域结束后，在其中定义的变量会被丢弃或释放。

```move
module book::ownership;

public fun owner() {
    let a = 1; // a 由 `owner` 函数拥有
} // a 在这里被丢弃

public fun other() {
    let b = 2; // b 由 `other` 函数拥有
} // b 在这里被丢弃

#[test]
fun test_owner() {
    owner();
    other();
    // a 和 b 在这里都无效
}
```

在上面的例子中，变量 `a` 由 `owner` 函数拥有，变量 `b` 由 `other` 函数拥有。当调用这些函数时，变量被定义，当函数结束时，变量被丢弃。

## 返回值

如果我们将 `owner` 函数更改为返回变量 `a`，那么 `a` 的所有权将转移给函数的调用者。

```move
module book::ownership;

public fun owner(): u8 {
    let a = 1; // a 在这里定义
    a // 作用域结束，a 被返回
}

#[test]
fun test_owner() {
    let a = owner();
    // a 在这里有效
} // a 在这里被丢弃
```

## 按值传递

另外，如果我们将变量 `a` 传递给另一个函数，`a` 的所有权将转移给该函数。执行此操作时，我们将值从一个作用域_移动_到另一个作用域。这也称为_移动语义_。

```move
module book::ownership;

public fun owner(): u8 {
    let a = 10;
    a
} // a 被返回

public fun take_ownership(v: u8) {
    // v 由 `take_ownership` 拥有
} // v 在这里被丢弃

#[test]
fun test_owner() {
    let a = owner();
    // `u8` 是可复制的，在调用函数时传递 `move a` 来强制转移其所有权
    take_ownership(move a);
    // a 在这里无效
}
```

## 带有块的作用域

每个函数都有一个主作用域，它还可以通过使用块来拥有子作用域。块是语句和表达式的序列，它有自己的作用域。在块中定义的变量由该块拥有，当块结束时，变量被丢弃。

```move
module book::ownership;

public fun owner() {
    let a = 1; // a 由 `owner` 函数的作用域拥有
    {
        let b = 2; // 声明 b 的块拥有它
        {
            let c = 3; // 声明 c 的块拥有它
        }; // c 在这里被丢弃
    }; // b 在这里被丢弃
    // a = b; // 错误：b 在这里无效
    // a = c; // 错误：c 在这里无效
} // a 在这里被丢弃
```

但是，如果我们从块中返回一个值，变量的所有权将转移给块的调用者。

```move
module book::ownership;

public fun owner(): u8 {
    let a = 1; // a 由 `owner` 函数的作用域拥有
    let b = {
        let c = 2; // 声明 c 的块拥有它
        c // c 从块中返回并转移给 b
    };
    a + b // a 和 b 在这里都有效
}
```

## 可复制类型

Move 中的某些类型是_可复制的_，这意味着它们可以在不转移所有权的情况下被复制。这对于小且复制成本低的类型（如整数和布尔值）很有用。当这些类型传递给函数或从函数返回时，或者当它们被_移动_到另一个作用域然后在原始作用域中访问时，Move 编译器会自动复制这些类型。

## 扩展阅读

- Move 参考中的[局部变量和作用域](./../../reference/variables)。