# 事件

事件是一种通知链下监听器有关链上事件的方式。它们用于发出关于交易的附加信息，这些信息不被存储 - 因此，无法在链上访问。事件由位于[Sui框架](./sui-framework)中的`sui::event`模块发出。

> 任何具有[copy](./../move-basics/copy-ability)和[drop](./../move-basics/drop-ability)能力的自定义类型都可以作为事件发出。Sui验证器要求类型为模块内部类型。

```move
module sui::event;

/// 发出自定义Move事件，将数据发送到链下。
///
/// 用于创建自定义索引和以最适合特定应用程序的方式
/// 跟踪链上活动。
///
/// 类型`T`是索引事件的主要方式，可以包含
/// 幻影参数，例如`emit(MyEvent<phantom T>)`。
public native fun emit<T: copy + drop>(event: T);
```

## 发出事件

事件使用`sui::event`模块中的`emit`函数发出。该函数接受一个参数 - 要发出的事件。事件数据按值传递，

```move file=packages/samples/sources/programmability/events.move anchor=emit

```

Sui验证器要求传递给`emit`函数的类型为_模块内部类型_。因此，发出来自另一个模块的类型将导致编译错误。原始类型，虽然它们符合_copy_和_drop_要求，但不允许作为事件发出。

## 事件结构

事件是交易结果的一部分，存储在_交易效果_中。因此，它们本身具有`sender`字段，该字段是发送交易的地址。因此，向事件添加"sender"字段是不必要的。类似地，事件元数据包含时间戳。但重要的是要注意，时间戳是相对于节点的，可能会因节点而异。

<!-- ## Reliability -->