# 能力：Store

现在您已经了解了由[`key`](./key-ability)能力启用的顶级存储函数，我们可以讨论列表中的最后一个能力——`store`。

## 定义

`store`是一种特殊能力，允许类型被_存储_在对象中。要将类型用作具有`key`能力的结构体中的字段，需要此能力。换句话说，`store`能力允许值被_包装_在对象中。

> `store`能力还放宽了对转移操作的限制。我们在[受限和公共转移](./transfer-restrictions)部分更详细地讨论这一点。

## 示例

在前面的部分中，我们已经使用了具有`key`能力的类型：所有对象都必须有一个`UID`字段，我们在示例中使用了它；我们还使用了`Storable`类型作为`Config`结构体的一部分。`Config`类型也具有`store`能力。

```move
/// 这个类型具有`store`能力。
public struct Storable has store {}

/// Config包含一个必须具有`store`能力的`Storable`字段。
public struct Config has key, store {
    id: UID,
    stores: Storable,
}

/// MegaConfig包含一个具有`store`能力的`Config`字段。
public struct MegaConfig has key {
    id: UID,
    config: Config, // 就在这里！
}
```

## 具有`store`能力的类型

Move中的所有原生类型（除了引用）都具有`store`能力。这包括：

- [bool](./../move-basics/primitive-types#booleans)
- [无符号整数](./../move-basics/primitive-types#integer-types)
- [vector](./../move-basics/vector)
- [address](./../move-basics/address)

标准库中定义的所有类型也都具有`store`能力。这包括：

- [Option](./../move-basics/option)
- [String](./../move-basics/string)
- [TypeName](./../move-basics/type-reflection)

## 延伸阅读

- Move参考中的[类型能力](./../../reference/abilities)。