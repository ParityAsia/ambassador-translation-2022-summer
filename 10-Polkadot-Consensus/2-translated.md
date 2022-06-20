---
title: 'Substrate 中共识'
src: https://zhuanlan.zhihu.com/p/508849408
author: Akagi201 (https://github.com/Akagi201)
snapshot-date: 2022-06-18
---

# Substrate 中共识

区块链节点使用共识引擎来在区块链状态上达成一致。本文介绍了区块链系统中共识的基本原理，共识如何与 Substrate 框架中的运行时互动，以及该框架可用的共识引擎。

## 状态机和冲突
区块链运行时是一个[状态机](https://en.wikipedia.org/wiki/Finite-state_machine)。它有一些内部状态，以及允许它从当前状态转换到未来状态的状态转换函数。在大多数运行时中，有些状态有多个有效的转换到多个未来状态，但必须选择一个转换。

区块链必须达成一致：

一些初始状态，称为 "genesis"。
一系列的状态转换，每个称为 "区块"，和
一个最终（当前）状态。
为了就转换后的状态达成一致，区块链的[状态转换功能](https://docs.substrate.io/v3/concepts/runtime/)中的所有操作都必须是确定性的。

## 排除冲突
在中心化系统中，中央机构通过按照看到的顺序记录状态转换，在相互冲突的备选方案中进行选择，并在出现冲突时选择竞争方案中的第一个。在去中心化系统中，节点会以不同的顺序看到交易，因此它们必须使用一种更复杂的方法来排除冲突交易。更进一步的复杂性，区块链网络要努力做到容错，这意味着即使一些参与者不遵守规则，它们也应该继续提供一致的数据。

区块链将交易批量化为区块，并有一些方法来选择哪个参与者有权提交一个区块。例如，在工作量证明链中，首先找到有效工作证明的节点有权向链上提交一个区块。

Substrate 提供了几种区块构建算法，也允许你创建自己的算法。

- Aura（round robin）
- BABE (基于槽的)
- 工作量证明

## 分叉选择规则
作为一个基本要素，一个区块包含一个头和一批[外部交易](https://docs.substrate.io/v3/concepts/extrinsics/)。头必须包含对其父块的引用，这样就可以追溯到其起源的链。当两个块引用同一个父块时就会出现分叉。分叉必须被解决，以便只存在一个规范的链。

分叉选择规则是一种算法，它采用区块链并选择 "最佳" 链，因此是应该被扩展的。Substrate 通过 [SelectChain Trait](https://paritytech.github.io/substrate/master/sp_consensus/trait.SelectChain.html) 暴露了这个概念。

Substrate 允许你编写一个自定义的分叉选择规则，或者使用一个开箱即用的规则。比如：

### 最长链规则
最长链规则简单地说，最好的链就是最长的链。Substrate 通过 [LongestChain 结构](https://paritytech.github.io/substrate/master/sc_consensus/struct.LongestChain.html)提供了这种链选择规则。GRANDPA 使用最长链规则进行投票。

![longest chain](assets/longest.png)

### GHOST(Greedy Heaviest Observed SubTree) 规则
贪婪的最重观察子树规则说，从创世块开始，每个分叉都是通过选择在其上构建了最多块的分支来解决。

[!GHOST](assets/ghost.png)

## 区块生成
区块链网络中的一些节点能够产生新的区块，这个过程被称为创作 (authoring)。究竟哪些节点可以生成区块，取决于你使用的是哪种共识引擎。在一个中心化的网络中，一个节点可能会编写所有的区块，而在一个完全无权限的网络中，一个算法必须在每个区块高度选择区块作者。

### 工作量证明
在像比特币这样的工作证明系统中，任何节点都可以在任何时候生产一个区块，只要它已经解决了一个计算密集型问题。解决该问题需要 CPU 时间，因此矿工只能按照其计算资源的比例生产区块。Substrate 提供了一个工作量证明的区块生产引擎。

### 槽
基于槽的共识算法必须有一组已知的验证者，他们被允许产生区块。时间被划分为不连续的槽，在每个槽中，只有一些验证者可以产生一个区块。在每个槽内，哪些验证者可以生产区块的具体细节因引擎而异。Substrate 提供了 Aura 和 Babe，它们都是基于槽的区块创作引擎。

## 最终一致性
任何系统中的用户都想知道他们的交易何时最终完成，区块链也不例外。在一些传统的系统中，当收据被移交，或文件被签署时，就会出现最终结果。

使用到目前为止描述的区块生成方案和分叉选择规则，交易永远不会完全最终完成。总是有一个机会，一个更长（或更重）的链会出现，并回滚你的交易。然而，越多的区块被建立在一个特定的区块之上，它就越不可能被回滚。这样一来，区块生成和适当的分叉选择规则提供了概率上的最终性。

当需要确定的最终性时，可以在区块链的逻辑中添加一个最终性程序。一个固定的权威机构的成员对最终性进行投票，当某一区块有足够的票数时，该区块就被视为最终性。在大多数系统中，这个门槛是 2/3。如果没有外部协调，如硬分叉，被这样的程序最终确定的区块是不能被恢回滚的。

注意
一些共识系统将区块生成和最终性结合在一起，例如，最终性是区块生成过程的一部分，在区块 N 被最终确定之前，一个新的区块 N+1 不能被生成。然而，Substrate 将这两个过程分隔开来，并允许你使用任何区块生产引擎来实现概率最终性，或者将其与最终性程序结合起来以实现确定性最终性。
在使用最终性程序的系统中，分叉选择规则必须被修改以考虑最终性游戏的结果。例如，一个节点不是选择最长的链期，而是选择包含最近完成的区块的最长的链。

## Substrate 中的共识
Substrate 框架配备了几个共识引擎，它们提供了区块创作，或者说是最终性。本文简要介绍了 Substrate 本身所提供的产品。我们随时欢迎开发者提供他们自己的自定义共识算法。

### Aura
[Aura](https://paritytech.github.io/substrate/master/sc_consensus_aura/index.html) 提供了一个基于插槽的区块生成机制。在 Aura 中，一组已知的权威者轮流制作区块。

### BABE
[BABE](https://paritytech.github.io/substrate/master/sc_consensus_babe/index.html) 还提供了基于槽的区块生成，有一套已知的验证人。在这些方面，它与 Aura 相似。与 Aura 不同的是，槽的分配是基于可验证的随机函数（VRF）的评估。每个验证者都被分配了一个 epoch 的权重。这个 epoch 被分成若干个槽，验证人在每个槽评估其 VRF。对于验证人的 VRF 输出低于其权重的每个槽，它被允许编写一个区块。

由于多个验证人可能在同一个槽中产生一个区块，因此在 BABE 中，分叉比在 Aura 中更常见，甚至在良好的网络条件下也很常见。

Substrate 对 BABE 的实现也有一个回退机制，用于在给定的 slot 中没有验证人被选中时。这些 "次级" 槽位分配使 BABE 能够实现恒定的区块时间。

### 工作量证明共识
[工作量证明](https://paritytech.github.io/substrate/master/sc_consensus_pow/index.html)区块生成不是基于槽位的，不需要一个已知的权威集。在工作量证明中，任何人都可以在任何时候产生一个区块，只要他们能够解决一个具有计算难度的问题（通常是哈希预像搜索 (hash preimage search)）。这个问题的难度可以被调整，以提供一个统计上的目标区块时间。

## GRANDPA
[GRANDPA](https://paritytech.github.io/substrate/master/sc_finality_grandpa/index.html) 提供了区块的最终确定。它有一个像 BABE 一样的已知加权权威集。然而，GRANDPA 并不生成区块；它只是监听一些生成引擎产生的区块的 gossip 协议，如上面讨论的三个引擎。GRANDPA 验证者对链进行投票，而不是对区块进行投票，也就是说，他们对他们认为 "最好" 的区块进行投票，他们的投票将被转用于所有先前的区块。一旦超过 2/3 的 GRANDPA 验证者对某一特定区块进行了投票，该区块就被视为最终结果。

## 与运行时协作
最简单的静态共识算法完全在运行时之外工作，正如我们目前所描述的。然而，许多共识游戏通过增加需要与运行时协作的功能而变得更加强大。例子包括工作量证明中的可调整难度，权威证明中的权威轮换，以及权益证明网络中基于权益的加权。

为了适应这些共识功能，Substrate 有一个 [DigestItem](https://paritytech.github.io/substrate/master/sp_runtime/enum.DigestItem.html) 的概念，这是一个从节点的外部传递给运行时的消息，即共识所在的地方，反之亦然。

## 学习更多
因为 BABE 和 GRANDPA 都将在 Polkadot 网络中使用，所以 Web3 基金会提供算法的研究级介绍。

[BABE 研究](https://research.web3.foundation/en/latest/polkadot/block-production/Babe.html)
[GRANDPA 研究](https://research.web3.foundation/en/latest/polkadot/finality.html)
所有确定性的最终算法，包括 GRANDPA，都要求至少有 2f+1 个非故障节点，其中 f 是故障或恶意节点的数量。了解更多关于这个阈值的来源以及为什么它是理想的，请看开创性的论文在[有故障的情况下达成协议](https://lamport.azurewebsites.net/pubs/reaching.pdf)或在维基百科：[拜占庭故障](https://en.wikipedia.org/wiki/Byzantine_fault)。

并非所有的共识协议都定义了一个单一的、规范的链。一些协议验证了有向[无环图 DAG](https://en.wikipedia.org/wiki/Directed_acyclic_graph)，当两个具有相同父辈的块没有冲突的状态变化时。