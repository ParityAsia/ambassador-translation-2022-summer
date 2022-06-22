---
proofreader: Jimmy Chu (jimmy.chu@parity.io)
completion-date: 2022-06-22
---

# 建立在 Substrate 之上：Centrifuge 链

目前，金融信息和资金的流动是不透明和无法互通的。当今存在的数据孤岛（data silos）和碎片化为企业（尤其是中小企业）获得快速、可持续增长所需融资产生了障碍。Centrifuge 是 Centrifuge OS 和 Tinlake 背后的团队，正在提供下一代方法来取代发票和支付的传统金融供应链。

Centrifuge OS 是一个开放的去中心化平台，用于连接传统的发票和区块链上的和代币资产，具体地是到 Centrifuge 链上。该区块链将允许企业交换经过验证的商业文件（例如发票）并将这些资产代币化以方便作进行进一步融资。

建基于 Substrate，Centrifuge 链的测试网 “Amber” 已经上线。那里正在试用最新实现的功能。该测试网奠定了开放给任何企业使用的基础，同时保持其数据的所有权，包括经过验证的公司详细信息、声誉、业务关系，和后续交易。与 Centrifuge Chain 相结合的点对点层，允许企业交换业务文档（例如发票）并将这些资产代币化。Tinlake，其上有着证券化协议，通过将这些代币化资产更容易获得融资来完善 Centrifuge OS，从而释放出以前无法获得的价值。

## 改变获得报酬的方式

Paperchain 是采用 Centrifuge 为其用户提供更多融资渠道的项目之一。Paperchain 为数字内容创建者整合他们使用的许多服务（如 YouTube、Spotify、Apple iTunes 等）的收入和性能数据的一种新方式。这些服务中的每一项都有其专有系统，用于向创作者和艺术家分配版税，它们自己碎片化的数据结构，及有不同的支付方法和时间框架。更糟糕的是，支付流程有时涉及多达 30 个独立的参与方，而每个参与方都会导致整个金融供应链的阻塞，这使得该过程对所有相关人员来说都极其昂贵和耗时。最后，导致艺术家有时必须等待长达一年才能结清款项。

Paperchain 已经完成了分析来自多个内容提供商的数据流的工作，以透明、准确地预测艺术家的收入，并正在与 Centrifuge 团队密切合作，以启用一个新系统，使版税能够以更及时的方式分配。通过利用保护隐私的不可替代代币 (NFT)，结合来自内容发行商的不可伪造数据流，创建了一种新的市场范式，允许借贷市场快速向艺术家付款。Centrifuge 链中内置的 NFT 基于以太坊的 ERC-721 标准。该演示已在以太坊主网上得到验证，随着上个月 Centrifuge 的测试网推出，更多优化正在进行中。

## Centrifuge 为何选择 Substrate？

Substrate 是构建区块链的框架，使组织能够只需要构建特定目的所需的逻辑。由于 Substrate 的架构已包含了点对点网络、密码库、可扩展共识算法，和其他开箱即用的区块链底层组件，Centrifuge 能够专注于其链的业务逻辑。这使得 Centrifuge 可以利用上 Parity 构建区块链方面的许多经验并重用这些通用功能，以便他们可以更快速实现所需的功能。

虽然整个 Centrifuge OS 包括 Ethereum 和 Substrate，但他们需要一个对开发人员友好的框架，能够提供创建成功生态系统所需的功能并及时推出完整的功能。Centrifuge 团队正在使用 Substrate 进行四个主要优化领域：

**速度**：针对特定于金融供应链的用例进行优化，可以更快地执行逻辑和交易安全性。借助 Substrate，Centrifuge 可以自定义低级协议逻辑以确保其速度。

**实惠的交易**：Centrifuge 使用 Substrate 专门构建了一个快速，有着实惠交易的链。他们选择了 Substrate 的权益证明共识来进一步降低交易成本。

**存储**：使用 “状态租金” 模型要求用户为其数据的持续可用性付费来鼓励去中心化。这样使运行节点所需的资源更少。Substrate 允许开发人员创建尖端的存储优化技术。

**用户/开发人员体验**：构建自定义 Substrate 链可以快速改进 Centrifuge 的用户和开发人员的体验。用户需要隐私，Substrate 允许 Centrifuge 针对他们需要的功能，直接打造这个功能。

<iframe width="560" height="315" src="https://www.youtube.com/embed/gFcb7jeEdjg" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

来自 Centrifuge 的 Cassidy 和 Philip 展示 Centrifuge OS

## 组装正确的模块 (Pallets)

Pallets 是 Substrate 框架中的模块。它们减少技术开销，允许开发人员和创始人自行组合和定制化，实现了对 Substrate 链的高度可扩展性，并带有一个使用和可强化该模块的活跃开发人员社区。Centrifuge 已成为 Substrate 生态系统的贡献者，为 Centrifuge OS 和更广泛的 Substrate 社区提供自己的模块和工具。

Centrifuge 创建的模块有：

* 零知识验证者：验证为链上文档隐私生成的零知识证明（ZK 验证者）。

* 多重签名帐户：创建 总共(m)-只需(n) 的多重签名帐户以提高安全性和组件的控制/访问权限（多重签名账户）。

* 基于 ERC-721 的 NFT：创建不可替代的代币，用于关联链上文档，例如 Centrifuge 链上的发票（NFT）。

通过利用许多预包装的模块，Centrifuge 团队能够在 Substrate 上快速启动他们的 Amber 测试网。由于沙盒化的 WebAssembly 运行环境，链上升级变得轻松和快速，他们新增的模块可以不用要求链分叉或所有节点更新其客户端。

Centrifuge 还一直在 Substrate 运行环境外开发两个非常重要的功能：Golang JSON-RPC 接口和 Substrate-Ethereum 桥。Centrifuge JSON-RPC 接口是用 Go 编写的，因此他们庞大的 Go 开发人员团队可以更轻松地与 Centrifuge 链交互，而无需深入研究 Rust 的细节。其他熟悉 Go 的开发人员团队也可以使用它。如前所述，Centrifuge 正在基于 Substrate 的链和以太坊主网，开发 Centrifuge OS。因此其中一项重要功能是在这些链之间建立一座桥梁以启用它们打造的新功能，包括依赖于以太坊但在 Centrifuge 链上使用的非同质化代币(NFT)。

大家可以订阅 Parity 的通讯来了解更多关于 Substrate  上构建的团队的最新信息。要跟进 Centrifuges 的工作，请关注他们的 Twitter 或 Medium 帐号以获得更新。
