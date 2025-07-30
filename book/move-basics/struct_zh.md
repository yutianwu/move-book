# 使用结构体定义自定义类型

Move 的类型系统在定义自定义类型方面表现出色。用户定义的类型可以根据应用程序的特定需求进行定制，不仅在数据层面，还在其行为方面。在本节中，我们介绍结构体的定义以及如何使用它。

## 结构体

要定义自定义类型，您可以使用 `struct` 关键字，后跟类型的名称。在名称之后，您可以定义结构体的字段。每个字段都使用 `field_name: field_type` 语法定义。字段定义必须用逗号分隔。字段可以是任何类型，包括其他结构体。

> Move 不支持递归结构体，这意味着结构体不能包含自身作为字段。

```move file=packages/samples/sources/move-basics/struct.move anchor=def

```

在上面的例子中，我们定义了一个有五个字段的 `Record` 结构体。`title` 字段是 `String` 类型，`artist` 字段是 `Artist` 类型，`year` 字段是 `u16` 类型，`is_debut` 字段是 `bool` 类型，`edition` 字段是 `Option<u16>` 类型。`edition` 字段是 `Option<u16>` 类型，用来表示版本是可选的。

结构体默认是私有的，这意味着它们不能被导入并在定义它们的模块之外使用。它们的字段也是私有的，不能从模块外部访问。有关不同可见性修饰符的更多信息，请参阅[可见性](./visibility)。

> 结构体的字段是私有的，只能由定义结构体的模块访问。在其他模块中读取和写入结构体的字段，只有在定义结构体的模块提供访问字段的公共函数时才可能。

## 创建和使用实例

我们描述了结构体的_定义_。现在让我们看看如何初始化结构体并使用它。结构体可以使用 `struct_name { field1: value1, field2: value2, ... }` 语法进行初始化。字段可以按任何顺序初始化，并且必须设置所有必需的字段。

```move file=packages/samples/sources/move-basics/struct.move anchor=pack

```

在上面的例子中，我们创建了一个 `Artist` 结构体的实例，并将 `name` 字段设置为字符串 "The Beatles"。

要访问结构体的字段，您可以使用 `.` 操作符，后跟字段名称。

```move file=packages/samples/sources/move-basics/struct.move anchor=access

```

只有定义结构体的模块才能访问其字段（可变和不可变）。所以上面的代码应该与 `Artist` 结构体在同一个模块中。

<!-- ## Accessing Fields

Struct fields are private and can be accessed only by the module defining the struct. To access the fields of a struct, you can use the `.` operator followed by the field name.

```move
# anchor: access file=packages/samples/sources/move-basics/struct.move anchor=access
```
-->

## 解包结构体

结构体默认是不可丢弃的，这意味着初始化的结构体值必须被使用，要么通过存储它，要么通过解包它。解包结构体意味着将其解构为其字段。这使用 `let` 关键字，后跟结构体名称和字段名称来完成。

```move file=packages/samples/sources/move-basics/struct.move anchor=unpack

```

在上面的例子中，我们解包了 `Artist` 结构体，并用 `name` 字段的值创建了一个新变量 `name`。因为变量没有被使用，编译器会发出警告。要抑制警告，您可以使用下划线 `_` 来表示变量是故意未使用的。

```move file=packages/samples/sources/move-basics/struct.move anchor=unpack_ignore

```

## 进一步阅读

- Move 参考手册中的[结构体](./../../reference/structs)。