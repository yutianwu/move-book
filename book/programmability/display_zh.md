# 对象显示

Sui上的对象在其结构和行为方面是明确的，可以以可理解的方式显示。但是，为了支持客户端的更丰富元数据，有一种标准且高效的"描述"对象给客户端的方式 - 在[Sui框架](./sui-framework)中定义的`Display`对象。

## 背景

历史上，有不同的尝试来同意对象的标准结构，以便它可以在用户界面中显示。其中一种方法是在对象结构体中定义某些字段，当存在时，将在UI中使用。这种方法不够灵活，需要开发者在每个对象中定义相同的字段，有时这些字段对对象来说没有意义。

```move file=packages/samples/sources/programmability/display.move anchor=background

```

如果任何字段包含静态数据，它将在每个对象中重复。而且，由于Move没有接口，不可能在不"手动"检查对象类型的情况下知道对象是否具有特定字段，这使得客户端获取更加复杂。

## 对象显示

为了解决这些问题，Sui引入了一种描述对象以供显示的标准方式。显示元数据不是在对象结构体中定义字段，而是存储在与类型关联的单独对象中。这样，显示元数据不会重复，并且易于扩展和维护。

Sui显示的另一个重要功能是能够定义模板并在这些模板中使用对象字段。它不仅允许更灵活的显示，而且还使开发者免于在每个对象中定义具有相同名称和类型的相同字段的需要。

> 对象显示由[Sui全节点](https://docs.sui.io/guides/operator/sui-full-node)原生支持，如果对象类型有关联的Display，客户端可以获取任何对象的显示元数据。

```move file=packages/samples/sources/programmability/display.move anchor=hero

```

## 创建者特权

虽然对象可以由账户拥有并且可能受到[真正所有权](./../object/ownership#account-owner-or-single-owner)的约束，Display可以由对象的创建者拥有。这样，创建者可以更新显示元数据并全局应用更改，而无需更新每个对象。创建者还可以将Display转移到另一个账户，甚至围绕对象构建具有自定义功能的应用程序来管理元数据。

## 标准字段

最广泛支持的字段是：

- `name` - 对象的名称。当用户查看对象时显示名称。
- `description` - 对象的描述。当用户查看对象时显示描述。
- `link` - 在应用程序中使用的对象链接。
- `image_url` - 对象图像的URL或blob。
- `thumbnail_url` - 在钱包、浏览器和其他产品中用作预览的较小图像的URL。
- `project_url` - 与对象或创建者关联的网站链接。
- `creator` - 表示对象创建者的字符串。

> 请参考[Sui文档](https://docs.sui.io/standards/display)获取最新的支持字段列表。

虽然有一套标准字段，Display对象并不强制执行它们。开发者可以定义他们需要的任何字段，客户端可以根据需要使用它们。一些应用程序可能需要额外的字段，并省略其他字段，Display足够灵活以支持它们。

## 使用Display

`Display`对象在`sui::display`模块中定义。它是一个通用结构体，接受一个幻影类型作为参数。幻影类型用于将`Display`对象与它描述的类型关联。`Display`对象的`fields`是键值对的`VecMap`，其中键是字段名称，值是字段值。`version`字段用于版本化显示元数据，并在`update_display`调用时更新。

```move
module sui::display;

public struct Display<phantom T: key> has key, store {
    id: UID,
    /// Contains fields for display. Currently supported
    /// fields are: name, link, image and description.
    fields: VecMap<String, String>,
    /// Version that can only be updated manually by the Publisher.
    version: u16
}
```

创建新Display需要[Publisher](./publisher)对象，因为它作为类型所有权的证明。

## 模板语法

目前，Display支持简单的字符串插值，可以在其模板中使用结构体字段（和路径）。语法很简单 - `{path}`被替换为路径处字段的值。在嵌套字段的情况下，路径是从根对象开始的以点分隔的字段名称列表。

```move file=packages/samples/sources/programmability/display.move anchor=nested

```

上述类型`LittlePony`的Display可以定义如下：

```json
{
  "name": "Just a pony",
  "image_url": "{image_url}",
  "description": "{metadata.description}"
}
```

## 多个Display对象

对于特定的`T`，可以创建多少个`Display<T>`对象没有限制。但是，最近更新的`Display<T>`将被全节点使用。

## 进一步阅读

- [Sui对象显示](https://docs.sui.io/standards/display)在Sui文档中
- [Publisher](./publisher) - 创建者的表示