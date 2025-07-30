---
title: 'Index Syntax | Reference'
description: ''
---

# 索引语法

Move 提供语法属性，允许您定义看起来和感觉像原生 Move 代码的操作，将这些操作降级到您用户提供的定义中。

我们的第一个语法方法 `index` 允许您定义一组操作，这些操作可以用作您的数据类型的自定义索引访问器，比如将矩阵元素访问为 `m[i,j]`，通过注释应该用于这些索引操作的函数。此外，这些定义是每种类型的定制，并且对使用您类型的任何程序员都隐式可用。

## 概述和总结

首先，考虑一个使用向量的向量来表示其值的 `Matrix` 类型。您可以使用 `index` 语法注释在 `borrow` 和 `borrow_mut` 函数上编写一个小库，如下所示：

```move
module matrix::matrix;

public struct Matrix<T> { v: vector<vector<T>> }

#[syntax(index)]
public fun borrow<T>(s: &Matrix<T>, i: u64, j: u64): &T {
    vector::borrow(vector::borrow(&s.v, i), j)
}

#[syntax(index)]
public fun borrow_mut<T>(s: &mut Matrix<T>, i: u64, j: u64): &mut T {
    vector::borrow_mut(vector::borrow_mut(&mut s.v, i), j)
}

public fun make_matrix<T>(v: vector<vector<T>>):  Matrix<T> {
    Matrix { v }
}
```

现在任何使用这个 `Matrix` 类型的人都可以访问它的索引语法：

```move
let mut m = matrix::make_matrix(vector[
    vector[1, 0, 0],
    vector[0, 1, 0],
    vector[0, 0, 1],
]);x

let mut i = 0;
while (i < 3) {
    let mut j = 0;
    while (j < 3) {
        if (i == j) {
            assert!(m[i, j] == 1, 1);
        } else {
            assert!(m[i, j] == 0, 0);
        };
        *(&mut m[i,j]) = 2;
        j = j + 1;
    };
    i = i + 1;
}
```

## 用法

如示例所示，如果您定义一个数据类型和相关联的索引语法方法，任何人都可以通过在该类型的值上编写索引语法来调用该方法：

```move
let mat = matrix::make_matrix(...);
let m_0_0 = mat[0, 0];
```

在编译期间，编译器根据表达式的位置和可变用法将这些转换为适当的函数调用：

```move
let mut mat = matrix::make_matrix(...);

let m_0_0 = mat[0, 0];
// 转换为 `copy matrix::borrow(&mat, 0, 0)`

let m_0_0 = &mat[0, 0];
// 转换为 `matrix::borrow(&mat, 0, 0)`

let m_0_0 = &mut mat[0, 0];
// 转换为 `matrix::borrow_mut(&mut mat, 0, 0)`
```

您还可以将索引表达式与字段访问混合使用：

```move
public struct V { v: vector<u64> }

public struct Vs { vs: vector<V> }

fun borrow_first(input: &Vs): &u64 {
    &input.vs[0].v[0]
    // 转换为 `vector::borrow(&vector::borrow(&input.vs, 0).v, 0)`
}
```

### 索引函数接受灵活的参数

注意，除了本章其余部分描述的定义和类型限制外，Move 对索引语法方法作为参数接受的值没有限制。这允许您在定义索引语法时实现复杂的程序化行为，比如一个数据结构，如果索引超出范围就取默认值：

```move
#[syntax(index)]
public fun borrow_or_set<Key: copy, Value: drop>(
    input: &mut MTable<Key, Value>,
    key: Key,
    default: Value
): &mut Value {
    if (contains(input, key)) {
        borrow(input, key)
    } else {
        insert(input, key, default);
        borrow(input, key)
    }
}
```

现在，当您索引到 `MTable` 时，您还必须提供一个默认值：

```move
let string_key: String = ...;
let mut table: MTable<String, u64> = m_table::make_table();
let entry: &mut u64 = &mut table[string_key, 0];
```

这种可扩展的功能允许您为您的类型编写精确的索引接口，具体地强制执行定制行为。

## 定义索引语法函数

这种强大的语法形式允许您所有用户定义的数据类型以这种方式行为，假设您的定义遵守以下规则：

1. `#[syntax(index)]` 属性被添加到与主体类型相同模块中定义的指定函数。
2. 指定的函数具有 `public` 可见性。
3. 函数将引用类型作为其主体类型（其第一个参数）并返回匹配的引用类型（如果主体是 `mut` 则为 `mut`）。
4. 每种类型只有单个可变和单个不可变定义。
5. 不可变和可变版本具有类型一致性：
   - 主体类型匹配，仅在可变性上不同。
   - 返回类型与其主体类型的可变性匹配。
   - 类型参数（如果存在）在两个版本之间具有相同的约束。
   - 主体类型之外的所有参数都相同。

以下内容和附加示例更详细地描述了这些规则。

### 声明

