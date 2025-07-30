# 代码质量检查清单

Move语言及其生态系统的快速发展使许多旧的实践过时。本指南作为开发者审查代码的检查清单，确保其符合Move开发的当前最佳实践。请仔细阅读并尽可能多地将建议应用到您的代码中。

## 代码组织

本指南中提到的一些问题可以通过使用[Move Formatter](https://www.npmjs.com/package/@mysten/prettier-plugin-move)作为CLI工具、[作为CI检查](https://github.com/marketplace/actions/move-formatter)或[作为VSCode (Cursor)插件](https://marketplace.visualstudio.com/items?itemName=mysten.prettier-move)来修复。

## 包清单

### 使用正确的版本

本指南中的所有功能都需要Move 2024版本，必须在包清单中指定。

```toml
[package]
name = "my_package"
edition = "2024.beta" # 或者（仅）"2024"
```

### 隐式框架依赖

从Sui 1.45开始，您不再需要在`Move.toml`中指定框架依赖：

```toml
# 旧版本，1.45之前
[dependencies]
Sui = { ... }

# 现代版本，Sui、Bridge、MoveStdlib和SuiSystem隐式导入！
[dependencies]
```

### 前缀命名地址

如果您的包有通用名称（例如`token`）——特别是如果您的项目包含多个包——确保为命名地址添加前缀：

```toml
# 不好！不能表明任何内容，可能冲突
[addresses]
math = "0x0"

# 好！清楚地说明项目，不太可能冲突
[addresses]
my_protocol_math = "0x0"
```

## 导入、模块和常量

### 使用模块标签

```move
// 不好：增加缩进，遗留样式
module my_package::my_module {
    public struct A {}
}

// 好！
module my_package::my_module;

public struct A {}
```

### `use`语句中没有单独的`Self`

```move
// 正确，成员 + self导入
use my_package::other::{Self, OtherMember};

// 不好！`{Self}`是多余的
use my_package::my_module::{Self};

// 好！
use my_package::my_module;
```

### 将`use`语句与`Self`分组

```move
// 不好！
use my_package::my_module;
use my_package::my_module::OtherMember;

// 好！
use my_package::my_module::{Self, OtherMember};
```

### 错误常量采用`EPascalCase`

```move
// 不好！全大写用于常规常量
const NOT_AUTHORIZED: u64 = 0;

// 好！清楚地表明这是错误常量
const ENotAuthorized: u64 = 0;
```

### 常规常量采用`ALL_CAPS`

```move
// 不好！PascalCase与错误常量相关
const MyConstant: vector<u8> = b"my const";

// 好！清楚地表明这是常量值
const MY_CONSTANT: vector<u8> = b"my const";
```

## 结构

### 能力以`Cap`为后缀

```move
// 不好！如果它是能力，添加`Cap`后缀
public struct Admin has key, store {
    id: UID,
}

// 好！审查者知道从类型中期待什么
public struct AdminCap has key, store {
    id: UID,
}
```

### 名称中没有`Potato`

```move
// 不好！它没有能力，我们已经知道它是Hot-Potato类型
public struct PromisePotato {}

// 好！
public struct Promise {}
```

### 事件应该用过去时命名

```move
// 不好！不清楚这个结构做什么
public struct RegisterUser has copy, drop { user: address }

// 好！清楚，这是一个事件
public struct UserRegistered has copy, drop { user: address }
```

### 对动态字段键使用位置结构 + `Key`后缀

```move
// 不算太坏，但违背了规范样式
public struct DynamicField has copy, drop, store {}

// 好！规范样式，Key后缀
public struct DynamicFieldKey() has copy, drop, store;
```

## 函数

### 没有`public entry`，只有`public`或`entry`

```move
// 不好！函数在交易中可调用不需要entry
public entry fun do_something() { /* ... */ }

// 好！public函数更宽松，可以返回值
public fun do_something_2(): T { /* ... */ }
```

### 为PTB编写可组合函数

```move
// 不好！不可组合，更难测试！
public fun mint_and_transfer(ctx: &mut TxContext) {
    /* ... */
    transfer::transfer(nft, ctx.sender());
}

// 好！可组合！
public fun mint(ctx: &mut TxContext): NFT { /* ... */ }

// 好！故意不可组合
entry fun mint_and_keep(ctx: &mut TxContext) { /* ... */ }
```

### 对象排在前面（除了Clock）

```move
// 不好！难以阅读！
public fun call_app(
    value: u8,
    app: &mut App,
    is_smth: bool,
    cap: &AppCap,
    clock: &Clock,
    ctx: &mut TxContext,
) { /* ... */ }

// 好！
public fun call_app(
    app: &mut App,
    cap: &AppCap,
    value: u8,
    is_smth: bool,
    clock: &Clock,
    ctx: &mut TxContext,
) { /* ... */ }
```

### 能力排在第二位

```move
// 不好！破坏方法关联性
public fun authorize_action(cap: &AdminCap, app: &mut App) { /* ... */ }

// 好！保持Cap在签名中可见并维护`.calls()`
public fun authorize_action(app: &mut App, cap: &AdminCap) { /* ... */ }
```

### Getter以字段命名 + `_mut`

```move
// 不好！不必要的`get_`
public fun get_name(u: &User): String { /* ... */ }

// 好！清楚地表明它访问字段`name`
public fun name(u: &User): String { /* ... */ }

// 好！对于可变引用使用`_mut`
public fun details_mut(u: &mut User): &mut Details { /* ... */ }
```

## 函数体：结构方法

### 常见Coin操作

```move
// 不好！遗留代码，难以阅读！
let paid = coin::split(&mut payment, amount, ctx);
let balance = coin::into_balance(paid);

// 好！结构方法让它更容易！
let balance = payment.split(amount, ctx).into_balance();

// 更好（在此示例中 - 无需创建临时coin）
let balance = payment.balance_mut().split(amount);

// 也可以这样做！
let coin = balance.into_coin(ctx);
```

### 不要导入`std::string::utf8`

```move
// 不好！不幸的是，非常常见！
use std::string::utf8;

let str = utf8(b"hello, world!");

// 好！
let str = b"hello, world!".to_string();

// 另外，对于ASCII字符串
let ascii = b"hello, world!".to_ascii_string();
```

### UID有`delete`

```move
// 不好！
object::delete(id);

// 好！
id.delete();
```

### `ctx`有`sender()`

```move
// 不好！
tx_context::sender(ctx);

// 好！
ctx.sender()
```

### Vector有字面量。和关联函数

```move
// 不好！
let mut my_vec = vector::empty();
vector::push_back(&mut my_vec, 10);
let first_el = vector::borrow(&my_vec);
assert!(vector::length(&my_vec) == 1);

// 好！
let mut my_vec = vector[10];
let first_el = my_vec[0];
assert!(my_vec.length() == 1);
```

### 集合支持索引语法

```move
let x: VecMap<u8, String> = /* ... */;

// 不好！
x.get(&10);
x.get_mut(&10);

// 好！
&x[&10];
&mut x[&10];
```

## Option -> 宏

### 销毁并调用函数

```move
// 不好！
if (opt.is_some()) {
    let inner = opt.destroy_some();
    call_function(inner);
};

// 好！有宏可以做这个！
opt.do!(|value| call_function(value));
```

### 销毁某些带默认值

```move
let opt = option::none();

// 不好！
let value = if (opt.is_some()) {
    opt.destroy_some()
} else {
    abort EError
};

// 好！有宏！
let value = opt.destroy_or!(default_value);

// 你甚至可以在`none`时中止
let value = opt.destroy_or!(abort ECannotBeEmpty);
```

## 循环 -> 宏

### 执行操作N次

```move
// 不好！难以阅读！
let mut i = 0;
while (i < 32) {
    do_action();
    i = i + 1;
};

// 好！任何uint都有这个宏！
32u8.do!(|_| do_action());
```

### 从迭代创建新向量

```move
// 更难读！
let mut i = 0;
let mut elements = vector[];
while (i < 32) {
    elements.push_back(i);
    i = i + 1;
};

// 易于阅读！
vector::tabulate!(32, |i| i);
```

### 对向量的每个元素执行操作

```move
// 不好！
let mut i = 0;
while (i < vec.length()) {
    call_function(&vec[i]);
    i = i + 1;
};

// 好！
vec.do_ref!(|e| call_function(e));
```

### 销毁向量并对每个元素调用函数

```move
// 不好！
while (!vec.is_empty()) {
    call(vec.pop_back());
};

// 好！
vec.destroy!(|e| call(e));
```

### 将向量折叠成单个值

```move
// 不好！
let mut aggregate = 0;
let mut i = 0;

while (i < source.length()) {
    aggregate = aggregate + source[i];
    i = i + 1;
};

// 好！
let aggregate = source.fold!(0, |acc, v| {
    acc + v
});
```

### 过滤向量元素

> 注意：`source`向量中的`T: drop`

```move
// 不好！
let mut filtered = [];
let mut i = 0;
while (i < source.length()) {
    if (source[i] > 10) {
        filtered.push_back(source[i]);
    };
    i = i + 1;
};

// 好！
let filtered = source.filter!(|e| e > 10);
```

## 其他

### 解包中被忽略的值可以完全忽略

```move
// 不好！非常稀疏！
let MyStruct { id, field_1: _, field_2: _, field_3: _ } = value;
id.delete();

// 好！2024语法
let MyStruct { id, .. } = value;
id.delete();
```

## 测试

### 合并`#[test]`和`#[expected_failure(...)]`

```move
// 不好！
#[test]
#[expected_failure]
fun value_passes_check() {
    abort
}

// 好！
#[test, expected_failure]
fun value_passes_check() {
    abort
}
```

### 不要清理`expected_failure`测试

```move
// 不好！清理不是必要的
#[test, expected_failure(abort_code = my_app::EIncorrectValue)]
fun try_take_missing_object_fail() {
    let mut test = test_scenario::begin(@0);
    my_app::call_function(test.ctx());
    test.end();
}

// 好！容易看到测试预期在哪里失败
#[test, expected_failure(abort_code = my_app::EIncorrectValue)]
fun try_take_missing_object_fail() {
    let mut test = test_scenario::begin(@0);
    my_app::call_function(test.ctx());

    abort // 将与EIncorrectValue不同
}
```

### 在测试模块中不要给测试加`test_`前缀

```move
// 不好！模块已经叫_tests
module my_package::my_module_tests;

#[test]
fun test_this_feature() { /* ... */ }

// 好！结果是更好的函数名
#[test]
fun this_feature_works() { /* ... */ }
```

### 在不必要的地方不要使用`TestScenario`

```move
// 不好！不需要，只使用ctx
let mut test = test_scenario::begin(@0);
let nft = app::mint(test.ctx());
app::destroy(nft);
test.end();

// 好！简单情况有虚拟上下文
let ctx = &mut tx_context::dummy();
app::mint(ctx).destroy();
```

### 在测试的`assert!`中不要使用中止代码

```move
// 不好！可能意外匹配应用程序错误代码
assert!(is_success, 0);

// 好！
assert!(is_success);
```

### 尽可能使用`assert_eq!`

```move
// 不好！旧式代码
assert!(result == b"expected_value", 0);

// 好！如果失败将打印两个值
use std::unit_test::assert_eq;

assert_eq!(result, expected_value);
```

### 使用"黑洞"`destroy`函数

```move
// 不好！
nft.destroy_for_testing();
app.destroy_for_testing();

// 好！- 无需为清理定义特殊函数
use sui::test_utils::destroy;

destroy(nft);
destroy(app);
```

## 注释

### 文档注释以`///`开始

```move
// 不好！工具不支持JavaDoc样式注释
/**
 * Cool method
 * @param ...
 */
public fun do_something() { /* ... */ }

// 好！将在docgen和IDE中渲染为文档注释
/// Cool method!
public fun do_something() { /* ... */ }
```

### 复杂逻辑？留下注释`//`

友好并帮助审查者理解代码！

```move
// 好！
// 注意：如果值小于10可能下溢。
// TODO：在这里添加`assert!`
let value = external_call(value, ctx);
```