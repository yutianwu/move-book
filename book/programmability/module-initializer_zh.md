# 模块初始化器

许多应用程序中的常见用例是在包发布时只运行一次某些代码。想象一个简单的商店模块，需要在发布时创建主Store对象。在Sui中，这是通过在模块内定义`init`函数来实现的。这个函数将在模块发布时自动调用。

> 所有模块的`init`函数都在发布过程中被调用。目前，此行为仅限于发布命令，不扩展到包升级。
>
> <!-- [package upgrades]() -->

```move file=packages/samples/sources/programmability/module-initializer.move anchor=main

```

在同一个包中，另一个模块可以有自己的`init`函数，封装不同的逻辑。

```move file=packages/samples/sources/programmability/module-initializer-2.move anchor=other

```

## `init`功能特性

如果函数存在于模块中并遵循规则，则在发布时调用该函数：

- 函数必须命名为`init`，是私有的，没有返回值。
- 接受一个或两个参数：[一次性见证](./one-time-witness)（可选）和[TxContext](./transaction-context)。其中`TxContext`始终是最后一个参数。

```move
fun init(ctx: &mut TxContext) { /* ... */}
fun init(otw: OTW, ctx: &mut TxContext) { /* ... */ }
```

TxContext也可以作为不可变引用传递：`&TxContext`。但是，实际上，它应该始终是`&mut TxContext`，因为`init`函数无法访问链上状态，要创建新对象，它需要对上下文的可变引用。

```move
fun init(ctx: &TxContext) { /* ... */}
fun init(otw: OTW, ctx: &TxContext) { /* ... */ }
```

## 信任和安全

虽然`init`函数可以用来一次性创建敏感对象，但重要的是要知道相同的对象（例如第一个示例中的`StoreOwnerCap`）仍然可以在另一个函数中创建。特别是考虑到在升级过程中可以向模块添加新函数。因此，`init`函数是设置模块初始状态的好地方，但它本身不是安全措施。

有方法可以保证对象只创建一次，例如[一次性见证](./one-time-witness)。还有方法可以限制或禁用模块的升级，我们将在包升级章节中介绍。

## 下一步

从定义可以看出，`init`函数保证在模块发布时只调用一次。因此，它是放置初始化模块对象并设置环境和配置的代码的好地方。

例如，如果有一个某些操作所需的[能力](./capability)，它应该在`init`函数中创建。在下一章中，我们将更详细地讨论`Capability`模式。