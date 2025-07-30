# 结构体方法

Move 编译器支持_接收者语法_ `e.f()`，它允许定义可以在结构体实例上调用的方法。术语"接收者"特指接收方法调用的实例。这类似于其他编程语言中的方法语法。这是一种定义操作结构体字段的函数的便捷方式，提供对结构体字段的直接访问，并创建比将结构体作为参数传递更清洁、更直观的代码。

## 方法语法

如果函数的第一个参数是定义该函数的模块内部的结构体，那么该函数可以使用 `.` 操作符调用。然而，如果第一个参数的类型在另一个模块中定义，那么方法默认不会与结构体关联。在这种情况下，`.` 操作符语法不可用，必须使用标准函数调用语法调用函数。

当导入模块时，其方法会自动与结构体关联。

```move file=packages/samples/sources/move-basics/struct-methods.move anchor=hero

```

## 方法别名

方法别名有助于避免模块定义多个结构体及其方法时的名称冲突。它们还可以为结构体提供更具描述性的方法名称。

语法如下：

```move
// 用于本地方法关联
use fun function_path as Type.method_name;

// 导出的别名
public use fun function_path as Type.method_name;
```

> 公共别名仅允许用于在同一模块中定义的结构体。对于在其他模块中定义的结构体，仍然可以创建别名，但不能设为公共。

在下面的例子中，我们更改了 `hero` 模块并添加了另一个类型 - `Villain`。`Hero` 和 `Villain` 都有相似的字段名和方法。为了避免名称冲突，我们分别为方法添加了 `hero_` 和 `villain_` 前缀。然而，使用别名允许这些方法在结构体实例上调用时不需要前缀：

```move file=packages/samples/sources/move-basics/struct-methods-2.move anchor=hero_and_villain

```

在测试函数中，`health` 方法直接在 `Hero` 和 `Villain` 实例上调用，不需要前缀，因为编译器自动将方法与各自的结构体关联。

> 注意：在测试函数中，`hero.health()` 调用的是别名方法，而不是直接访问私有的 `health` 字段。虽然 `Hero` 和 `Villain` 结构体是公共的，但它们的字段对模块仍然是私有的。方法调用 `hero.health()` 使用由 `public use fun hero_health as Hero.health` 定义的公共别名，这提供了对私有字段的受控访问。

<!-- ## Aliasing an external module's method

It is also possible to associate a function defined in another module with a struct from the current
module. Following the same approach, we can create an alias for the method defined in another
module. Let's use the `bcs::to_bytes` method from the [Standard Library](./standard-library) and
associate it with the `Hero` struct. It will allow serializing the `Hero` struct to a vector of
bytes.

```move file=packages/samples/sources/move-basics/struct-methods-3.move anchor=hero_to_bytes
``` -->

## 进一步阅读

- Move 参考手册中的[方法语法](./../../reference/method-syntax)。