要声明索引语法方法，在与主体类型定义相同模块中的相关函数定义上方添加 `#[syntax(index)]` 属性。这向编译器发出信号，该函数是指定类型的索引访问器。

#### 不可变访问器

不可变索引语法方法为只读访问定义。它接受主体类型的不可变引用并返回元素类型的不可变引用。在 `std::vector` 中定义的 `borrow` 函数是这方面的例子：

```move
#[syntax(index)]
public native fun borrow<Element>(v: &vector<Element>, i: u64): &Element;
```

#### 可变访问器

可变索引语法方法是不可变方法的对偶，允许读写操作。它接受主体类型的可变引用并返回元素类型的可变引用。在 `std::vector` 中定义的 `borrow_mut` 函数是这方面的例子：

```move
#[syntax(index)]
public native fun borrow_mut<Element>(v: &mut vector<Element>, i: u64): &mut Element;
```

#### 可见性

为了确保索引函数在使用类型的任何地方都可用，所有索引语法方法都必须具有公共可见性。这确保了在 Move 中跨模块和包使用索引的人体工程学。

#### 无重复

除了上述要求外，我们限制每个主体基本类型为不可变引用定义单个索引语法方法和为可变引用定义单个索引语法方法。例如，您不能为多态类型定义专门版本：

```move
#[syntax(index)]
public fun borrow_matrix_u64(s: &Matrix<u64>, i: u64, j: u64): &u64 { ... }

#[syntax(index)]
public fun borrow_matrix<T>(s: &Matrix<T>, i: u64, j: u64): &T { ... }
    // 错误！Matrix 已经有其不可变索引语法方法的定义
```

这确保您总是可以知道正在调用哪个方法，而无需检查类型实例化。

### 类型约束

默认情况下，索引语法方法具有以下类型约束：

**其主体类型（第一个参数）必须是对与标记函数相同模块中定义的单个类型的引用。** 这意味着您不能为元组、类型参数或值定义索引语法方法：

```move
#[syntax(index)]
public fun borrow_fst(x: &(u64, u64), ...): &u64 { ... }
    // 错误，因为主体类型是元组

#[syntax(index)]
public fun borrow_tyarg<T>(x: &T, ...): &T { ... }
    // 错误，因为主体类型是类型参数

#[syntax(index)]
public fun borrow_value(x: Matrix<u64>, ...): &u64 { ... }
    // 错误，因为 x 不是引用
```

**主体类型必须与返回类型的可变性匹配。** 这个限制允许您澄清在将索引表达式借用为 `&vec[i]` 与 `&mut vec[i]` 时的预期行为。Move 编译器使用可变性标记来确定调用哪种借用形式以产生适当可变性的引用。因此，我们不允许主体和返回可变性不同的索引语法方法：

```move
#[syntax(index)]
public fun borrow_imm(x: &mut Matrix<u64>, ...): &u64 { ... }
    // 错误！不兼容的可变性
    // 期望可变引用 '&mut' 返回类型
```

### 类型兼容性

当定义不可变和可变索引语法方法对时，它们受到许多兼容性约束：

1. 它们必须接受相同数量的类型参数，这些类型参数必须具有相同的约束。
2. 类型参数必须*按位置*使用相同，而不是名称。
3. 它们的主体类型必须完全匹配，除了可变性。
4. 它们的返回类型必须完全匹配，除了可变性。
5. 所有其他参数类型必须完全匹配。

这些约束是为了确保索引语法的行为无论是在可变还是不可变位置都完全相同。

为了说明其中一些错误，回顾之前的 `Matrix` 定义：

```move
#[syntax(index)]
public fun borrow<T>(s: &Matrix<T>, i: u64, j: u64): &T {
    vector::borrow(vector::borrow(&s.v, i), j)
}
```

以下所有都是可变版本的类型不兼容定义：

```move
#[syntax(index)]
public fun borrow_mut<T: drop>(s: &mut Matrix<T>, i: u64, j: u64): &mut T { ... }
    // 错误！`T` 这里有 `drop`，但在不可变版本中没有

#[syntax(index)]
public fun borrow_mut(s: &mut Matrix<u64>, i: u64, j: u64): &mut u64 { ... }
    // 错误！这接受不同数量的类型参数

#[syntax(index)]
public fun borrow_mut<T, U>(s: &mut Matrix<U>, i: u64, j: u64): &mut U { ... }
    // 错误！这接受不同数量的类型参数

#[syntax(index)]
public fun borrow_mut<U>(s: &mut Matrix<U>, i_j: (u64, u64)): &mut U { ... }
    // 错误！这接受不同数量的参数

#[syntax(index)]
public fun borrow_mut<U>(s: &mut Matrix<U>, i: u64, j: u32): &mut U { ... }
    // 错误！`j` 是不同的类型
```

同样，这里的目标是使不可变和可变版本之间的使用保持一致。这允许索引语法方法在工作时不根据可变与不可变使用改变行为或约束，最终确保了一个一致的编程接口。