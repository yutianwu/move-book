# Move - 数字资产的语言

智能合约编程语言历来专注于定义和管理数字资产。例如，以太坊中的ERC-20标准开创了一套与数字货币代币交互的标准，为在区块链上创建和管理数字货币建立了蓝图。随后，ERC-721标准的引入标志着一个重大演进，普及了非同质化代币（NFT）的概念，它们代表独特的、不可分割的资产。这些标准为我们今天看到的复杂数字资产奠定了基础。

<!-- ## Move and Digital Assets -->

<!-- note: consider "native" -> "fine-grained" -->

然而，以太坊的编程模型缺乏资产的原生表示。换句话说，从外部看，智能合约表现得像资产，但语言本身没有固有地表示资产的方式。从一开始，Move就旨在为资产提供一流的抽象，为思考和编程资产开辟了新途径。

<!-- Move was initially created in 2018 as part of the Libra project. The language was designed to address shortcomings in existing smart contract languages, especially in handling assets and access control. The Move language aims to provide first-class abstractions for these concepts, improving the safety and productivity of smart contract programming. -->

重要的是要强调资产的哪些属性是必要的：

- **所有权：** 每个资产都与所有者关联，反映了物理世界中所有权的直观概念——就像您拥有一辆汽车一样，您可以拥有数字资产。Move以这样的方式强制执行所有权：一旦资产被_移动_，前所有者就完全失去对它的任何控制。这种机制确保了清晰和安全的所有权变更。

- **不可复制：** 在现实世界中，独特的物品不能轻易复制。Move将这一原则应用于数字资产，确保它们不能在程序中任意复制。这一属性对于维护数字资产的稀缺性和独特性至关重要，反映了物理资产的内在价值。

- **不可丢弃：** 就像您不能意外地毫无痕迹地丢失房子或汽车一样，Move确保没有资产可以在程序中被丢弃或丢失。相反，资产必须明确转移或销毁。这一属性保证了数字资产的谨慎处理，防止意外丢失并确保资产管理的问责制。

Move成功地将这些属性封装在其设计中，成为数字资产的理想语言。

## 总结

- Move被设计为为数字资产提供一流的抽象，使开发者能够原生地创建和管理资产。
- 数字资产的基本属性包括所有权、不可复制性和不可丢弃性，Move在其设计中强制执行这些属性。
- Move的资产模型反映了现实世界的资产管理，确保安全和负责任的资产所有权和转移。

## 延伸阅读

- [Move: A Language With Programmable Resources (pdf)](https://developers.diem.com/papers/diem-move-a-language-with-programmable-resources/2019-06-18.pdf)
  由Sam Blackshear, Evan Cheng, David L. Dill, Victor Gao, Ben Maurer, Todd Nowacki, Alistair Pott,
  Shaz Qadeer, Rain, Dario Russi, Stephane Sezer, Tim Zakian, Runtian Zhou撰写\*