---
title: '建立在 Substrate 上：Centrifuge 链'
src: https://docs.google.com/document/d/1rI5P_nbxgs1d7-mafnQZCn7DZn-qmkKWobZPCJFSOSI/edit?usp=sharing
author: 俊龙
snapshot-date: 2022-06-21
---

# 建立在 Substrate 上：Centrifuge 链

目前，金融信息和资金的流动是不透明和無法互通的。当今存在的数据孤岛（data silos）和碎片化为企业（尤其是中小企业）获得快速、可持续增长所需融资产生了障碍。Centrifuge 是 Centrifuge OS 和 Tinlake 背后的团队，正在提供下一代方法来取代发票和支付的传统金融供应链。

Centrifuge OS是一个开放的去中心化平台，用于连接区块链上的传统发票和基于代币的资产，特别是 Centrifuge 链。该区块链将允许企业交换经过验证的商业文件（例如发票）并将这些资产代币化以进行进一步金融活动。

基于 Substrate 构建的Centrifuge 链测试网“Amber”已经上线。在那里，正在测试最新功能的实现。该测试网为任何企业使用链奠定了基础，同时保持其数据的所有权，包括经过验证的公司详细信息、声誉、业务关系和后续交易。点对点层与 Centrifuge Chain 相结合，允许企业交换业务文档（例如发票）并将这些资产标记化。Tinlake 是建立在上面的证券化协议，它通过使这些代币化资产更容易获得融资来完善 Centrifuge OS，从而释放以前无法获得的价值。

## 改变我们获得报酬的方式

Paperchain 是采用 Centrifuge 为其用户提供更多融资渠道的项目之一。Paperchain 是数字内容创建者整合他们使用的许多服务（如 YouTube、Spotify、Apple iTunes 等）的收入和性能数据的一种新方式。这些服务中的每一项都有其专有系统，用于向创作者和艺术家分配版税，它们自己的碎片化数据结构，所有这些都具有不同的支付方法和时间框架。更糟糕的是，支付流程有时涉及多达 30 个单独的参与方，每个参与方都会导致整个金融供应链的阻塞，这使得该过程对所有相关人员来说都极其昂贵和耗时。最后，艺术家有时必须等待长达一年的时间才能结清款项。

Paperchain 已经完成了分析来自多个内容提供商的数据流的工作，以透明、准确地预测艺术家的收入，并正在与 Centrifuge 团队密切合作，以启用一个新系统，使版税能够以更及时的方式分配。通过利用保护隐私的不可替代代币 (NFT)，结合来自内容发行商的不可伪造数据流，创建了一种新的市场范式，允许借贷市场快速向艺术家付款。Centrifuge 链中内置的 NFT 基于以太坊的 ERC-721 标准。该演示已在以太坊主网上得到验证，随着上个月 Centrifuge 的测试网推出，更多优化正在进行中。

## Centrifuge 为什么选择 Substrate？

Substrate 是构建区块链的框架，使组织能够仅构建特定目的所需的逻辑。由于 Substrate 的架构包括点对点网络、密码库、可扩展共识算法和其他开箱即用的区​​块链原语，Centrifuge 能够专注于其链的业务逻辑。这使得 Centrifuge 可以利用 Parity 在构建许多区块链方面的经验并使用通用功能，以便他们可以更快地移动并快速实现所需的功能。
虽然整个 Centrifuge OS 包括 Ethereum 和 Substrate，但他们需要一个对开发人员友好的框架，能够提供创建成功生态系统所需的功能并及时推出功能完整性。Centrifuge 团队正在使用 Substrate 进行四个主要优化领域：

**速度**：针对特定于金融供应链的用例进行优化，可以更快地执行逻辑和交易安全性。借助 Substrate，Centrifuge 可以自定义低级协议逻辑以确保速度。

**能负担的交易**：Centrifuge 使用 Substrate 构建了一个针对快速、负担得起的交易进行优化的链。他们选择了 Substrate 的股权证明共识来进一步降低交易成本。

**存储**：要求用户为其数据的持续可用性付费的“状态租金”模型鼓励去中心化，因为运行节点所需的资源更少。Substrate 允许开发人员创建尖端的存储优化。

**用户/开发人员体验**：构建自定义 Substrate 链可以快速改进 Centrifuge 的用户和开发人员体验。用户需要隐私，这是 Substrate 允许 Centrifuge 直接构建的东西——从一开始就针对他们需要的功能。

<iframe width="560" height="315" src="https://www.youtube.com/embed/gFcb7jeEdjg" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

来自 Centrifuge 的 Cassidy 和 Philip 展示 Centrifuge OS_

## 组装正确的pallets

Pallets 是 Substrate 框架中的模块。它们允许开发人员和创始人想要的可组合性和定制化，因为它们减少了技术开销，实现了对 Substrate 链的高度可扩展性，并带有一个使用和增强托盘的活跃开发人员社区。Centrifuge 已成为 Substrate 生态系统的贡献者，为 Centrifuge OS 和更广泛的 Substrate 社区提供自己的pallets和工具。

Centrifuge 创建的pallets：

* 零知识验证者：验证为链上文档隐私生成的零知识证明。（ZK 验证者）

* 多重签名帐户：创建 m-of-n 帐户以提高安全性和组控制/访问权限。（多重签名账户）

* 基于 ERC-721 的 NFT：创建不可替代的代币，用于关联链上文档，例如 Centrifuge 链上的发票。（NFT）

通过利用许多预包装的托盘，Centrifuge 团队能够在 Substrate 上快速启动他们的 Amber 测试网。由于沙盒化的 WebAssembly 运行时，链上升级的轻松和速度，可以添加他们将完成的托盘，而无需分叉链或要求所有节点更新其客户端软件。

Centrifuge 还一直在开发 Substrate 运行时之外的两个非常重要的功能：Golang JSON-RPC 接口和 Substrate-Ethereum 桥。Centrifuge JSON-RPC 接口是用 Go 编写的，因此他们庞大的 Go 开发人员团队可以更轻松地与 Centrifuge 链交互，而无需深入研究 Rust 的细节。其他更熟悉 Go 的开发人员和团队也可以使用它。如前所述，Centrifuge 正在开发 Centrifuge OS，它包括基于 Substrate 的链和以太坊主网。因此，在这些链之间建立一座桥梁以启用它们正在创建的一些功能至关重要，包括依赖于以太坊但在 Centrifuge 链上使用的不可替代代币(NFT)。
