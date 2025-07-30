# 交易

交易是区块链世界中的基本概念。它是与区块链交互的一种方式。交易用于改变区块链的状态，它们是唯一的方式。在Move中，交易用于调用包中的函数、部署新包和升级现有包。

<!--

- how user interacts with a program
    - mention public functions
    - give a concept of an entry / public function without getting into details
    - mention that functions are called in transactions
    - mention that transactions are sent by accounts
    - every transaction specifies object it operates on

 -->

## 交易结构

> 每个交易都明确指定它操作的对象！

交易包含：

- 发送者 - [签名](./what-is-an-account)交易的账户；
- 命令列表（或链）- 要执行的操作；
- 命令输入 - 命令的参数：要么是`pure` - 数字或字符串等简单值，要么是`object` - 交易将访问的对象；
- gas对象 - 用于支付交易费用的`Coin`对象；
- gas价格和预算 - 交易的成本；

## 输入

交易输入是交易的参数，分为2种类型：

- 纯参数：这些主要是[原始类型](../move-basics/primitive-types)，还有一些额外的补充。纯参数可以是：
  - [`bool`](../move-basics/primitive-types#booleans)。
  - [无符号整数](../move-basics/primitive-types#integer-types)（`u8`、`u16`、`u32`、`u64`、`u128`、`u256`）。
  - [`address`](../move-basics/address)。
  - [`std::string::String`](../move-basics/string)，UTF8字符串。
  - [`std::ascii::String`](../move-basics/string#ascii-strings)，ASCII字符串。
  - [`vector<T>`](../move-basics/vector)，其中`T`是纯类型。
  - [`std::option::Option<T>`](../move-basics/option)，其中`T`是纯类型。
  - [`std::object::ID`](../storage/uid-and-id)，通常指向对象。另请参阅[什么是对象](../object/object-model)。
- 对象参数：这些是交易将访问的对象或对象引用。对象参数需要是共享对象、冻结对象或交易发送者拥有的对象，交易才能成功。更多信息请参阅[对象模型](../object)。

## 命令

Sui交易可能包含多个命令。每个命令都是单个内置命令（如发布包）或对已发布包中函数的调用。命令按它们在交易中列出的顺序执行，它们可以使用前面命令的结果，形成链。交易要么作为整体成功，要么失败。

从示意上看，交易如下所示（伪代码）：

```
Inputs:
- sender = 0xa11ce

Commands:
- payment = SplitCoins(Gas, [ 1000 ])
- item = MoveCall(0xAAA::market::purchase, [ payment ])
- TransferObjects(item, sender)
```

在此示例中，交易包含三个命令：

1. `SplitCoins` - 一个内置命令，从传递的对象中分离出新硬币，在这种情况下是`Gas`对象；
2. `MoveCall` - 一个命令，使用给定参数调用包`0xAAA`、模块`market`中的函数`purchase` - `payment`对象；
3. `TransferObjects` - 一个内置命令，将对象转移给接收者。

<!--
> There are multiple different implementations of transaction building, for example
-->

## 交易效果

交易效果是交易对区块链状态所做的更改。更具体地说，交易可以通过以下方式改变状态：

- 使用gas对象支付交易费用；
- 创建、更新或删除对象；
- 发出事件；

执行交易的结果包含不同部分：

- 交易摘要 - 用于标识交易的交易哈希；
- 交易数据 - 交易中使用的输入、命令和gas对象；
- 交易效果 - 交易的状态和"效果"，更具体地说：交易的状态、对象的更新及其新版本、使用的gas对象、交易的gas成本以及交易发出的事件；
- 事件 - 交易发出的自定义[事件](./../programmability/events)；
- 对象更改 - 对对象所做的更改，包括_所有权的更改_；
- 余额更改 - 对参与交易的账户的聚合余额所做的更改；