# 注释

<!--

Chapter: Basic Syntax
Goal: Introduce comments.
Notes:
    - doc comments are used in docgen
    - only public members are documented
    - doc comments are placed in between attributes and the definition
    - doc comments are allowed for: modules, structs, functions, constants
    - give an example of how doc comments are translated
 -->

注释是在代码中添加说明或文档的方法。它们被编译器忽略，不会生成 Move 字节码。你可以使用注释来解释代码的功能，为自己或其他开发者添加备注，临时移除部分代码，或生成文档。Move 中有三种类型的注释：行注释、块注释和文档注释。

## 行注释

你可以使用双斜杠 `//` 来注释该行的其余部分。`//` 之后的所有内容都会被编译器忽略。

```move file=packages/samples/sources/move-basics/comments-line.move anchor=main

```

## 块注释

块注释用于注释一段代码。它们以 `/*` 开始，以 `*/` 结束。`/*` 和 `*/` 之间的所有内容都会被编译器忽略。你可以使用块注释来注释单行或多行。你甚至可以使用它们来注释行的一部分。

```move file=packages/samples/sources/move-basics/comments-block.move anchor=main

```

这个例子有点极端，但它展示了使用块注释的所有方式。

## 文档注释

文档注释是用于为代码生成文档的特殊注释。它们类似于块注释，但以三个斜杠 `///` 开头，并放置在它们要文档化的项的定义之前。

```move file=packages/samples/sources/move-basics/comments-doc.move anchor=main

```

## 空白字符

与某些语言不同，空白字符（空格、制表符和换行符）对程序的含义没有影响。

<!-- TODO: docgen, which members are in the documentation -->