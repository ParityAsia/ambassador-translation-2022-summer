```
---
proofreader: Kaichao
completion-date: 2022-06-21
---
```

![img](https://www.parity.io/images/sbp-acala.jpeg)

SBP（Substrate Builders Program）毕业生中的一些重点团队已经成功完成 Substrate Builders 计划并在 Polkadot 上启动平行链。这篇文章我们将介绍第一个在 Polkadot 上赢得平行链 Auction 的项目：**Acala**。

Acala 这个名字来自佛教中不动明王，代表了 DeFi 生态系统中稳定币的基础方面，以及更广泛的区块链生态系统。Acala 的使命是帮助在 Polkadot 和 Kusama 网络及其他网络中，建立金融工具和基础设施的基础。

作为首批采用 Substrate 作为其区块链框架的团队之一，Acala 从一开始就得到了 Substrate Builders Program **直接支持**——从 Substrate 的第一个分叉到赢得  Polkadot 上的第一个平行链插槽拍卖。

Acala 是一个与以太坊兼容的金融应用平台，可以使用智能合约或内置协议，具有开箱即用的跨链功能和强大的安全性。它提供了一套金融应用程序，包括多**抵押稳定币**、**去中心化交易所**以及可以**用多种代币以极低价格支付手续费**。

“Acala 是 Polkadot 的 DeFi 和流动性中心，它使用 Substrate 为 DeFi 开发者和用户定制的平行链和以太坊兼容的 dapp 平台。Substrate 使我们能够构建 DeFi 底层基础协议和优化，以改善开发者和用户体验，而 Substrate 和 Polkadot 的可升级性使我们能够在未来证明我们的链。” 

——Bryan Chen

Acala 联合创始人

在 Substrate.io 上查阅 Acala 案例：

[https://substrate.io/ecosystem/projects/case-studies/acala/](https://substrate.io/ecosystem/projects/case-studies/acala/?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJhdWQiOiJhY2Nlc3NfcmVzb3VyY2UiLCJleHAiOjE2NTU3OTk3NjksImZpbGVHVUlEIjoibTRrTUxKb2xMTkNlUDlxRCIsImlhdCI6MTY1NTc5OTQ2OSwidXNlcklkIjoyMzkwOTU3NX0.KRAktyBCPAtwcTNtMKV4YA4-yJAUCoOz6w4QrKbOjbE)

## 在Substrate Builders Program的旅程

Acala 是 Substrate Builders Program 的17名首批参与者之一，这个计划在2020 年3月5日首次宣布。从 Substrate 使用期开始，Acala 团队就开始建立了整个网络，也取得了许多成就。因为太多，这里就不一一列出了。但是从他们的 Substrate node template 到赢得 Polkadot 平行链插槽的过程中，有几个主要亮点值得提出：

- **Mandala 测试网的开发和发布**。这是一个基于 Substrate 的链，用于测试在 Substrate 上构建 DeFi 链的第一个功能开发。

- 在 Kusama 上 **Karura****（Acala 的金丝雀网络）**的开发、发布和平行链上线。这个已经上线、具有价值的试验网络，让社区体验 Acala DeFi 中心成为了可能，同时还能测试它的极限和功能。

- 一组具有不同功能的 **Substrate Pallet** 的开发和实现：

  - [**交易支付模块**](https://github.com/AcalaNetwork/Acala/blob/master/modules/transaction-payment/src/lib.rs) **(Transaction Payment pallet)** 处理要在不同资产中支付的费用。它使用去中心化交易所来处理交易费用，并使用 SignedExtension 来执行。

  - 交易支付模块与[**预言机模块**](https://github.com/open-web3-stack/open-runtime-module-library/blob/master/oracle/src/lib.rs) **(Oracle** **pallet)** 的紧密结合，预言机模块配备了最新的 MEV ( Miner Extractable Value 矿工可提取价值) 保护，这都得益于 Substrate 允许开发者构建每个 pallet。

  - 预言机模块使用了 Substrate 中的**权重类别** **(Weight Class)** 和**自定义发送信息**的功能。

  - Acala 还构建了自己的[**以太坊虚拟机模块**](https://github.com/AcalaNetwork/Acala/blob/master/modules/evm/src/lib.rs) (EVM pallet)，这使用 Substrate 中所有 pallet 的可扩展性的属性。

- 最后需要强调的是，Acala 团队建立了一个**社区**，帮助他们赢得了 Polkadot 上的首个平行链插槽拍卖，**作为平行链的** **Acala 网络已于2021年12月18日正式上线**。Acala 平行链将托管去中心化交易所、定价预言机、EVM 支持和多抵押稳定币，以及更多功能，所有这些都由 Acala 社区运行和管理。

## 你在Acala上能做什么  

Acala 旨在成为一站式 Polkadot 生态 DeFi 中心，内置并通过治理添加原生集成的 DeFi 底层基础协议 (primitives)，以及为 dapp 和链开发者提供专门的 EVM 环境。

对于开发者来说，使用构建 Acala 的方式有以下三种：

- **在 Acala 网络上部署许可协议的 [runtime modules/pallets](https://wiki.acala.network/build/development-guide/deploy-ecosystem-modules)**。现在可以更灵活定制和集成。[Ren Protocol 的比特币桥](https://medium.com/acalanetwork/bringing-btc-to-polkadot-acala-x-ren-e7959855d5aa)就是这样实现的。

- 在 Acala EVM 上使用 [Solidity **部署无需许可的智能合约**](https://wiki.acala.network/learn/acala-evm)。这与 BTC、DOT 和 Acala 现有 DeFi 堆栈的聚合跨链流动性完全可组合，并且是一个 dapp 访问 Polkadot 生态系统的登陆平台。[Ampleforth](https://medium.com/acalanetwork/ampleforth-a-defi-building-block-brings-rebasing-currency-and-elastic-finance-to-acala-and-fd1388e8e8fc) 就是这样部署的。

-  [使用 Polkadot 的**跨链消息传递协议**](https://wiki.acala.network/build/development-guide/composable-chains)构建一个链并与 Acala 连接。这目前作为平行链部署在 Rococo 测试网上，作为测试跨链通信、代币转移和其他功能的一种方式。

终端用户可以关注 **Acala portal**（[https://apps.acala.network/](https://apps.acala.network/?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJhdWQiOiJhY2Nlc3NfcmVzb3VyY2UiLCJleHAiOjE2NTU3OTk3NjksImZpbGVHVUlEIjoibTRrTUxKb2xMTkNlUDlxRCIsImlhdCI6MTY1NTc5OTQ2OSwidXNlcklkIjoyMzkwOTU3NX0.KRAktyBCPAtwcTNtMKV4YA4-yJAUCoOz6w4QrKbOjbE)）查看自己的账户余额并开始使用 DeFi 底层基础协议，例如 swapping、farming 和参与治理计划，在 Acala 上启用更多功能和无缝升级。

## Substrate Builders Program 是怎样帮助 Acala 的

Substrate Builders Program 支持 Substrate 生态系统中的构建者。该计划允许构建基于 Substrate 的区块链、应用程序或生态系统组件的团队，从 Parity 的丰富经验和资源中受益，能够帮助他们取得成功。

Substrate Builders Program 委员会评估每个项目的不同愿景、目标和需求，并帮助他们取得成功制定一系列计划。该计划分为三个重大事件，每个重大事件持续三个月。

Parity 的 Runtime 工程师团队会定期检查阶段任务完成质量，为团队实施解决方案提供指导和反馈。在成功完成所有重大事件（或独立上线或通过成为平行链）后，该团队将从 Substrate Builders Program 毕业。

—

如果你有关于 Substrate 或在 Substrate 上构建项目的想法，Substrate Builders 计划可以支持您的项目直到项目上线，并将根据您的项目需求继续提供长期支持。

**了解更多**：

[https://www.substrate.io/builders-program/](https://www.substrate.io/builders-program/?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJhdWQiOiJhY2Nlc3NfcmVzb3VyY2UiLCJleHAiOjE2NTU3OTk3NjksImZpbGVHVUlEIjoibTRrTUxKb2xMTkNlUDlxRCIsImlhdCI6MTY1NTc5OTQ2OSwidXNlcklkIjoyMzkwOTU3NX0.KRAktyBCPAtwcTNtMKV4YA4-yJAUCoOz6w4QrKbOjbE)

**立即申请**：

[https://docs.google.com/forms/d/e/1FAIpQLSfEYJE3X0RQs3Kucqthe4D8zyUcV1yEvyIw98L2X9_78b4BVA/viewform](https://docs.google.com/forms/d/e/1FAIpQLSfEYJE3X0RQs3Kucqthe4D8zyUcV1yEvyIw98L2X9_78b4BVA/viewform?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJhdWQiOiJhY2Nlc3NfcmVzb3VyY2UiLCJleHAiOjE2NTU3OTk3NjksImZpbGVHVUlEIjoibTRrTUxKb2xMTkNlUDlxRCIsImlhdCI6MTY1NTc5OTQ2OSwidXNlcklkIjoyMzkwOTU3NX0.KRAktyBCPAtwcTNtMKV4YA4-yJAUCoOz6w4QrKbOjbE)

