---
title: '朋友 | 参考手册'
description: ''
---

# 已弃用：朋友

注意：此特性已被 [`public(package)`](./functions#visibility) 取代。

`friend` 语法用于声明当前模块信任的模块。受信任的模块被允许调用当前模块中定义的任何具有 `public(friend)` 可见性的函数。有关函数可见性的详细信息，请参阅[函数](./functions)中的 _可见性_ 部分。

## 朋友声明

模块可以通过朋友声明语句将其他模块声明为朋友，格式为

- `friend <address::name>` — 使用完全限定模块名称的朋友声明，如下面的示例，或

  ```move
  module 0x42::a {
      friend 0x42::b;
  }
  ```

- `friend <module-name-alias>` — 使用模块名称别名的朋友声明，其中模块别名通过 `use` 语句引入。

  ```move
  module 0x42::a {
      use 0x42::b;
      friend b;
  }
  ```

一个模块可能有多个朋友声明，所有朋友模块的并集构成朋友列表。在下面的示例中，`0x42::B` 和 `0x42::C` 都被视为 `0x42::A` 的朋友。

```move
module 0x42::a;

friend 0x42::b;
friend 0x42::c;
```

与 `use` 语句不同，`friend` 只能在模块作用域中声明，不能在表达式块作用域中声明。`friend` 声明可以位于允许顶级构造（例如 `use`、`function`、`struct` 等）的任何地方。但是，为了可读性，建议将朋友声明放在模块定义的开头附近。

### 朋友声明规则

朋友声明受以下规则约束：

- 模块不能将自己声明为朋友。

  ```move
  module 0x42::m { friend Self; // 错误! }
  //                      ^^^^ 不能将模块本身声明为朋友

  module 0x43::m { friend 0x43::M; // 错误! }
  //                      ^^^^^^^ 不能将模块本身声明为朋友
  ```

- 朋友模块必须为编译器所知

  ```move
  module 0x42::m { friend 0x42::nonexistent; // 错误! }
  //                      ^^^^^^^^^^^^^^^^^ 未绑定的模块 '0x42::nonexistent'
  ```

- 朋友模块必须在相同的账户地址内。

  ```move
  module 0x42::m {}

  module 0x42::n { friend 0x42::m; // 错误! }
  //                      ^^^^^^^ 不能将当前地址之外的模块声明为朋友
  ```

- 朋友关系不能创建循环模块依赖。

  朋友关系中不允许循环，例如，关系 `0x2::a` 朋友 `0x2::b` 朋友 `0x2::c` 朋友 `0x2::a` 是不允许的。更一般地说，声明朋友模块会在朋友模块上添加对当前模块的依赖（因为目的是让朋友调用当前模块中的函数）。如果该朋友模块已经被使用，无论是直接还是传递使用，都会创建依赖循环。

  ```move
  module 0x2::a {
      use 0x2::c;
      friend 0x2::b;

      public fun a() {
          c::c()
      }
  }

  module 0x2::b {
      friend 0x2::c; // 错误!
  //         ^^^^^^ 这个朋友关系创建了依赖循环：'0x2::b' 是 '0x2::a' 的朋友，使用 '0x2::c' 是 '0x2::b' 的朋友
  }

  module 0x2::c {
      public fun c() {}
  }
  ```

- 模块的朋友列表不能包含重复项。

  ```move
  module 0x42::a {}

  module 0x42::m {
      use 0x42::a as aliased_a;
      friend 0x42::A;
      friend aliased_a; // 错误!
  //         ^^^^^^^^^ 重复的朋友声明 '0x42::a'。模块中的朋友声明必须是唯一的
  }
  ```