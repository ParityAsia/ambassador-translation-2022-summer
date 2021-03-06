---
proofreader: Jimmy Chu (jimmy.chu@parity.io)
completion-date: 2022-06-19
---

# 平行链

:::info 在Rococo上测试

关于如何参与 Rococo 的众贷和平行链拍卖测试的信息，请参阅平行链开发指南上的 [Rococo内容](https://wiki.polkadot.network/docs/build-pdk#rococo-testnet)。

:::

平行链是一种特定于应用程序的数据结构，它们对于中继链的验证人来说是全局一致并且是可验证的。他们从平行于中继链的概念中取名。一般情况平行链会采用区块链的形式，但没有特定需求它们要是区块链。

![One parachain](https://cdn.jsdelivr.net/gh/VegeBun-csj/Images/771651159350_.pic.jpg)

平行链由于它们的并行性，所以能够并行地处理交易，实现波卡系统的可伸缩性。它们共享整个网络的安全性，并可以通过 XCM 格式与其他平行链进行通信。

平行链由一个称为收集人节点的网络维护程序来运作。收集人节点的作用是维护平行链的完整节点，保留平行链的所有必要信息，并产生新的候选区块来传递给中继链验证人节点，以便验证人节点在波卡的共享状态下进行验证并包含其中。收集人节点的激励是平行链的实现细节。它们不需要在中继链上质押或拥有原生代币，除非平行链上的实现如此规定。

波卡宿主机 (Polkadot Host, 简称 PH) 需要平行链上执行的状态转换被转换成 [Wasm](https://wiki.polkadot.network/docs/learn-wasm) 可执行文件。对于发生在平行链上的新的状态转换的证明，必须在确认状态转换发生之前，由中继链上的验证人根据已注册的状态转换函数 (state transition function，简称 STF) 进行验证。平行链逻辑的关键约束是，它必须可以由中继链验证人验证。验证最常见的形式是状态转换的绑定证明，即一个Proof-of-Verification (PoV) 区块，它是由一个或者多个平行链的收集人提交给验证人进行核实。

## 平行链经济生态

平行链可能拥有自己原生代币的经济生态。像 (Proof-of-Stake，简称 PoS) 之类的方案通常用于选择验证人来处理验证并最终确认；平行链则不需要处理这些。然而，由于波卡并不需刻意知道平行链的实现方法，所以平行链上也可以实现一个可质押的代币，但通常都不须要这么做的。

收集人可能会被本地平行链代币的通胀所激励，也可能还有其他方法来激励收集人节点，而不涉及通胀原生的平行链代币。

原生平行链代币中的交易费用也可是平行链自身实现的选择。波卡对于平行链如何决定交易的原始有效性并没有制定严格的规则。例如，平行链可以实现为，交易必须向收集人支付最低费用才能有效。中继链会加强这种有效性。同样，平行链也可以不包含这些，而波卡仍会履行其有效性。

平行链不需要有自己的代币。但如果他们这样做了，那平行链则需决定他们代币的经济理由，而不是波尔。

## 平行链枢纽中心

尽管波卡可以在平行链之间启用跨链功能，但一条平行链从发送消息，直到接受方平行链收到消息，难免会存在一定延迟。在最好的情况下，这个消息的延迟至少为两个区块 - 一个块是用来发出消息，另一个块用于平行链对接收的消息进行处理并发生出结果区块。但是，在某些情况下，我们可能看到如果队列中有许多待处理的消息，或者两条平行链上其中一条没有节点运行以致不能快速串通网络消息，那么消息的延迟会更高。

由于发送跨链消息会有一定的延迟，一些平行链计划成为某个行业的中心。例如，平行链项目 [Acala](https://acala.network/) 计划成为去中心化金融 (DeFi) 应用的枢纽中心。许多DeFi 应用利用一种称为 *可组合性* 的属性，可组合性意味着一个应用的功能可以与其他应用协同组合以创建新的应用。这方面的一个例子包括闪电贷款，它借用某些资金来执行一些链上逻辑，然后在交易结束时偿还。

跨链延迟的另一个问题是，与单条区块链相比，平行链之间的可组合性减弱。**这问题所有分片区块链设计都通用的，包括波卡、以太坊 2.0，及其他**。解决这个问题的方法是引入平行链枢纽中心，它保持了较强的单块可组合性。

## 获取平行链插槽

波卡支持有限数量的平行链，目前估计约有100个。由于插槽的数量是有限的，有以下几种方法来分配它们：

- 治理授权的平行链，或 "公益" 平行链 (common good parachains)
- 拍卖获取的平行链
- 平行线程 (parathreads)

[“公益”平行链](https://wiki.polkadot.network/docs/learn-parachains#common-good-parachains) 由波卡的链上治理系统分配，它们被认为是网络的 “公共利益”，比如作为连接其他网络或链的桥接。它们通常被认定为系统级别的链或公用的链，没有一个经济模型，以及可以帮助从中继链上删除交易，使得更有效的处理平行链。

[拍卖获取的平行链](https://wiki.polkadot.network/docs/learn-auction) 是在一个公开的拍卖中被授权的。平行链团队可以使用自己的 DOT 代币竞标，也可以使用[众贷功能](https://wiki.polkadot.network/docs/learn-crowdloans) 从社区借取代币。

[平行线程](https://wiki.polkadot.network/docs/learn-parathreads) 与平行链具有相同的 API，但它的执行是以现收现付为基础，对每个区块进行拍卖。

### 插槽过期

当一个平行链赢得拍卖时，它投标的代币将被保留直到租期结束。被保留的金额是无法转出，也不能用于质押。在租期结束时，代币会被解除保留。没有获得新租期的平行链将自动成为平行线程。

## 公益平行链

“公益” 平行链是为使整个生态系统受益的功能而保留的平行链插槽。通过将一组平行链插槽分配给公益平行链，整个网络可以获得这些平行链所提供的价值，否则会由于搭便车问题，这些平行链会得不到应有的资金支持。它们不是通过平行链插槽拍卖分配的，而是通过链上[治理](https://wiki.polkadot.network/docs/learn-governance) 系统分配的。一般来说，公益平行链的租期不会到期；只会在治理通过下移除它们。

读者可通过 [这篇波卡博客文章](https://polkadot.network/blog/common-good-parachains-an-introduction-to-governance-allocated-parachain-slots/)以及 [公益平行链页](https://wiki.polkadot.network/zh-CN/docs/learn-common-goods)来了解更多信息。

### 例子

一些平行链的例子：

- **加密的联盟链**：这些链可能是不向公众泄漏任何信息的私有链，但由于 XCMP 协议的性质，仍然可以与它们进行去信任的交互。
- **高频交易链**：这些链可以通过进行某些权衡取舍或优化，在短时间内计算大量交易。
- **隐私链**：这些链通过使用新颖的加密技术，从而不向公众泄露任何信息。
- **智能合约链**：这些链可以通过部署智能合约代码并在其上实现额外的逻辑。

## 常见问题

### 什么是 “平行链共识”

“平行链共识” 是特别的，因为它将遵循波卡中继链。平行链不能使用其他具有自身确定性的共识算法。只有主权链 (必须通过平行链桥接到的中继链) 才能决定自己的共识机制。平行链决定了区块是如何生成以及由谁生成。而波卡则保证有效的状态转换。在中继链的上下文之外执行区块确认操作则超出了波卡提供的信任范围。

#### 不是基于 Substrate 构建的平行链呢?

Substrate 提供了 [FRAME Pallets](https://docs.substrate.io/v3/runtime/frame/) 作为其框架的一部分来无缝实现基于 Rust 的区块链。FRAME 的一部分是可用于共识的 pallets。波卡作为一个基于 Substrate 的链，依赖于 BABE 作为区块生成的方案，而 GRANDPA 作为共识机制最终确认性的一部分。总的来说，这是一个 [混合共识模型](https://wiki.polkadot.network/docs/learn-consensus#hybrid-consensus)，其中块的产生和最终确认是分开的。平行链只需要生成区块，因为它们可以依赖于中继链来验证状态转换。因此，就算平行链不是基于 Substrate 构建的，平行链也可以有自己的区块生成逻辑，而其中[收集人](https://wiki.polkadot.network/docs/learn-collator) 则是区块的生成者。

### 平行链的插槽如何分配？

平行链槽可以通过拍卖获得，请参阅[平行链槽](https://wiki.polkadot.network/docs/learn-auction) 的文章。此外，一些平行链插槽将被留出来运行平行线程，它们会以每个区块为基础进行投标，以被包含在中继链中。

### 当验证人的数量低于某个阈值时，平行链会发生什么?

每个平行链的验证人的最小安全比率为 5：1。借助足够大的验证人集合，其分布的随机性以及[可用性和有效性](https://wiki.polkadot.network/docs/learn-availability) 将确保其安全性。但是，如果云服务商发生重大事故或出现灾难性网络连接问题，则可以合理地预期每条链的验证人数量会减少。

根据有多少验证人离线，结果会有所不同。

如果只有少数几个验证人离线，那些验人集太小而无法验证一个块的平行链将跳过这些块。它们的区块生产速度将减慢，并以每 6 秒增加，直到问题得以解决，即最优的验证人数量再次出现于该平行链的验证人组中。

如果有 30% 到 50% 的验证人离线，可用性将受到影响，因为我们需要设置三分之二的验证人集来支持平行链的候选区块。换句话说，所有的平行链将停止，直到问题得到解决。区块最终确认性也会停止，但是中继链上的低价值交易还足够安全地执行，尽管分叉会经常出现了。一旦所需的验证人数量再次进入验证人集中，平行链将恢复生成区块。

考虑到收集人是他们正在运行的中继链和平行链的完整节点，他们能够在中断发生时识别出中断，并且应该停止产生候选块。同样，他们应该很容易识别什么时候可以安全重新生成区块 —— 可能是基于最终确认性延迟、验证人集的大小，或其他一些在 [Cumulus](https://github.com/paritytech/cumulus) 内尚未确定的因素。

### 平行链开发工具集 (Parachain Development Kits, 简称 PDK)

平行链开发工具集是一组工具允许开发人员将自己的应用程序创建为平行链。有关更多信息，请参阅 [PDK内容](https://wiki.polkadot.network/docs/build-pdk#parachain-development-kit-pdk)。

## 其他资源

- [波卡：平行链](https://medium.com/polkadot-network/polkadot-the-parachain-3808040a769a) - 波卡的联合创始人 Rob Habermeier 在 2017 年发表的平行链博客文章，他将平行链作为 "区块链的一种更简单的形式，它连接到中继链，中继链提供了安全性，而不是自己提供安全性。中继链为连接的平行链提供了安全性，也为它们之间的安全消息传递提供了保证。"

- [平行链区块的历程](https://polkadot.network/the-path-of-a-parachain-block/) - 一篇说明平行链与中继链如何相互作用的技术文章。
