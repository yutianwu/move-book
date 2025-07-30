# 能力：Drop

<!-- TODO: reiterate, given that we introduce abilities one by one -->

<!-- TODO:

- introduce abilities first
- mention them all
- then do one by one

consistency: we / I / you ?
who is we? I am alone, there's no one else here


-->

<!--

// Shall we only talk about `drop` ?
// So that we don't explain scopes and `copy` / `move` semantics just yet?

Chapter: Basic Syntax
Goal: Introduce Copy and Drop abilities of Move. Follows the `struct` section
Notes:
    - compare them to primitive types introduces before;
    - what is an ability without drop
    - drop is not necessary for unpacking
    - make a joke about a bacteria pattern in the code
    - mention that a struct with only `drop` ability is called a Witness
    - mention that a struct without abilities is called a Hot Potato
    - mention that there are two more abilities which are covered in a later chapter

Links:
    - language reference (abilities)
    - authorization patterns (or witness)
    - hot potato pattern
    - key and store abilities (later chapter)

 -->

`drop` 能力是最简单的能力，它允许结构体的实例被 _忽略_ 或 _丢弃_。在许多编程语言中，这种行为被认为是默认的。然而，在 Move 中，没有 `drop` 能力的结构体不允许被忽略。这是 Move 语言的一个安全特性，确保所有资产都得到正确处理。尝试忽略没有 `drop` 能力的结构体将导致编译错误。

```move file=packages/samples/sources/move-basics/drop-ability.move anchor=main

```

`drop` 能力经常用于自定义集合类型，以消除在不再需要集合时进行特殊处理的需要。例如，`vector` 类型具有 `drop` 能力，这允许向量在不再需要时被忽略。然而，Move 类型系统的最大特性是能够不具有 `drop`。这确保资产得到正确处理而不被忽略。

只具有 `drop` 能力的结构体被称为 _见证者_（Witness）。我们在[见证者和抽象实现](./../programmability/witness-pattern)部分解释了 _见证者_ 的概念。

## 具有 `drop` 能力的类型

Move 中的所有原生类型都具有 `drop` 能力。这包括：

- [`bool`](./../move-basics/primitive-types#booleans)
- [无符号整数](./../move-basics/primitive-types#integer-types)
- [`vector<T>`](./../move-basics/vector) 当 `T` 具有 `drop` 时
- [`address`](./../move-basics/address)

标准库中定义的所有类型也都具有 `drop` 能力。这包括：

- [`Option<T>`](./../move-basics/option) 当 `T` 具有 `drop` 时
- [`String`](./../move-basics/string)
- [`TypeName`](./../move-basics/type-reflection)

## 进一步阅读

- Move 参考中的[类型能力](./../../reference/abilities)。