---
title: '标记控制流 | 参考'
description: ''
---

# 标记控制流

Move在编写循环和代码块时支持标记控制流，允许您`break`和`continue`循环以及从块中`return`（这在存在宏的情况下特别有用）。

## 循环

循环允许您在函数中定义和将控制转移到特定标签。例如，我们可以嵌套两个循环并使用带有这些标签的`break`和`continue`来精确指定控制流。您可以在任何`loop`或`while`形式前加上`'label:`形式，以允许直接在那里中断或继续。

为了演示这种行为，考虑一个函数，它接受嵌套的数字向量（即`vector<vector<u64>>`）来对照某个阈值求和，其行为如下：

- 如果所有数字的总和低于阈值，返回该总和。
- 如果将一个数字添加到当前总和会超过阈值，返回当前总和。

我们可以通过将向量的向量作为嵌套循环迭代并标记外层循环来编写此函数。如果内层循环中的任何加法会使我们超过阈值，我们可以使用带有外层标签的`break`来一次性退出两个循环：

```move
fun sum_until_threshold(input: &vector<vector<u64>>, threshold: u64): u64 {
    let mut sum = 0;
    let mut i = 0;
    let input_size = input.length();

    'outer: loop {
        // 中断到外层，因为它是最近的封闭循环
        if (i >= input_size) break sum;

        let vec = &input[i];
        let size = vec.length();
        let mut j = 0;

        while (j < size) {
            let v_entry = vec[j];
            if (sum + v_entry < threshold) {
                sum = sum + v_entry;
            } else {
                // 我们看到的下一个元素会打破阈值，
                // 所以我们返回当前总和
                break 'outer sum
            };
            j = j + 1;
        };
        i = i + 1;
    }
}
```

这些类型的标签也可以与嵌套循环形式一起使用，在更大的代码体中提供精确控制。例如，如果我们正在处理一个大表，其中每个条目都需要迭代，可能会看到我们继续内层或外层循环，我们可以使用标签表达该代码：

```move
let x = 'outer: loop {
    ...
    'inner: while (cond) {
        ...
        if (cond0) { break 'outer value };
        ...
        if (cond1) { continue 'inner }
        else if (cond2) { continue 'outer }
        ...
    }
        ...
};
```

## 标记块

标记块允许您编写包含函数内非局部控制流的Move程序，包括在宏lambda内部和返回值：

```move
fun named_return(n: u64): vector<u8> {
    let x = 'a: {
        if (n % 2 == 0) {
            return 'a b"even"
        };
        b"odd"
    };
    x
}
```

在这个简单的示例中，程序检查输入`n`是否为偶数。如果是，程序离开标记为`'a:`的块，值为`b"even"`。如果不是，代码继续，以值`b"odd"`结束标记为`'a:`的块。最后，我们将`x`设置为该值然后返回它。

这个控制流功能也适用于宏体。例如，假设我们想编写一个函数来查找向量中的第一个偶数，并且我们有一些宏`for_ref`在循环中迭代向量元素：

```move
macro fun for_ref<$T>($vs: &vector<$T>, $f: |&$T|) {
    let vs = $vs;
    let mut i = 0;
    let end = vs.length();
    while (i < end) {
        $f(vs.borrow(i));
        i = i + 1;
    }
}
```

使用`for_ref`和标签，我们可以编写一个lambda表达式传递给`for_ref`，它将退出循环，返回找到的第一个偶数：

```move
fun find_first_even(vs: vector<u64>): Option<u64> {
    'result: {
        for_ref!(&vs, |n| if (*n % 2 == 0) { return 'result option::some(*n)});
        option::none()
    }
}
```

这个函数将迭代`vs`直到找到偶数，并返回它（或者如果不存在偶数则返回`option::none()`）。这使得命名标签成为与控制流宏（如`for!`）交互的强大工具，允许您在这些上下文中自定义迭代行为。

## 限制

为了澄清程序行为，您只能对循环标签使用`break`和`continue`，而`return`只适用于块标签。为此，以下程序产生错误：

```move
fun bad_loop() {
    'name: loop {
        return 'name 5
            // ^^^^^ 在循环块标签上使用'return'的无效用法
    }
}

fun bad_block() {
    'name: {
        continue 'name;
              // ^^^^^ 在循环块标签上使用'break'的无效用法
        break 'name;
           // ^^^^^ 在循环块标签上使用'break'的无效用法
    }
}
```