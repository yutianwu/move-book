# 发布者权限

在应用程序设计和开发中，通常需要证明发布者权限。这在数字资产的上下文中尤为重要，发布者可能为其资产启用或禁用某些功能。Publisher对象是在[Sui框架](./sui-framework)中定义的对象，允许发布者证明他们对类型的_权限_。

## 定义

Publisher对象在Sui框架的`sui::package`模块中定义。它是一个非常简单的、非泛型对象，每个模块可以初始化一次（每个包可以多次），用于证明发布者对类型的权限。要声明Publisher对象，发布者必须向`package::claim`函数提供[一次性见证](./one-time-witness)。

```move
module sui::package;

public struct Publisher has key, store {
    id: UID,
    package: String,
    module_name: String,
}
```

> 如果您不熟悉一次性见证，可以在[这里](./one-time-witness)阅读更多信息。

这是在模块中声明`Publisher`对象的简单示例：

```move file=packages/samples/sources/programmability/publisher.move anchor=publisher

```

## 使用

Publisher对象有两个与之关联的函数，用于证明发布者对类型的权限：

```move file=packages/samples/sources/programmability/publisher.move anchor=use_publisher

```

## Publisher作为管理员角色

对于小型应用程序或简单用例，Publisher对象可以用作管理员[能力](./capability)。虽然在更广泛的上下文中，Publisher对象控制系统配置，但它也可以用于管理应用程序的状态。

```move file=packages/samples/sources/programmability/publisher.move anchor=publisher_as_admin

```

然而，Publisher缺少[能力](./capability)的一些原生属性，如类型安全和表达性。`admin_action`的签名不是很明确，可以被任何人调用。由于`Publisher`对象是标准的，如果不执行`from_module`检查，现在存在未经授权访问的风险。因此，在将`Publisher`对象用作管理员角色时要谨慎。

## 在Sui上的角色

Publisher是Sui上某些功能所必需的。[对象显示](./display)只能由Publisher创建，TransferPolicy - Kiosk系统的重要组件 - 也需要Publisher对象来证明类型的所有权。

## 下一步

在下一章中，我们将介绍需要Publisher对象的第一个功能 - 对象显示 - 一种为客户端描述对象并标准化元数据的方式。这是用户友好应用程序的必备功能。