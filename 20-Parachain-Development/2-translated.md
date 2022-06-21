---
title: 'Polkadot 平行链开发指南'
src: https://zhuanlan.zhihu.com/p/514483229
author: 刘博
snapshot-date: 2022-05-14
---

## 开发平行链的首选概述

本指南将涵盖构建 parachain 或 parathread 的动机，可用于促进这一工作的工具，测试的步骤，以及最后，如何在 Polkadot 上启动你的网络。

## 为什么创建平行链？

Parachains 与中继链相连，并由中继链保障。它们受益于集合安全、深思熟虑的治理以及网络的异质分片方法的整体可扩展性。创建 parachain 可以被看作是创建一个第一层区块链，它有自己的逻辑，并在 Polkadot 生态系统中平行运行。

开发人员可以专注于创建最先进的链，利用 Polkadot 的下一代方法。parachain 的一些例子有。

- DeFi（去中心化金融）应用
- 数字钱包
- IoT（物联网）应用
- 游戏
- Web 3.0 基础设施

以及更多。

Polkadot 旨在成为对区块链的最大赌注，Polkadot 的异质多链方法的成功将在 Web 3.0 和去中心化系统的整体推进中起到关键作用。因此，Polkadot 的 parachain 模型在设计时相信，未来的互联网将有许多不同类型的区块链一起工作。

## 部署一条平行链的好处是？

