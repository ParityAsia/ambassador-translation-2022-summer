---
title: '波卡跨共识消息 (XCM) 格式'
src: https://zhuanlan.zhihu.com/p/500758866
author: Akagi201 (https://github.com/Akagi201)
snapshot-date: 2022-06-18
---

# 波卡跨共识消息 (XCM) 格式

**版本 3，进行中。**
**作者: Gavin Wood.**


本文件详细介绍了基于 Polkadot 的链间消息传递的消息格式。它描述了正式的数据格式，任何可能额外需要的环境数据以及数据报的相应含义。

## **1** 背景
为不同的共识系统间提供通信便利是有好处的。这包括智能合约和其环境之间的信息，主权区块链之间通过桥的信息，以及受同一共识支配的分片之间的信息。不幸的是，每个人都有自己的信息传递方式和标准，或者根本没有标准。

XCM 旨在抽象出这些系统之间的典型消息意图，并为向前兼容、可扩展和实用的通信数据报提供一个基本框架，促进全局共识世界内不同数据系统之间的典型交互。

来自 IPFS 项目的概念，特别是自我描述格式的想法，贯穿始终，并引入了两种新的自我描述格式，用于指定资产（基于 `MultiAsset`）和共识系统位置（基于 `MultiLocation`）。

Polkadot 有三个主要的传输系统，用于在链之间传递消息，所有这些都将使用这种格式。XCMP（有时被称为 HRMP）以及两种 VMP：UMP 和 DMP。

- **XCMP** Cross-Chain Message Passing 平行链之间的安全消息传递。有两个变种。Direct和Relay。
在Direct模式下，消息数据在平行链之间直接传递，在中继链方面是 O(1)，并且是非常可扩展的。
在Relayed模式下，消息数据通过中继链传递，并在 VMP 上捎带着进行。它的可扩展性要差得多，特别是平行线程 (parathreads) 可能会因为队列的过度增长而无法收到消息。


- **VMP** Vertical Message Passing 在中继链本身和平行链之间传递消息。两种情况下的消息数据都存在于 Relay-chain 上。这包括。
  - **UMP** Upward Message Passing 从一个平行链到中继链的消息传递。
  - **DMP** Downward Message Passing 从中继链到平行链的消息传递。


### **1.1** XCM 通信模型
XCM 是围绕 4 个 'A' 来设计的：

- 异步的 (Asynchronous): XCM 消息不会假定发件人在完成时是阻塞的。
- 绝对的 (Absolute): XCM 消息保证准确、有序、及时地被传递和解释。
- 非对称性 (Asymmetric): XCM 消息没有结果。任何结果都必须用额外的消息单独告知发送者。
- 不可知论 (Agnostic): XCM 对传递信息的共识系统的性质不做任何假设。
XCM 提供了这些绝对的保证，这使得它实际上是非对称的，而其他非绝对协议会发现这很困难。

作为不可知论，XCM 不仅仅适用于平行链和/或中继链之间的消息，而且是适用于通过一个或多个桥连接的不同链之间的信息，甚至适用于智能合约之间的消息。使用 XCM，上述所有各方都可以相互通信，或通过对方通信。

例如，完全可以想象，使用 XCM，托管在 Polkadot 平行链上的智能合约可以通过 Polkadot 将其拥有的不可伪造的资产转移到位于另一个平行链上的 Ethereum-mainnet 桥，通过 Polkadot-Kusama 桥在托管在 Kusama 平行链上的第三个专门的 Substrate NFA 链上登记所有权的转移，从而进入 Ethereum 主网上控制的账户。

### **1.2** XCVM
XCM 格式在很大程度上借鉴了一个高度特定领域的虚拟机，称为 跨共识虚拟机 或 XCVM 。XCM 消息直接对应于具有版本意识的 XCVM 程序，而 XCVM 指令集代表了 XCM 消息可以组成的一系列动作。

XCVM 是一种基于寄存器的虚拟机，其寄存器都不是通用的。因此，XCVM 指令格式、机器寄存器集和交互定义是 XCM 报文格式的主要内容，本文档的大部分文字都是用来表达这些定义的。



### **1.3** 词汇表
- 共识系统 一个链、合约或其他全局的、封装的、状态机单例。它可以是存在于共识中的任何程序化的状态转换系统，可以发送/接收数据报。可以通过一个 `MultiLocation` 值来指定（尽管不是所有这样的值都能确定一个共识系统）。例子包括 Polkadot Relay chain, XDAI Ethereum PoA chain, Ethereum Tether smart contract.
- 位置 一个共识系统，或一个存在于其中的可寻址账户或数据结构。例如，Polkadot Relay-chain 上的财政部账户，Edgeware parachain 上的主要 Web3 基金会账户，Edgeware parachain 本身，Web3 基金会的 Ethereum multisig 钱包账户。由`MultiLocation`指定。
- 主权账户 (Sovereign Account) 一个由特定的共识系统控制的账户，在一些其他共识系统内。这样的账户可能有很多，也可能只有一个。如果有很多，那么这就假定并确定了一个独特的主要账户。
- XCVM 交叉共识的虚拟机，XCM 信息的定义很大程度上依赖于此。
- Reserve Location 在特定（衍生）Consensus System上作为特定资产的储备的Consensus System。储备共识系统总是被衍生品所知道。它将有一个衍生品的主权账户，其中包含衍生品资产的全部抵押品。
- 起源 某一信息是由其（直接和立即）传递的共识系统。指定为 "多处"。
- 收件人 某一信息被传递到的共识系统。指定为 `MultiLocation`。
- 远程传输 (Teleport) 在一个地方销毁一个资产（或资金/代币/货币的数量），并在第二个地方铸造一个相应的数量。想象一下星际迷航中的传送器。这两个地方的性质不一定相等（例如，可以是一个销毁资产的 UTXO 链和一个铸造资产的基于账户的链）。任何一个地方都不能作为另一个地方的储备或衍生品。虽然代币的性质可能不同，但两个地方都不比另一个地方更权威。这只有在 STF 和它们之间的有效性/财务性/可用性都存在双边信任关系的情况下才有可能。
- 转移 资金从一个控制机构转移到另一个控制机构。这是在同一链条或整体资产所有权环境中，在同一抽象层次上。


### **1.4** 文档结构
该格式被定义为五个主要部分。顶层数据报格式在第 2 节中规定。XCVM 在第 3、4 和 5 节中定义。第 6 和第 7 节定义了 `MultiLocation` 和 `MultiAsset` 格式。第 8 节中规定了消息的例子。


### **1.5** 编码
所有的数据都是 SCALE 编码的。基本类型用 Rust 语言格式表示，例如。

- 32 位有符号的整数被写成 `s32`。
- 64 位无符号整数写成 `u64`。
当一个类型的无序列表 -- 可能是命名的 -- 被给出时，它意味着单个类型值的简单串联。例如，一个 64 位无符号的 "索引"，后面是一串字节的 "数据"，可以写成。

- `index: u64`
- `data: Vec<u8>`


## **2** 基础顶级格式
我们将顶级的 XCM 数据类型命名为 `VersionedXcm`。它是这样定义的。

- `version: u8`: XCM 的版本。
- `message`: 消息；不透明的，除非版本是已知的和明确定义的。
本文件只定义了版本标识符为`2`的 XCM 消息。版本低于此值的消息可能在本文件的早期版本中定义。

鉴于目前定义的 XCM 版本是`2`，因此我们可以将消息格式具体化为。

- `version: u8 = 2`。XCM 的版本。
- `message: Vec<Instruction>`。
因此，消息是简单的字节 `2`，用于识别当前的版本，加上一系列 SCALE 编码的 XCVM 指令。

任何给定的 XCM 消息的效果被定义为 XCVM 在给定 `message`（在 XCVM 中被称为Original Programme）和消息来源的位置（在 XCVM 中被称为Original Origin）时正确初始化所采取的行动。初始化的具体内容将在下一节中定义。

## **3** XCVM 寄存器
XCVM 包含几个机器寄存器，这些寄存器一般不能随意设置，而是以特定的值开始，只有在特定的情况下和/或遵守特定的规则才可以改变。

这些寄存器被命名。

- 程序 (Programme)
- 程序计数器 (Programme Counter)
- 错误 (Error)
- 错误处理程序 (Error Handler)
- 附录 (Appendix)
- 起源 (Origin)
- 持有 (Holding)
- 过剩的权重 (Surplus Weight)
- 退款的权重 (Refunded Weight)

### **3.1** 程序 (Programme)
类型为Vec<Instruction>，初始化为原始程序 (Original Programme) 的值。

表示 XCVM 当前执行的指令方案。在执行完最后一条指令或发生错误后，该程序可能会被改变。

### **3.2** 程序计数器 (Programme Counter)
类型为`u32`，初始化为`0`。

表示当前正在执行的指令在程序寄存器中的索引。在每条指令成功执行后增加`1`，当程序寄存器被替换时重置为`0`。


### **3.3** 错误 (Error)
类型为`Option<(u32, Error)>`，初始化为`None`。

表示程序执行过程中发生的最后一个已知错误的信息。只在程序遇到错误时设置。可以随意清除。两个内部字段表示错误发生时程序计数器的值和发生的错误类型。


### **3.4** 错误处理程序 (Error Handler)
类型为`Vec<Instruction>`，初始化为空列表。

表示任何在错误情况下应该运行的代码。当程序遇到错误时，这个寄存器被清空，其内容用于替换程序寄存器。

### **3.5** 附录 (Appendix)
类型为`Vec<Instruction>`，初始化为空列表。

表示任何应该在当前程序之后运行的代码。当一个程序成功到达终点，或者在错误处理程序为空的情况下，这个寄存器被清空，其内容用于替换程序寄存器。

### **3.6** 起源 (Origin)
类型为 `Option<MultiLocation>`，初始化后，其内部值为原来的起源 (Original Origin)。

表示当前程序运行的位置的授权。可以随意重置为`None`（意味着没有权限），也可以随意设置为一个严格的内部位置（意味着严格的权限子集）。

### **3.7** 持有 (Holding)
类型为`MultiAssets`，初始化为空集（即没有资产）。

表示在程序控制下存在的一些资产，但在链上没有表示。可以认为是 "未使用的" 资产的非持久性登记。

### **3.8** 过剩的权重 (Surplus Weight)
类型为`u64`，初始化为`0`。

表示对原始程序的估计必须是高估的权重。这包括由于之前的指令发生错误而没有发送的指令的任何权重，以及由于成功的结论而没有生效的错误处理程序和在必然的保守估计后知道其权重的指令。

### **3.9** 退款的权重 (Refunded Weight)
类型为`u64`，初始化为`0`。

表示已经退还的剩余权重的部分。在 XCM 平台没有使用的，不需要为执行支付的部分。

## **4** 基本的 XCVM 操作
XCVM 的操作是状态机中常见的指令 （获取 - 分配）的循环。该循环的步骤是

- 如果程序寄存器的值为空，则停止。
- 试图通过程序计数器寄存器对程序寄存器进行索引来获取指令。
- 如果有指令存在，则执行该指令（见下一节），并检查其是否导致错误。
  - 如果没有导致错误，那么。
    - 将程序计数器增加`1`。
- 如果它导致了错误，那么。
  - 相应地设置错误寄存器。
  - 将程序计数器重置为`0`。
  - 将程序寄存器设置为错误处理寄存器的值。
  - 重置错误处理寄存器，使其为空。

- 如果没有指令存在。
  - 将程序计数器重置为 `0`。
  - 将程序寄存器设置为附录寄存器的值。
  - 用错误处理程序寄存器中的内容的估计重量来增加剩余重量寄存器。
  - 将错误处理程序寄存器和附录寄存器都重置为空。


与基本的获取/分配循环的区别在于增加了错误处理程序和附录寄存器。值得注意的是：

- 当程序成功完成时，错误寄存器不被清除。这使得附录寄存器中的代码可以利用其值。
- 当一个程序完成时，无论成功与否，错误处理寄存器被清空。这可以确保以前的程序中的任何错误处理逻辑不会影响后来的附加代码。


## **5** XCVM 指令集
XCVM 指令类型 (Instruction) 表示为每个单独指令的标记联合 (Rust 语言中的 enum)，包括它们的操作数，如果有的话。由于这是 SCALE 编码，一条指令被编码为它在指令列表中的 0-索引位置，并与它的操作数的 SCALE 编码相连接。

这些指令按顺序排列如下：

- `提取资产 (WithdrawAsset)`
- `保留的储备资产 (ReserveAssetDeposited)`
- `接收远程传输的资产 (ReceiveTeleportedAsset)`
- `查询响应 (QueryResponse)`
- `转移资产 (TransferAsset)`
- `转移保留资产 (TransferReserveAsset)`
- `交易 (Transact)`
- `HrmpNewChannelOpenRequest`
- `HrmpChannelAccepted`
- `渠道关闭 (HrmpChannelClosing)`
- `清除起源 (ClearOrigin)`
- `下降起源 (DescendOrigin)`
- `报告错误 (ReportError)`
- `存款资产 (DepositAsset)`
- `存款准备金资产 (DepositReserveAsset)`
- `兑换资产 (ExchangeAsset)`
- `启动储备金提取 (InitiateReserveWithdraw)`
- `启动远程传输 (InitiateTeleport)`
- `查询持有量 (QueryHolding)`
- `报告持有 (ReportHolding)`
- `买入执行 (BuyExecution)`
- `退款盈余 (RefundSurplus)`
- `设置错误处理程序 (SetErrorHandler)`
- `设置附录 (SetAppendix)`
- `清除错误 (ClearError)`
- `索取资产 (ClaimAsset)`
- `陷阱 (Trap)`
- `订阅版本 (SubscribeVersion)`
- `取消订阅的版本 (UnsubscribeVersion)`


### 关于术语的说明
- 当使用术语 起源 (Origin) 时，它的意思是 "其值为起源寄存器的位置"。因此，"由起源控制 "这一短语等同于 "由其值为起源寄存器的位置控制"。
- 当术语*持有 (Holding)*被使用时，它的意思是 "持有寄存器的值"。因此，"通过资产减少持有量" 这句话相当于 "通过资产减少持有寄存器的值"。
- 类似的还有附录 (Appendix)（附录寄存器的值）和错误处理 (Error Handler)（错误处理寄存器的值）。
- 术语on-chain应该被理解为 "在本地共识系统的持久状态内"，而不应该被认为是将当前的共识系统限制在一个专门的区块链系统。
- 类型 Compact、Weight 和 QueryId 应该是指 SCALE 定义中的紧凑整数。


### `提取资产 (WithdrawAsset)`
移除链上资产 (`assets`) 并将其计入持有。

操作数：

assets: `MultiAsset`s: 要移除的资产；必须由 Origin 拥有。
类型：Instruction。

错误：Fallible。

### `保留的储备资产 (ReserveAssetDeposited)`
累积到持有的衍生品资产中，以代表 Origin 上的资产（asset）。

操作数：

- `assets: MultiAssets`: 在 Origin 的本地共识系统的主权账户中收到的资产。
类型：Trusted Indication。

信任：必须信任 Origin 作为 assets 的储备。

错误：Fallible。

### `接收远程传输的资产 (ReceiveTeleportedAsset)`
将资产累积到相当于 Origin 上给定资产（assets）的 Holding。

操作数：

assets: `MultiAsset`s: 已从 Origin 移除的资产。
类型：Trusted Indication。

信任：必须相信 Origin 已经删除了 assets，作为发送此消息的结果。

错误：Fallible。

### `查询响应 (QueryResponse)`
提供来自 Origin 的预期信息。

操作数：

- `query_id: QueryId`: 导致此信息被发送的查询的标识符。
- `response: Response`: 信息内容。
- `max_weight: Weight`: 处理这个响应应该采取的最大权重。如果正确的执行需要更多的权重，那么将抛出一个错误。如果需要较少的权重，那么过剩权重寄存器可能会增加。
类型：Information。

错误：Fallible。

权重：权重估计可能利用max_weight，这可能导致在运行时增加过剩权重寄存器。

### `响应 (Response)`
Response 类型用于表达 QueryResponse XCM 指令中的消息内容。它可以代表几种不同的数据类型之一，因此它被编码为 SCALE 编码的标签联合体。

- `Null = 0`: 没有信息。
- `Assets { assets: MultiAssets } = 1`: 一些资产。
- `ExecutionResult { result: Result<(), (u32, Error)> } = 2`: 一个错误（或不是），相当于错误寄存器中包含的值的类型。
- `Version { version: Compact } = 3`: 一个 XCM 版本。

### `转移资产 (TransferAsset)`
从 Origin 的所有权中提取资产（assets），并在受益人 (beneficiary) 的所有权中存入同等的资产。

操作数：

- `assets: MultiAssetFilter`: 要提取的资产。
- `beneficiary`: 资产的新主人。
类型：Instruction。

错误：Fallible。

### `转移保留资产 (TransferReserveAsset)`
从 Origin 的所有权中提取资产 (assets)，并将同等的资产存入destination的所有权中 (即在其主权账户中)。

向 ReserveAssetDeposited 的 destination 发送一个成功 XCM 信息，带有指定的xcm.

操作数：

- `assets: MultiAsset`: 要提取的资产。
- `destination: MultiLocation`: 主权账户将拥有资产的位置，因此是资产的有效受益人，也是储备资产存款信息的通知对象。
- `xcm: Xcm`: 跟随ReserveAssetDeposited的指令，该指令将继续发送到destination。
类型：Instruction。

错误：Fallible。

### `交易 (Transact)`
调度编码后的 functor call，其调度源应是 Origin，由 origin_type表示。

操作数：

- `origin_type: OriginKind`: 将消息起源表达为调度起源的方法。
- `max_weight: Weight`: 重量"。调度 "call "时要消耗的最大权重。如果调度需要更多的重量，那么将抛出一个错误。如果调度需要较少的重量，那么 Surplus Weight Register 可能会增加。
- `call: Vec<u8>`: Vec<u8>: 要应用的编码交易。
类型：Instruction.

错误：Fallible.

权重：权重估计可能利用max_weight，这可能导致运行时剩余权重寄存器的增加。

### `HrmpNewChannelOpenRequest`
通知一个新的 HRMP 通道的消息。这个消息是由中继链发送到一个平行链。

操作数：

- `sender: Compact`: 即将打开的信道中的发送者。同时，也是信道开启的发起者。
- `max_message_size: Compact`: Compact: 发送方提议的信息的最大尺寸。
- `max_capacity: Compact`: 信道中可以排队的最大信息数量。
安全：消息应直接来自中继链。

类型：System Notification

### `HrmpChannelAccepted`
一个通知先前发送的开放通道请求已被接收方接收的消息。这意味着该通道将在下一次中继链的会话变更中被打开。这条消息是由中继链发送给一个平行链的。

操作数：

- `recipient: Compact`:  接受了上一次开放通道请求的接收方平行链。
安全：消息应该直接来自中继链。

类型：System Notification。

错误：Fallible.

### `HrmpChannelClosing`
一条消息，通知开放通道中的另一方来决定关闭它。特别是，initiator将关闭从sender到recipient的通道。关闭将在下一次中继链会话改变时进行。这条消息是由中继链发送至一个平行链。

操作数：

- `initiator: Compact`: 启动此关闭操作的平行链索引。
- `sender: Compact`: 被关闭的信道的发送方的平行链索引。
- `recipient: Compact`: 被关闭的通道的接收方的索引。
安全：信息应该直接来自中继链。

类型：System Notification。

错误：Fallible。

### `清除起源 (ClearOrigin)`
清除起源寄存器。

XCM 作者可以利用这一点来确保以后的指令不能命令原始起源的权限（例如，如果它们是从一个不受信任的来源转发的，如 ReserveAssetDeposited经常发生的情况）。

类型：Instruction。

错误：Infallible。

### `下降起源 (DescendOrigin)`
将起源改变到某个内部位置。

操作数：

`interior: Interior`MultiLocation`": 从 Origin 的上下文解释得到的位置，要放在 Origin 寄存器中。
类型：Instruction。

错误：Fallible。

### `报告错误 (ReportError)`
立即通过 XCM 向指定的目的地报告错误寄存器的内容。

一个 ExecutionOutcome 类型的 QueryResponse 消息被发送到 destination，其中包括给定的
query_id和 XCM 的结果。

操作数：

- `query_id: QueryId`: 用于 QueryResponse 消息的 query_id 字段的值。
- `destination: MultiLocation`: 发送 QueryResponse 信息的位置。
- `max_response_weight: Weight`: 用于 QueryResponse 消息的 `max_weight` 字段的值。
类型：Instruction

错误：Fallible。

### `存款资产 (DepositAsset)`
从持有中减去资产（assets），并在受益人 (beneficiary) 的所有权下存入链上的同等资产。

操作数：

- `assets: MultiAssetFilter`: 要从持有寄存器中删除的资产。
- `max_assets: Compact`: 要从持有登记册中删除的最大数量的独特资产/资产实例。只有第一个max_assets的资产/实例与assets匹配的才会被移除，按照标准的资产排序优先处理。其他的将保留在持有中。
- `beneficiary: MultiLocation`: 资产的新主人。
类型：Instruction。

错误：Fallible。

### `存款准备金资产 (DepositReserveAsset)`
从持有寄存器中删除资产 (assets)，并将链上的同等资产存入destination的所有权中 (即存入其主权账户)。

向destination发送一个 ReserveAssetDeposited的 XCM 消息，并给出 effects。

操作数：

- `assets: MultiAssetFilter`: 要从持有寄存器中删除的资产。
- `max_assets: Compact`: 要从持有寄存器中删除的最大数量的独特资产/资产实例。只有第一个max_assets的资产/实例与assets匹配，才会被移除，按照标准的资产排序优先处理。其他的将保持在持有状态。
- `destination: MultiLocation`: 主权账户拥有资产的地址，因此是资产的有效受益人，也是储备资产存放信息的通知目标。
- `xcm: Xcm`: 应该跟随 ReserveAssetDeposited 指令的订单，该指令将继续发送到destination。
类型：Instruction

错误：Fallible。



### `兑换资产 (ExchangeAsset)`
以一定数量的资产减少持有量（give），并以一定数量的替代资产累积持有量（receive）。

操作数：

- `give: MultiAssetFilter`: 应减少持有量的资产。
- `receive: MultiAssets`: 必须增加持有的资产。在receive中出现的任何可替换的资产可以增加，但 Holding 不能增加receive中没有说明的资产。
类型：Instruction

错误：Fallible。



### `启动储备金提取 (InitiateReserveWithdraw)`
用资产 (assets) 减少持有寄存器的值，并发送以WithdrawAsset开头的 XCM 消息到一个储备位置。

操作数：

- `assets: MultiAssetFilter`: 要从持有寄存器中删除的资产。
- `reserve`: 一个有效的位置，作为assets中所有资产的储备。这个共识系统的主权账户在储备位置将有适当的资产被提取，并且effects将被执行。在任何给定的资产/链组合上，通常只有一个有效的位置。
xcm: 一旦被提取，在储备位置的资产上执行的指令。
类型：Instruction

错误：Fallible。



### `启动远程传输 (InitiateTeleport)`
将资产（assets）从持有寄存器中移除，并向一个destination位置发送以ReceiveTeleportedAsset开头的 XCM 消息。

注意：destination 位置必须尊重这个起源，作为所有assets的有效远程传输起源。如果它不这样做，那么资产可能会丢失。

操作符：

- `assets: MultiAssetFilter`:  要从持有寄存器中删除的资产。
- `destination: MultiLocation`: 一个有效的位置，尊重来自这个位置的传输。
- `xcm`: 在ReceiveTeleportedAsset指令之后，在目标位置上执行的指令。
类型：Instruction

错误：Fallible。

### `报告持有 (ReportHolding)`
发送一个QueryResponse XCM 消息，其中assets值等于持有的内容，或其中的一部分。

操作数：

- `query_id: QueryId`: 用于QueryResponse信息的query_id字段的值。
- `destination: MultiLocation`: 发送QueryResponse信息的地点。
- `assets: MultiAssetFilter`: 应报告的资产的过滤器。
- `max_response_weight: Weight`: 用于 QueryResponse 消息的 max_weight 字段的值。
类型：Instruction

错误：Fallible。

### `买入执行 (BuyExecution)`
支付当前来自 Holding 的消息的执行费用。

操作数：

- `fees: MultiAsset`: 用于减少 Holding 支付执行费用的资产。
- `weight_limit: Option<Weight>`: 如果提供，那么说明要购买的权重。如果这低于该信息的估计权重，则会出现错误。
类型：Instruction

错误：Fallible。

### `退款盈余 (RefundSurplus)`
将退还的权重寄存器增加到剩余权重寄存器的值。试图将之前通过BuyExecution支付的费用计入退款权重寄存器增加的金额。

类型：Instruction

错误：Infallible。

### `设置错误处理程序 (SetErrorHandler)`
设置错误处理程序寄存器。

操作数：

- `error_handler: Xcm`: 设置错误处理程序寄存器的值。
类型：Instruction

错误：Infallible。

权重：该指令的估计权重必须包括error_handler的估计权重。在运行时，剩余权重寄存器应该在改变之前增加错误处理程序的估计权重。

### `设置附录 (SetAppendix)`
设置附录寄存器。

操作数：

- `appendix: Xcm`: 要设置附录寄存器的值。
类型：Instruction

错误：Infallible。

权重：该指令的估计权重必须包括appendix的估计权重。在运行时，剩余权重寄存器在被改变之前应增加附录的估计权重。

### `清除错误 (ClearError)`
清除错误寄存器。

类型：Instruction

错误：Infallible。

### `索取资产 (ClaimAsset)`
创建一些代表 Origin 持有的资产。

操作数：

- `assets: MultiAssets`: 要索取的资产。这必须与 Origin 可以用给定的票据 (ticket) 索取的资产完全匹配。
- `ticket: MultiLocation`: 资产的票据，这是一个抽象的标识符，帮助定位资产。
类型：Instruction

错误：Fallible。

### `陷阱 (Trap)`### `
总是抛出一个Trap类型的错误。

操作数：

id: Compact: 抛出的错误的参数值。
类型：Instruction

错误：Always。

### `订阅版本 (SubscribeVersion)`
向 Origin 发送 `QueryResponse` 消息，在 `response` 字段中指定 XCM 版本 2。

对本地共识的任何升级，导致支持 XCM 的更高版本，应引起类似的响应。

操作符：

- `query_id: QueryId`: 用于`QueryResponse`消息的`query_id`字段的值。
- `max_response_weight: Weight`: 用于 `QueryResponse`消息的`max_weight`字段的值。
类型：Instruction

### `取消订阅的版本 (UnsubscribeVersion)`
取消之前来自 Origin 的SubscribeVersion指令的效果。

类型：Instruction

### `销毁资产 (BurnAsset(MultiAssets))`
减少持有量，最多不多于给定的资产。

尽可能地减少持有量，直到参数中给定的资产。

操作数：

- `assets: MultiAssets`: 用于减少持有量的资产。
类型：Instruction

错误：Fallible。

### `预计资产 (ExpectAsset(MultiAssets))`  
如果 Holding 不包含至少给定的资产，则抛出一个错误。

操作数：

- `assets: MultiAssets`: 预计在持有中的最小资产。
类型：Instruction

错误：

- `ExpectationFalse`: 如果 Holding 不包含参数中的资产。

### `ExpectOrigin(MultiLocation)`
确保起源寄存器等于某个给定值，如果不等于，则抛出一个错误。

操作数：

origin: `MultiLocation`: 起源寄存器的预期值。
类型：Instruction

错误：

- `ExpectationFalse`: 如果 Origin 不是某个值，或者该值不等于参数。

### `ExpectError(Option<(u32, Error)>)`
确保错误寄存器等于某个给定值，如果不等于，则抛出一个错误。

操作数：

error: Option<(u32, Error)>: 错误寄存器的预期值。
类型：Instruction

错误：

- `ExpectationFalse`: 如果错误寄存器的值不等于该参数。

## **6** 通用资产标识符
关于版本的说明：这描述了在本文档中的 XCM 版本中使用的 `MultiAsset`（和关联），它的版本是由它所使用的 XCM 严格暗示的。如果有必要在 XCM 之外使用`MultiAsset`值（其版本无法推断），则应使用有版本意识的Versioned`MultiAsset`，这与Xcm和`VersionedXcm`的关系完全类似。

### 描述
`MultiAsset`是一个资产的一般标识符。它可以代表可替换和不可替换的资产，在可替换资产的情况下，它代表资产的一些规定数量。

由于一个`MultiAsset`的值只能用来代表一个单一的资产，所以有一个`MultiAsset`s类型，代表一组不同的资产。有时需要在资产范围内表达一个模式；为此有Wild`MultiAsset`，它允许"通配符"匹配。最后，经常需要提供一个"选择器"，它可能是一个一般的模式或一组具体的资产，为此，有`MultiAsset`Filter。

可选资产由一个class和一个资产的数量来识别，即资产价值所代表的资产类别的单位数。不可伪造的资产必须是唯一的，所以不需要amount，但这被一个资产实例的标识符所取代，允许在同一整体类别中的多个唯一资产。

资产类可以用两种方式之一来识别：抽象标识符或具体标识符。一个资产可以从多个资产标识符中引用，但往往只有一个规范的具体标识符。

#### 抽象标识符
抽象标识符是绝对的标识符，它代表了一个可以存在于多个共识系统中的名义资产。这类标识符处理起来比较简单，因为无论在哪个共识系统中解释，其广泛的含义都是不变的。

然而，在试图提供跨共识系统的统一性时，它们可能会将一些名义资产的不同实例（例如，储备资产和一个本地的储备衍生品）混在同一个名称下，导致混淆。这也意味着一种名义资产只以一种方式在本地进行核算。情况可能并非如此，例如，有多个桥实例，每个实例提供一个桥接的 "BTC" 代币，但没有一个是可以与其他代币互换的。

由于它们是绝对的和通用的，所以需要一个全局注册表来确保名称的冲突不会发生。

一个抽象的标识符被表示为一个简单的可变大小的字节字符串。截至目前，还没有全球性的注册机构，也没有提出任何关于资产标签的建议。

#### 具体标识符
具体的标识符是相对标识符，通过其在共识系统中相对于上下文解释的位置来具体识别单一资产。使用`MultiLocation`可以确保同一基础资产的类似但不可伪造的变体可以被适当区分，并且不需要任何类型的中央登记处。

限制是，资产标识符不能在共识系统之间简单复制，而是必须使用两个系统的相对路径，在移动到一个新的共识系统时"重新锚定" (re-anchored)。这点特别是因为`MultiLocation`值从根本上说是相对标识符。

在整个 XCM 中，信息的编写是当从接收者的角度解释时它们将具有预期的意义/效果。这意味着相对路径的构造应该总是从接收系统的角度来阅读，在编写系统中可能有完全不同的含义。

具体标识符是识别资产的首选方式，因为它们是完全不歧义的。

一个具体的标识符由一个`MultiLocation`表示。如果一个系统有一个明确的主要资产（如比特币的 BTC 或以太坊的 ETH），那么它将习惯性地被识别为链本身。其他更具体的指代系统内资产的方法包括：

- `<chain>/PalletInstance(<id>)`用于框架链的单一资产 pallet 实例（如 Balances pallet 的实例）。
- `<chain>/PalletInstance(<id>)/GeneralIndex(<index>)`用于框架链的多资产 pallet 实例的索引（例如 Assets pallet 的实例）。
- `<chain>/AccountId32`用于基于框架合约链上的 ERC-20 风格的单一资产智能合约。
- `<chain>/AccountKey20`用于类 Ethereum 链上的 ERC-20 风格的单一资产智能合约。


### 格式
一个 `MultiAsset` 值由 SCALE 编码的一对字段表示。

- `class: AssetId`: 资产类别。
- `fun`: 资产的可替代性，这是一个有标签的联合体，有两种可能的变体。
- `Fungible = 0 { amount: Compact }`: 在这种情况下，这是一个可替代资产，amount应该跟在 0 字节后面，以识别这个变体。
- `NonFungible = 1 { instance: AssetInstance }`: 在这种情况下，这是一个不可替代资产，instance应该跟在 1 字节后面，以识别这个变体。


如果需要在一个值中表达多个类，那么应该使用`MultiAsset`s类型。这与Vec<`MultiAsset`>的编码完全一样，但有一些额外的要求。

- 所有可替代资产必须放在编码中的非可替代资产之前。
- 可替代资产必须在标准排序中按类别排序（见下文）。
- 非可替代资产必须按类别排序，然后按标准排序中的实例排序。
- 每个可替代的资产类别只能有一个可替代的资产。
- 每一个非可替代的资产类别/实例对只能有一个非可替代的资产。
(该排序提供了一种有效的方法来保证每个资产在集合中确实是唯一的）。

一个 `WildMultiAsset` 的值由 SCALE 编码的标签联合体表示，有两个变体。

- `All = 0`: 匹配所有资产。
- `AllOf = 1 { class: AssetId, fun: WildFungibility }`: 匹配任何与给定的 class和可替代性（fun）匹配的资产。
- `一个 MultiAssetFilter` 的值由 SCALE 编码的标签联合体表示，有两个变体。

- `Definite = 0 { assets: MultiAssets }`。对assets中的所有资产进行过滤。
- `Wild = 1 { wildcard: WildMultiAsset }`: 对所有符合wildcard的资产进行过滤。


#### 标准排序
XCM 标准排序是基于 Rust 语言的排序，并进行了定义：

对于 SCALE 标记的联合体，值主要按变体索引（升序）排序，然后按变体项目的第一个字段排序，如果前面所有字段都是等价的，则按后续的字段排序。
对于 SCALE 元组（命名的或匿名的），值主要按第一个元素/字段排序，然后在所有先前的元素/字段相等的情况下，按每个后续的元素/字段排序。
对于 SCALE 向量 (`Vec<...>`)，数值是以标准的词法排序的 (主要是按第一项排序，然后在共享前缀的情况下按每个后续的项目排序，一个项目是另一个项目的严格前缀出现会排在前面)。


#### AssetId
一个资产类别的一般标识符。这是一个 SCALE 编码的标签联合体（Rust 术语中的 enum），有两个变体。

- `Concrete = 0 { location: MultiLocation }`: 一个具体的资产类别标识符，由一个`MultiLocation`值给出。
- `Abstract = 1 { name: Vec<u8> }`: 一个抽象的资产类标识符，由一个 `Vec<u8>` 值给出。


#### AssetInstance
在其类别中，一个非可替代资产实例的一般标识符。

由 SCALE 标签的联合体给出：

- `Undefined = 0`。未定义 -- 如果 NFA 类只有一个实例时使用。
- `Index = 1: { index: Compact }`: 一个紧凑的 index。技术上可以大于 u128，但是这个实现只支持到2**128-1的值。
- `Array4 = 2: { datum: [u8; 4] }`: 一个 4 字节的固定长度的 datum。
- `Array8 = 3: { datum: [u8; 8] }`: 一个 8 字节的固定长度的 datum。
- `Array16 = 4: { datum: [u8; 16] }`: 一个 16 字节的固定长度的 datum"。
- `Array32 = 5: { datum: [u8; 32] }`: 一个 32 字节的固定长度的 datum。
- `Blob = 6: { data: Vec<u8> }`: 一个任意精度的 data。仅在必要时使用。

#### WildFungibility
一个资产类别的一般标识符。这是一个 SCALE 编码的标签联合体（Rust 术语中的 enum），有两个变体：

- `Fungible = 0`: 匹配所有可替代资产。
- `NonFungible = 1`: 匹配所有非可替代资产。

## **7** 普遍共识的位置标识符
这描述了本文档的 XCM 版本中使用的 `MultiLocation`（及相关内容），它的版本是由它所使用的 XCM 严格暗示的。如果需要在 XCM 之外使用`MultiLocation`值（无法推断其版本），则应使用有版本意识的Versioned`MultiLocation`，完全类似于Xcm与`VersionedXcm`的关系。

### **7.1** 描述
含有状态的共识系统之间的相对路径。

`MultiLocation`的目标是在意义上足够抽象，性质上足够普遍，它能够在共识的宇宙中确定任意的逻辑 "位置"。

共识系统中的位置被定义为全球共识中的可隔离的状态机。有关的位置不需要有自己复杂的共识算法；例如，以太坊中的一个单一账户就可以被视为一个位置。

一个非常不完整的位置类型的列表包括：

- 一个（普通的，layer-1）区块链，例如，比特币主网或一个平行链。
- 一个 layer-0 的超级链，例如，Polkadot 中继链。
- 一个 layer-2 的智能合约，例如，以太坊的 ERC-20。
- 一条链的逻辑功能组件，例如，在 Frame-base 的 Substrate 链上的一个 pallet 单一实例。
- 一个账户。
`MultiLocation`是一个相对标识符，意味着它只能用于定义两个位置之间的相对路径，一般不能用来普遍地指代一个位置。就像一个相对的文件系统路径首先会以任何 ../ 组件开始，用于上升到包含的目录，然后是下降到的目录名称，一个`MultiLocation`有两个主要部分：从本地上升到外部共识的次数，和外部共识中的内部位置。

因此，一个 `MultiLocation` 被编码为一对值。

parents：u8: 在解释 interior 参数之前要上升的共识层级。
interior: Interior`MultiLocation`: 通过提升本地系统parents次数来找到外部共识系统的内部位置。


### 内部位置和交叉点
有第二种类型Interior`MultiLocation`，它总是识别一个共识系统内部的本地共识系统。严格意义上的内部意味着一种从属关系：一个共识系统 A 是 B 的内部，意味着 A 的状态变化会导致 B 的状态变化。作为一个例子，以太坊区块链中的智能合约位置被认为是以太坊区块链本身的内部位置。

一个 Interior`MultiLocation` 是由一些交叉点组成的，按顺序排列，每个交叉点都指定了一个比前一个更内部的位置。一个没有交叉点的Interior`MultiLocation`值只是指本地共识系统。

因此一个 Interior`MultiLocation` 被简单地编码为一个 Vec<Junction>。同时，一个Junction被编码为以下的标记联合体。

- `Parachain = 0` { index: Compact<u32> }: 一个有索引的平行链，属于并由上下文操作。通常在上下文是一个 Polkadot 中继链时使用。
- `AccountId32 = 1` { network: NetworkId, id: [u8; 32] }: 一个 32 字节的标识符（id），用于特定network的账户，在上下文中被视为一个主权终端。通常在上下文是一条基于 Substrate 的链时使用。
- `AccountIndex64 = 2` { network: NetworkId, index: Compact<u64> }: 一个 8 字节的 index，用于特定network的账户，在上下文中被视为一个主权终端。当上下文是一个Frame-based的链，包括例如一个索引 pallet 时，可以使用。network ID 可用于限制或限定索引的含义，可能不具体指代区块链网络。一个例子可能是一个智能合约链，它使用不同的network值来区分账户索引和智能合约索引。
- `AccountKey20 = 3` { network: NetworkId, key: [u8; 20] }: 一个 20 字节的标识符key，用于特定network的账户，在上下文中被尊重为主权终端。可以在上下文是以太坊或比特币链或智能合约时使用。
- `PalletInstance = 4` { index: u8 }: 一个实例，index的 pallet，构成了上下文的一个组成部分。一般用于Frame-based的链。
- `GeneralIndex = 5` { index: Compact }: 在上下文位置中的一个非描述性的 index。由于它的通用性，用法会有很大不同。注意：尽量避免使用这个，而是使用一个更具体的项目。
- `GeneralKey = 6` { key: Vec<u8> }: 一个非描述性的数据，作为上下文位置中的 key。由于它的通用性，用法会有很大不同。注意：尽量避免使用这个，而是使用一个更具体的项目。
- `OnlyChild = 7`: 不歧义的子项。


#### 网络 ID
一个有账户的共识系统的全局标识符。

编码为以下内容的标记联合体：

- `Any = 0`。未识别的/任何。
- `Named = 1` { name: Vec<u8> }: 一些命名 (name) 的网络。
- `Polkadot = 2`: Polkadot 中继链
- `Kusama = 3`: Kusama.


### 书面格式
注意：`MultiLocation`将倾向于使用以斜线为界的交叉点名称来书写，让人想起其他逻辑路径系统，如 URI 和文件系统的语法。例如，一个 `MultiLocation`值表示为../PalletInstance(3)/GeneralIndex(42)，就是一个有一个父级和两个Junction的 `MultiLocation`。PalletInstance{index: 3}和 GeneralIndex{index: 42}.

## **8** XCM 中的错误类型
在 XCM 中，有必要交流在执行某些信息时遇到的一些问题。Error类型允许表达这种情况，并被编码为 SCALE 标签的联合体：

- `Overflow = 0`:一个算术溢出发生了。
- `Unimplemented = 1`: 该指令没有实现的。
- `UntrustedReserveLocation = 2`: Origin Register 不包含资产转移通知的值。
- `UntrustedTeleportLocation = 3`: Origin Register 不包含资产传送通知的值。
- `MultiLocationFull = 4`: `MultiLocation`值过大，无法进一步下沉到子系统。
- `MultiLocationNotInvertible = 5`: `MultiLocation`值上升的层级位置超过了本地可以查询到的位置。
- `BadOrigin = 6`: Origin Register 不包含指令的有效值。
- `InvalidLocation = 7`: 位置参数不是指令的有效值。
- `AssetNotFound = 8`: 没有处理给定的资产。
- `FailedToTransactAsset = 9`：资产交易失败。一个资产交易（如提款或存款）失败（通常是由于类型转换）。
- `NotWithdable = 10`: 一个资产不能被提取，可能是由于缺乏所有权、可用性或权利。
- `LocationCannotHold = 11`: 资产不能存放在某个特定地点的所有权下。
- `ExceedsMaxMessageSize = 12`:试图发送一个超过传输协议支持的最大的信息。
- `DestinationUnsupported = 13`: 给定的信息不能被翻译成目的地支持的格式。
- `Transport = 14`: 目的地是可路由的，但传输机制有一些问题。
- `Unroutable = 15`: 目的地被认为是不可路由的。
- `UnknownClaim = 16`: 由`ClaimAsset`使用，当给定的索赔不能被识别/找到。
- `FailedToDecode = 17`: 当漏斗不能被解码时，由`Transact`使用。
- `TooMuchWeightRequired = 18`: 由`Transact`使用，表示该函数器可能突破了给定的`Weight`限制。
- `NotHoldingFees = 19`: 由`BuyExecution`使用，当持有资产不足以支付费用。
- `TooExpensive = 20`: 当声明购买重量的费用不足时，由`BuyExecution`使用。
- `Trap(u64) = 21`: 由Trap指令使用，故意强制出错。它的代码包括在内。
- `ExpectationFalse = 22`: 由`ExpectAsset`, `ExpectError`和`ExpectOrigin`使用，当期望值不真实时。