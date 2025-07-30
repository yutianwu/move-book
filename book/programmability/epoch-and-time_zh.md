# 时期和时间

Sui有两种访问当前时间的方式：`Epoch`和`Time`。前者表示系统中的操作周期，大约每24小时变化一次。后者表示自Unix纪元以来的当前时间（以毫秒为单位）。两者都可以在程序中自由访问。

## 时期

时期用于将系统分为操作周期。在一个时期内，验证者集合是固定的，但是，在时期边界，验证者集合可以改变。时期在共识算法中起着关键作用，用于确定当前的验证者集合。它们也被用作质押机制中的测量单位。

可以从[交易上下文](./transaction-context)中读取时期：

```move file=packages/samples/sources/programmability/epoch-and-time.move anchor=epoch

```

也可以获取时期开始的unix时间戳：

```move file=packages/samples/sources/programmability/epoch-and-time.move anchor=epoch_start

```

通常，时期用于质押和系统操作，但是，在自定义场景中，它们可以用来模拟24小时周期。如果应用程序依赖于质押逻辑或需要知道当前验证者集合，它们是关键的。

## 时间

为了更精确的时间测量，Sui提供了`Clock`对象。它是一个由系统在检查点期间更新的系统对象，存储自Unix纪元以来的当前时间（以毫秒为单位）。`Clock`对象在`sui::clock`模块中定义，具有保留地址`0x6`。

Clock是一个共享对象，但尝试可变访问它的交易将失败。这种限制允许对`Clock`对象的并行访问，这对于维护性能很重要。

```move
module sui::clock;

/// 向Move调用暴露时间的单例共享对象。此对象位于地址0x6，
/// 只能通过入口函数进行读取（通过不可变引用访问）。
///
/// 尝试通过可变引用或值接受`Clock`的入口函数将无法验证，
/// 诚实的验证者不会签署或执行使用`Clock`作为输入参数的交易，
/// 除非它通过不可变引用传递。
public struct Clock has key {
    id: UID,
    /// 时钟的时间戳，每次共识提交计划时由系统交易自动设置，
    /// 或在测试期间由`sui::clock::increment_for_testing`设置。
    timestamp_ms: u64,
}
```

`Clock`模块中只有一个公共函数可用 - `timestamp_ms`。它返回自Unix纪元以来的当前时间（以毫秒为单位）。

```move file=packages/samples/sources/programmability/epoch-and-time.move anchor=clock

```

<!-- TODO:

## Testing

TODO: how to use Clock in tests. -->