# 动态字段

Sui对象模型允许对象作为_动态字段_附加到其他对象上。这种行为类似于其他编程语言中`Map`的工作方式。但是，与在Move中严格类型化的`Map`（我们在[集合](./collections)部分已经介绍过）不同，动态字段允许附加任何类型的对象。前端开发世界中的类似方法是JavaScript对象类型，它允许动态存储任何类型的数据。

> 可以附加到对象的动态字段数量没有限制。因此，动态字段可用于存储不适合对象限制大小的大量数据。

动态字段允许广泛的应用，从将数据分割成更小的部分以避免[对象大小限制](./../guides/building-against-limits)到作为应用程序逻辑的一部分附加对象。

## 定义

动态字段在[Sui框架](./sui-framework)的`sui::dynamic_field`模块中定义。它们通过_名称_附加到对象的`UID`，并可以使用该名称访问。只能有一个具有给定名称的字段附加到对象。

```move
module sui::dynamic_field;

/// Internal object used for storing the field and value
public struct Field<Name: copy + drop + store, Value: store> has key {
    /// Determined by the hash of the object ID, the field name
    /// value and it's type, i.e. hash(parent.id || name || Name)
    id: UID,
    /// The value for the name of this field
    name: Name,
    /// The value bound to this field
    value: Value,
}
```

如定义所示，动态字段存储在内部`Field`对象中，该对象的`UID`基于对象ID、字段名称和字段类型以确定性方式生成。`Field`对象包含字段名称和绑定到它的值。`Name`和`Value`类型参数上的约束定义了键和值必须具有的能力。

## 使用

动态字段可用的方法很直接：可以使用`add`添加字段，使用`remove`删除字段，使用`borrow`和`borrow_mut`读取字段。此外，可以使用`exists_`方法检查字段是否存在（对于带类型的更严格检查，有`exists_with_type`方法）。

```move file=packages/samples/sources/programmability/dynamic-fields.move anchor=usage

```

在上面的示例中，我们定义了一个`Character`对象和两种不同类型的配件，它们永远不能放在同一个向量中。但是，动态字段允许我们将它们一起存储在单个对象中。两个对象都通过`vector<u8>`（字节字符串字面量）附加到`Character`，并可以使用它们各自的名称访问。

如您所见，当我们将配件附加到Character时，我们是_按值_传递它们的。换句话说，两个值都被移动到新的作用域，它们的所有权被转移到`Character`对象。如果我们改变了`Character`对象的所有权，配件也会随之移动。

我们应该强调的动态字段的最后一个重要属性是它们_通过其父对象访问_。这意味着`Hat`和`Mustache`对象不能直接访问，并遵循与父对象相同的规则。

## 外部类型作为动态字段

动态字段允许对象携带任何类型的数据，包括在其他模块中定义的数据。这是可能的，因为它们的通用性质和对类型参数相对较弱的约束。让我们通过将几个不同的值附加到`Character`对象来说明这一点。

```move file=packages/samples/sources/programmability/dynamic-fields.move anchor=foreign_types

```

在这个示例中，我们展示了如何将不同类型用于动态字段的_名称_和_值_。`String`通过`vector<u8>`名称附加，`u64`通过`u32`名称附加，`bool`通过`bool`名称附加。动态字段可以做任何事情！

## 孤立的动态字段

> 为了防止孤立的动态字段，请使用[动态集合类型](./dynamic-collections)，如`Bag`，因为它们跟踪动态字段，如果有附加字段则不允许解包。

用于删除UID的`object::delete()`函数不跟踪动态字段，无法防止动态字段变为孤立状态。一旦删除父UID，动态字段不会自动删除，它们就会变为孤立状态。这意味着动态字段仍然存储在区块链中，但它们将永远无法再次访问。

```move file=packages/samples/sources/programmability/dynamic-fields.move anchor=orphan_fields

```

孤立对象不受存储退款的影响，存储费用将无法申请。在解包对象期间避免孤立动态字段的一种方法是返回`UID`并将其临时存储在某处，直到动态字段被删除并正确处理。

## 自定义类型作为字段名称

在上面的示例中，我们使用原始类型作为字段名称，因为它们具有所需的能力集。但是当我们使用自定义类型作为字段名称时，动态字段变得更加有趣。这允许更结构化的数据存储方式，也允许保护字段名称不被其他模块访问。

```move file=packages/samples/sources/programmability/dynamic-fields.move anchor=custom_type

```

我们上面定义的两个字段名称是`AccessoryKey`和`MetadataKey`。`AccessoryKey`中有一个`String`字段，因此可以使用不同的`name`值多次使用。`MetadataKey`是一个空键，只能附加一次。

```move file=packages/samples/sources/programmability/dynamic-fields.move anchor=custom_type_usage

```

如您所见，自定义类型确实可以作为字段名称工作，但只要它们可以被模块_构造_，换句话说 - 如果它们是模块的_内部_并在其中定义。结构体打包的这种限制可以在应用程序设计中开辟新的方式。

这种方法在对象能力<!--[]](./object-capability)-->模式中使用，应用程序可以授权外部对象在其中执行操作，而不向其他模块暴露能力。

## 暴露UID

<div class="warning">

对`UID`的可变访问是一个安全风险。将您类型的`UID`暴露为可变引用可能导致不想要的修改或删除对象的动态字段。此外，它会影响对象传输<!--[](./../storage/transfer-to-object)-->和[动态对象字段](./dynamic-object-fields)。在将`UID`暴露为可变引用之前，请确保了解其含义。

</div>

因为动态字段附加到`UID`，它们在其他模块中的使用取决于是否可以访问`UID`。默认情况下，结构体可见性保护`id`字段，不让其他模块直接访问它。但是，如果有一个返回`UID`引用的公共访问器方法，动态字段可以在其他模块中读取。

```move file=packages/samples/sources/programmability/dynamic-fields.move anchor=exposed_uid

```

在上面的示例中，我们展示了如何暴露`Character`对象的`UID`。这个解决方案可能适用于某些应用程序，但是重要的是要记住，暴露的`UID`允许读取附加到对象的_任何_动态字段。

如果您需要仅在包内暴露`UID`，请使用限制性可见性，如`public(package)`，或者更好 - 使用更具体的访问器方法，只允许读取特定字段。

```move file=packages/samples/sources/programmability/dynamic-fields.move anchor=exposed_uid_measures

```

## 动态字段 vs 字段

动态字段比常规字段更昂贵，因为它们需要额外的存储和访问成本。它们的灵活性是有代价的，在决定使用动态字段还是常规字段时，了解其含义很重要。

## 限制

动态字段不受[对象大小限制](./../guides/building-against-limits)的约束，可用于存储大量数据。但是，它们仍然受到[动态字段创建限制](./../guides/building-against-limits)的约束，每个交易设置为1000个字段。

## 应用

动态字段可以在任何复杂度的应用程序中发挥关键作用。它们开辟了各种不同的用例，从存储异构数据到作为应用程序逻辑的一部分附加对象。它们允许基于_稍后_定义它们和更改字段类型的能力的某些[可升级性实践](./../guides/upgradeability-practices)。

## 下一步

在下一节中，我们将介绍[动态对象字段](./dynamic-object-fields)，并解释它们与动态字段的区别，以及使用它们的含义。