如[Polkadot 白皮书](https://link.zhihu.com/?target=https%3A//polkadot.network/PolkaDotPaper.pdf)所述，parachain 模型试图缓解目前技术栈的五个关键构建故障。

- 可扩展性：在资源上花了多少钱，网络会不会出现瓶颈问题？
- 隔离性：许多人的需求是否在同一框架下得到考虑？
- 可开发性：系统工具、系统支持和整个系统的完整性是否可靠？
- 治理：网络能否保持灵活的发展，并随着时间的推移进行调整？决策能否具有足够的包容性、合法性和透明度，以便对一个分散的系统进行高效的领导？
- 适用性：该技术是否能解决自身的迫切需求？是否需要其他 "中间件" 来弥补与实际应用的差距？

### [共享安全 (Pooled Security)](https://link.zhihu.com/?target=https%3A//wiki.polkadot.network/docs/learn-security)

Parachains 可以通过绑定 DOT 来租赁 Polkadot 网络的安全，来获得一个 parachain 名额。这意味着围绕你的项目建立一个社区并说服验证者参与你的网络安全的社会成本降低了。Polkadot 有很强的安全性，希望从这种安全性中获益的分散的应用项目会希望成为 parachain 来分享这种集合的安全性。

### [链上治理 (Thought-through Governance)](https://link.zhihu.com/?target=https%3A//wiki.polkadot.network/docs/learn-governance)

区块链中的大多数治理系统使用的是链外治理机制。Polkadot 的链上治理鼓励代币持有人的最大参与，并且是无摩擦和透明的。它还可以实现[无障碍升级](https://link.zhihu.com/?target=https%3A//wiki.polkadot.network/docs/learn-runtime-upgrades)。

### 可扩展性

分片式多链网络方法允许本质上的并行计算（处理能力），可以并行处理几个交易。孤立的区块链往往面临着依次处理交易的网络约束，造成瓶颈。

### 可操作性

任何想要向已经连接到 Polkadot 的其他 parachains 实现无信任信息传递的去中心化应用或链，都希望成为 parachain。主权链之间的互操作性涉及到某些限制和复杂的协议，以便在广泛的链中实现。

有了 Polkadot，如果你把你的应用程序构建为一个 parachain，你将会得到这个功能的开箱即用。[XCM 格式](https://link.zhihu.com/?target=https%3A//wiki.polkadot.network/docs/learn-crosschain)允许任何 parachain 通过在它们之间传递消息进行通信。此外，随着连接到其他链的桥梁（如比特币或以太坊的桥梁），Polkadot 的 parachains 也将能够与这些链通信。

> 注意
> 尽管成为 parachain 有很多好处，开发者应该意识到成为 parachain 的挑战，以及建立一个以成为 parachain 为最终目标的区块链对他们的项目来说是否可行。

在 Polkadot 上，你能够把你的区块链的最新区块头放到中继链上。作为一个 parachain，你提交的区块由具有 Wasm 运行时的验证者进行验证，可以存储在中继链上。你还可以使用[XCM 格式](https://link.zhihu.com/?target=https%3A//wiki.polkadot.network/docs/learn-crosschain)与其他 parachain 进行通信：一个抽象的消息传递系统。消息传递在中继链上被跟踪 -- 因此，你可以证明消息的传递，并促进无信任的互动。

由于你可以放置你的区块链的最新区块头，你可以为你的链实现确定性的最终确定。对区块链来说，达到最终确定的困难部分往往是共识，在 parachain 模型中，区块链可以将共识卸载给整个共享网络，并专注于区块生产。由于验证者拥有所有 parachain 的 Wasm 运行时间，你的 parachain 与中继链上的所有人共享验证者池的安全性。

验证者池中的任何验证者都可以帮助验证你的区块链。

## 需要考虑的东西

### [平行链经济 (Para-nomics)](https://link.zhihu.com/?target=https%3A//wiki.polkadot.network/docs/learn-parachains%23parachain-economies)

### 数字民族国家

Parachains 可以被看作是自主代理；作为分散的数字民族国家的网络。Parachains 有自己的社区、规则、经济、治理、财政以及与外部链的关系。因此，parachain 生态系统内的经济政策受制于该 parachain 生态系统的开发者和整体社区；不一定有一个 parachain 应该遵循的经济模式。

此外，成为 parachain 有一个相关的机会成本。理想情况下，你可以通过参与 parachain 的选择过程来增加网络的价值，而这应该作为一个良好的投资回报。

### 连接数字经济

[收集人](https://link.zhihu.com/?target=https%3A//wiki.polkadot.network/docs/learn-collator)作为网络维护者，维护一个完整 parachain 的节点。他们的激励是由原生代币支付的。

- 收集的交易费用

- Parathread 代币的赞助

- - 当一个 parathread 的出价低于原生代币的报酬时，自然会产生区块。



### 平行链对象 (Para-objects)

> 中继链可以承载任意的状态机，而不仅仅是区块链。
> Polkadot 网络将鼓励不同的 para-objects 之间的连接和互操作性。 这里，para-objects 指的是网络上平行运行的对象，一般来说，是可并行的对象。

这些可以是下面的形式：

- 系统级链（永久链）：[租借槽](https://link.zhihu.com/?target=https%3A//wiki.polkadot.network/docs/learn-auction)，[平行线程池](https://link.zhihu.com/?target=https%3A//wiki.polkadot.network/docs/learn-parathreads)
- [桥接](https://link.zhihu.com/?target=https%3A//wiki.polkadot.network/docs/learn-bridges)枢纽
- 嵌套的中继链：[Polkadot 2.0](https://link.zhihu.com/?target=https%3A//wiki.polkadot.network/docs/learn-launch%23polkadot-20)

### 迁移

已经作为 "solochain" 或在孤立环境中运作的项目可能对迁移到 Polkadot 上作为一个 para-object 感兴趣。虽然 parachain 模式有其好处，但它可能不是某些项目的首选策略。

作为迁移到 Polkadot 上的路径，迁移到某个保留槽中的链上可能更可行。

例如，目前通过在最新的插槽拍卖中获得插槽的网络，可以选择在 Kusama 上部署[智能合约](https://link.zhihu.com/?target=https%3A//wiki.polkadot.network/docs/build-smart-contracts)。

## 实现一条平行链

Parachain 实现者指南是一项正在进行的重要工作，由 Parity Tech 维护。[实时版本](https://link.zhihu.com/?target=https%3A//w3f.github.io/parachain-implementers-guide/)是由位于[官方 Polkadot 仓库](https://link.zhihu.com/?target=https%3A//github.com/paritytech/polkadot/tree/master/roadmap/implementers-guide)中的源代码构建的。

### 平行链开发包 (PDK)

PDK 是一套工具，可以让开发人员轻松地创建一个 parachain。在实践中，PDK 将由以下关键组件组成。

- 状态转换函数：你的应用程序从一个状态转移到另一个状态的方法。
- 收集人节点：Polkadot 网络中的一种点对点节点，在 parachain 方面带有一定的责任。

### 关键组件

状态转换函数（STF）可以是一个应用程序从一个状态到另一个状态的抽象方式。Polkadot 对这个 STF 的唯一约束是，它必须容易验证 -- 通常是通过我们所说的见证或证明。它必须如此，因为中继链验证人将需要检查它从收集人节点收到的每个状态是否正确，而不需要实际运行整个计算过程。这些证明的一些例子包括有效性证明（Proof-of-Validity）块或 zk-SNARKs，这些证明需要的计算资源比它们生成的要少。STF 的证明生成中的验证不对称性是允许 Polkadot 在扩展的同时保持高安全保障的整体见解之一。

收集人节点是协议中网络维护者的类型之一。他们负责保持 parachain 的状态和从状态转换函数的迭代中返回的新状态的可用性。它们必须保持在线，以跟踪状态，也跟踪它将在自己和其他 parachain 之间路由的 XCMP 消息。收集人节点负责将简洁的证明传递给中继链的验证者，并跟踪中继链的最新块。实质上，收集人节点也充当了中继链的轻型客户端。更多关于收集人节点的信息，请参见[收集人页面](https://link.zhihu.com/?target=https%3A//wiki.polkadot.network/docs/learn-collator)。

### PDK 包含哪些组件？

目前，唯一的 PDK 是 Parity Substrate 和 Cumulus。Substrate 是一个区块链框架，它提供了区块链的基本构件（如网络层、共识、Wasm 解释器），同时提供了一种直观的方式来构建你的运行时。Substrate 是为了简化创建新链的过程，但它并没有直接提供对 Polkadot 兼容性的支持。出于这个原因，Cumulus，一个附加的库包含了所有的 Polkadot adot 兼容胶水代码。

> Substrate 入门
> 开始使用 Substrate 的最好方法是探索[Substrate 开发者中心](https://link.zhihu.com/?target=https%3A//docs.substrate.io/)，这是一个由[Parity Technologies](https://link.zhihu.com/?target=https%3A//www.parity.io/)建立和维护的在线资源。

### Cumulus

> 积云 (cumulus) 的形状有点像圆点 (dots)；它们一起形成了一个复杂的系统。美丽而实用。

Cumulus 是 Substrate 的一个扩展，它使得任何 Substrate 构建的运行时都可以很容易地成为一个与 Polkadot 兼容的 parachain。

Cumulus Consensus 是 Substrate 的一个共识引擎，它遵循 Polkadot 中继链（即 parachains）。它在内部运行一个 Polkadot 节点，并向客户端和同步算法发出指令，要求遵循哪条链，最终确定，并将其视为正确。

关于 Cumulus 的更详细的描述，请参见[Cumulus 概述](https://link.zhihu.com/?target=https%3A//github.com/paritytech/cumulus/blob/master/docs/overview.md)，对于那些有 Substrate 经验的人来说，可以尝试一下[Cumulus 教程](https://link.zhihu.com/?target=https%3A//docs.substrate.io/tutorials/v3/cumulus/start-relay/)。

Cumulus 仍在开发中，但它的想法是，通过导入 `crates` 并添加一行代码，应该可以简单地采用 Substrate 链并添加 parachain 代码。从[Cumulus 部分](https://link.zhihu.com/?target=https%3A//wiki.polkadot.network/docs/build-pdk%23cumulus)了解最新的 Cumulus 发展。

> 注意
> Substrate 和 Cumulus 从区块链格式的抽象中提供了一个 PDK，但是一个 parachain 甚至不需要是一个区块链。例如，一个 parachain 只需要满足上面列出的两个约束：状态转换功能和收集人节点。 其他一切都由 PDK 的实现者决定。

Cumulus 处理任何 parachain 需要实现的网络兼容开销，以连接到 Polkadot。这包括。

- 跨链消息传递 (XCMP)
- 开箱即用的 Collator 节点设置
- 中继链的一个嵌入式完整客户端
- 区块授权的兼容性

你对构建 PDK 感兴趣吗？请参见[未来的 PDK 部分](https://link.zhihu.com/?target=https%3A//wiki.polkadot.network/docs/build-pdk%23future-pdks)，了解详情。

### 如何建立你的平行链

在用 Substrate 创建了你的链上运行时逻辑后，你将能够把它编译成 Wasm 可执行文件。这个 Wasm 代码 blob 将包含你的链的整个状态转换功能，也是你将项目部署到 Polkadot 作为 parachain 或 parathread 所需要的。

Polkadot 上的验证器将使用提交的 Wasm 代码来验证你的链或线程的状态转换，但这样做需要一些额外的基础设施。验证器需要一些方法来保持最新的状态转换，因为 Polkadot 的节点不需要也是你的链的节点。

这就是整理器节点发挥作用的地方。收集人是你的 parachain 的维护者，执行关键的行动，为你的链产生新的区块候选者，并将它们传递给 Polkadot 验证者，以纳入 Polkadot 中继链。

Substrate 内置了自己的网络层，但不幸的是，它只支持单体链（即不连接到中继链的链）。然而，有一个 Cumulus 扩展，它包括一个收集人节点，并允许你的 Substrate 构建的逻辑与 Polkadot 兼容，作为一个 parachain 或 parathread。

### 未来的 PDK

> 注意
> 你想从头开始建立一个 Parachain 开发工具包吗？Web3 基金会正在为从事这项工作的团队提供资助，了解更多信息并在[W3F 资助页面](https://link.zhihu.com/?target=https%3A//grants.web3.foundation/)申请。

W3F 有兴趣支持的 PDK 的一个例子是一个[roll-up 工具包](https://link.zhihu.com/?target=https%3A//ethresear.ch/t/roll-up-roll-back-snark-side-chain-17000-tps/3675)，它允许开发者创建基于 SNARK 的 parachains。如果我们回顾一下 roll-up write-up，我们会发现系统使用了两个角色：更新状态的用户和将状态更新聚合到一个单一链上更新的操作者。我们应该可以直接看到如何将其转化为平行链术语。roll-up-like 的 parachain 的状态转换函数将从用户的输入中更新状态（在实践中，很可能是 Merkle 树，这将很容易被验证）。操作员将充当收集人节点，它将汇总状态并创建 zk-SNARK 证明，它将交给中继链的验证人进行验证。

如果你或你的团队对开发 PDK 感兴趣，请随时在[W3F 协作库](https://link.zhihu.com/?target=https%3A//github.com/w3f/Grants-Program/)中开辟一个问题以征求意见。这种类型的工作可能会有补助。

## 测试平行链

### Rococo 测试网

[Rococo](https://link.zhihu.com/?target=https%3A//github.com/paritytech/cumulus%23rococo-)是一个为测试 parachains 而建立的测试网。Rococo 利用 Cumulus 和[HRMP](https://link.zhihu.com/?target=https%3A//wiki.polkadot.network/docs/learn-crosschain%23xcmp-lite-hrmp)（Horizontal Relay-routed Message Passing）在 parachains 和 Relay Chain 之间发送转账和信息。每条消息都被发送到中继链，然后从中继链发送到所需的 parachain。

Rococo 目前运行着四个测试系统的平行链：[Statemint Tick](https://link.zhihu.com/?target=https%3A//polkadot.js.org/apps/%3Frpc%3Dwss%3A%2F%2Fstatemint-rococo-rpc.parity.io%23/explorer), [Tick](https://link.zhihu.com/?target=https%3A//polkadot.js.org/apps/%3Frpc%3Dwss%3A//trick-rpc.polkadot.io%23/explorer) 和 [Track](https://link.zhihu.com/?target=https%3A//polkadot.js.org/apps/%3Frpc%3Dwss%3A//track-rpc.polkadot.io%23/explorer)。以及几个外部开发的 parachain。

### 现在在 Rococo 上平行链是什么？

你可以在[这里](https://link.zhihu.com/?target=https%3A//polkadot.js.org/apps/%3Frpc%3Dwss%3A%2F%2Frococo-rpc.polkadot.io%23/parachains)看到所包括的 parachains 的列表。在[这里](https://link.zhihu.com/?target=https%3A//polkadot.js.org/apps/%3Frpc%3Dwss%3A%2F%2Frococo-rpc.polkadot.io%23/parachains/proposals)可以看到被提议的 parachains 清单。

### 获取 ROC

ROC 可以在 Matrix 上的 [Rococo Faucet](https://link.zhihu.com/?target=https%3A//app.element.io/%23/room/%23rococo-faucet%3Amatrix.org) 频道获得。要接收 ROC 代币，请使用以下命令：

```text
!drip YOUR_ROCOCO_ADDRESS
```

### 构建和注册一个 Rococo 平行线程

Rococo 的 parachain 都使用相同的运行时代码。它们之间唯一的区别是用于在中继链上注册的 parachain ID。

你将需要运行一个 Rococo 收集人。要做到这一点，你需要编译以下二进制文件：

```text
cargo build --release --locked -p polkadot-collator
```

一旦编译完成，启动你的平行链的收集人：

```text
./target/release/polkadot-collator --chain $CHAIN --validator
```

如果你对运行和启动你自己的 parathread 或 parachain 感兴趣，Parity Technologies 已经创建了一个[Cumulus 教程](https://link.zhihu.com/?target=https%3A//docs.substrate.io/tutorials/v3/cumulus/start-relay/)来告诉你如何做。在这一过程中遇到困难或需要支持？加入[Parachain 技术 Matrix 聊天频道](https://link.zhihu.com/?target=https%3A//matrix.to/%23/%23parachain-technical%3Amatrix.parity.io)，在那里与其他建设者联系。

### 如何做跨链转账

要在 parachains 之间发送转账，请在 Polkadot-JS Apps 上导航到 "Accounts" > "Transfer"。从这里，你需要选择你正在运行的 parachain 节点。接下来，输入你想发送至另一个 parachain 的金额。请确保选择你想发送金额的正确 parachain。一旦你点击 "Submit" 按钮，你应该看到一个绿色的通知，表明转账成功。

### 向下转账

向下转账是指中继链上的一个账户向其在不同平行链上的账户发送转账。这种类型的转账使用了存管和铸币的模式，也就是说，当 DOT 离开中继链上的发送者的账户并被转入一个平行链上的账户时，平行链就会在平行链上铸造相应数量的代币。

例如，我们可以从 Alice 在中继链上的账户发送代币到她在 parachain 200 上的账户。要做到这一点，我们需要前往 "Network" > "Parachains" 标签，并点击 "Transfer to chain" 按钮。

![img](https://pic3.zhimg.com/80/v2-cb48e50144c13edb33db72408da51616_720w.jpg)

注意这里，我们可以选择发送资金给哪个平行链，指定要发送的金额，并为转账添加任何评论或 memo。

### 向上转账

向上转账发生在从一个 parachain 到中继链上的一个账户。要进行这种转账，我们需要连接到网络上的一个 parachain 节点，并在 "Network" > "Parachains" 标签上。点击 "Transfer to chain" 按钮。

![img](https://pic2.zhimg.com/80/v2-6457318a31f9daea50e9faf0604b3e69_720w.jpg)

注意，切换应该设置为关闭，确保资金流向中继链，而不是另一个平行链。

### 横向转账

横向转账只有在至少有两个不同的注册平行链的情况下才能实现。在真正的 XCMP 中，横向转账将允许消息直接从一个 parachain 发送至另一个。然而，这还没有实现，所以中继链暂时在帮助我们传递消息。横向转账是通过存管模式 (depository model) 进行的，这意味着为了将代币从 200 号平行链转移到 300 号平行链，代币必须已经被 200 号平行链拥有并存放在 300 号平行链上。横向转账被称为 HRMP(Horizontal Relay-Chain Message Passing).

在我们从一个平行链向另一个平行链发送资金之前，我们必须确保该链在接收链上的账户有一些资金。在这个例子中，Alice 将从她在 parachain 200 上的账户发送一些资金到她在 parachain 300 上的账户。

我们可以从 parachain 300 的终端获得该 parachain 账户地址：

```text
2020-08-26 14:46:34 Parachain Account: 5Ec4AhNv5ArwGxtngtW8qcVgzpCAu8nokvnh6vhtvvFkJtpq
```

从 Alice 在中继链上的账户，她可以向 parachain 200 的存管上发送一些金额。

![img](https://pic1.zhimg.com/80/v2-702a1adf3ccb23bdbab23b777808c800_720w.jpg)

Alice 现在能够从她在 parachain 200 的账户向她在 parachain 300 的账户汇款。

![img](https://pic2.zhimg.com/80/v2-548a5ce754f680c388b81a4bd666e5f9_720w.jpg)

### 如何连接到一个平行链

如果你想通过[Polkadot-JS Apps](https://link.zhihu.com/?target=https%3A//polkadot.js.org/apps/%23/)连接到 parachain，你可以通过点击导航左上角的网络选择，选择任何 parachain。



![img](https://pic1.zhimg.com/80/v2-ad67d4204a7da41c64b50c5f94034120_720w.jpg)



在下面的例子中，我们将使用 Rococo 测试网 "Development" 下的 "Custom Node"，遵循[Cumulus 教程](https://link.zhihu.com/?target=https%3A//docs.substrate.io/tutorials/v3/cumulus/start-relay/)。

### 平行链 Playground

你也可以利用 Polkadot-JS Apps 提供的账户功能来测试整个 Parachain 的业务流程（例如，众筹、拍卖、注册）。

在[Westend](https://link.zhihu.com/?target=https%3A//wiki.polkadot.network/docs/maintain-networks%23westend-test-network)上启动一个本地节点，运行：

```text
polkadot --chain=westend-dev --alice
```

然后，通过 Polkadot-JS Apps 连接本地节点。

## 部署

基于 Substrate 的链，包括 Polkadot 和 Kusama Relay 链，使用[SS58 编码](https://link.zhihu.com/?target=https%3A//docs.substrate.io/v3/advanced/ss58/)作为其地址格式。[本页](https://link.zhihu.com/?target=https%3A//github.com/paritytech/ss58-registry/blob/main/ss58-registry.json)作为典型的注册表，供团队查看哪条链对应于某个前缀，以及哪些前缀是可用的。

### Parachain

要将你的 parachain 纳入 Polkadot 网络，你将需要获得一个 parachain 插槽。

parachain 插槽将在公开拍卖中出售，其机制可以在维基的[平行链拍卖页面](https://link.zhihu.com/?target=https%3A//wiki.polkadot.network/docs/learn-auction)上找到。

### Parathread

Parathreads 将不需要 parachain 插槽，所以你将不需要参与蜡烛拍卖机制。相反，你将能够把你的 parathread 代码注册到一个中继链上，并从那时起能够开始参与每块拍卖，以将你的状态转换纳入中继链。

更多关于 parathread 每区块拍卖工作的信息，请看更详细的[parathread 页面](https://link.zhihu.com/?target=https%3A//wiki.polkadot.network/docs/learn-parathreads)。

## 资源

- [Parachains Guide Overview](https://link.zhihu.com/?target=https%3A//docs.substrate.io/how-to-guides/v3/parachains/connect/)
- [Common Good Parachains](https://link.zhihu.com/?target=https%3A//polkadot.network/blog/common-good-parachains-an-introduction-to-governance-allocated-parachain-slots/)
- [The Launch of Parachains](https://link.zhihu.com/?target=https%3A//polkadot.network/blog/the-launch-of-parachains/)
- [Parathreads: Pay-as-you-go Parachains](https://link.zhihu.com/?target=https%3A//medium.com/polkadot-network/parathreads-pay-as-you-go-parachains-7440d23dde06)
- [Polkadot Bridges](https://link.zhihu.com/?target=https%3A//medium.com/polkadot-network/polkadot-bridges-connecting-the-polkadot-ecosystem-with-external-networks-1118916392e3)
- [The Path of a Parachain Block](https://link.zhihu.com/?target=https%3A//polkadot.network/blog/the-path-of-a-parachain-block/)
- [The Path of a Parachain Block (Video)](https://link.zhihu.com/?target=https%3A//www.crowdcast.io/e/polkadot-path-of-a-parachain-block)
- [Polkadot Parachain Slots](https://link.zhihu.com/?target=https%3A//polkadot.network/blog/polkadot-parachain-slots/)
- [How to become a parachain on Polkadot (Video)](https://link.zhihu.com/?target=https%3A//www.youtube.com/watch%3Fv%3DfYc1yolanoE)
- [Trusted Execution Environments and the Polkadot Ecosystem](https://link.zhihu.com/?target=https%3A//polkadot.network/blog/trusted-execution-environments-and-the-polkadot-ecosystem/)