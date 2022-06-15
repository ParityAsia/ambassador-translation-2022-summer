Astar | Substrate Builders Program 毕业生
王奡 SubDev Insight 2022-04-26 23:10 发表于陕西


图片



● 原文链接：https://www.parity.io/blog/sbp-graduates-astar

● 翻译：SEP_003 王奡



SBP（Substrate Builders Program）毕业生中的一些重点团队已经成功完成 Substrate Builders 计划并在 Polkadot 上启动平行链。这篇文章我们将介绍第三个在 Polkadot 上赢得平行链 Auction 的项目：Astar。



图片



Astar Network（前 Plasm Network）是基于波卡 (Polkadot) 生态系统的 DApp 中心，支持多虚拟机（ EVM 和 WebAssembly）和包括 Rollups 在内的 Layer2 解决方案。尤其对于智能合约来说，多虚拟机和可扩展性是下一代智能合约平台至关重要的属性。





▌Astar为波卡生态系统提供了什么

 　

Astar 开发了一个名为创新机制——dApps Staking 。作为一个 Substrate pallet，允许账户质押 token 到智能合约中，为每个区块设置质押奖励。



图片



在 Astar Runtime 里实现的另一个功能是——自定义签名调用。这个 pallet 允许账户在执行 Substrate 外部信息时使用 ECDSA 签名方案（例如：使用 Ledger 以太坊账户签署调用）。Substrate 的外部信息模块化，使得打包本地调用使用外部签名的方式，将实现可扩展性成为可能。



最后，Astar 还有一个关键组件是多虚拟机。合约模块和 EVM 模块作为 Substrate 生态系统一部分，Astar 允许项目提供多个合约执行环境。通过预编译合约调用，允许两个不同 VM 之间的互操作性，进一步提高了这种可用性。



图片



可扩展性是所有区块链面临的最大挑战。为了使区块链技术得到广泛采用，更高的性能是有必要的。可扩展性是 Astar 寻求优化的一个关键问题。





“Astar Network 是基于波卡的多链去中心化应用层。Astar 整合了EVM、WebAssembly 和 Layer2 解决方案。该平台支持各种去中心化应用，例如DeFi、NFT 和 DAO 等等。”
——Hoon Kim

Astar 产品经理




▌在Substrate Builders Program的旅程

 　

构建者获得了 Parity 一系列系统性的支持，包括在技术、社区和资金的支持，以及战略规划和反馈意见等全面的帮助和建议。Astar 是 Substrate Builders Program 的首批参与者之一，并成功完成了该计划设定的三个重大成就：



Astar 团队取得的一些重要亮点包括：



1. 针对支持者锁仓空投机制 (Lockdrop) 的构思、测试和实现。这种方法最早由 Edgeware 首次公开并推广，并被 Astar 团队成功用于引导社区参与众贷。

2. 作为 Substrate 的一部分，Astar 以太坊虚拟机的执行和在其早期生命周期中的持续升级。EVM Pallet 已经有了许多改进，Astar 在执行 EVM 方面的工作为 Substrate 生态系统做出了很大贡献。


3. Shiden 是 Astar 的金丝雀网络，在 Rococo 测试网进行了几个月的测试后，最后在 Kusama 网络上赢得了平行链拍卖。



4. 看好 WebAssembly 是智能合约的未来。Astar 团队一直在为 Solang 做出重大贡献，Solang 是一个用 Rust 编写的编译器，它允许将 Solidity 合约编译为 Wasm 二进制文件。




▌你可以用 Astar 做什么？

对于开发者来说，可以在 Polkadot 上使用 Astar 网络，在 Kusama 上使用 Shiden 网络。以下是使用 Shiden 和 Astar 构建的四种方式：



图片



在 Astar / Shiden EVM上开发和部署你的第一个智能合约

https://docs.astar.network/tutorial/develop-and-deploy-your-first-smart-contract-on-aster-shiden-evm

使用 Patract Labs 的 Redspot 部署你的智能合约

https://docs.astar.network/integration/using-redspot-by-patract-lab

通过教程了解如何部署 Wasm 智能合约

https://docs.astar.network/build/smart-contracts/wasm/solidity

开发者和用户可以学习如何从 Astar Dapp Staking 的功能中受益，这是以奖励帮助和支持 Astar 生态系统发展的用户和开发者。    



▌Builders For Builders：Astar 构建者计划 


作为 Builders for Builders 计划的一部分，从 Substrate Builders Program 毕业的 Astar ，提供了创建自己 Builders 计划的工具和知识。到目前为止，已有非常多的应用寻求 Astar 团队和社区的支持。

https://forum.astar.network/c/builders-program/14



该计划以各种方式提供支持：



项目 Grants：浏览 Astar 和 Shiden Network 的捐赠流程，并支持和帮助你的项目。

推广支持：介绍相关区块链生态系统的参与者，旨在联名发布公告和其他推广合作机会。

技术支持：获得来自其他项目成员和工程师的支持，允许成员们之间相互协作和提问。

与支持者的联系：介绍 Astar 和 Shiden 生态系统的支持者。




▌什么是Substrate Builders Program



Substrate Builders Program 支持 Substrate 生态系统中的构建者。该计划允许构建基于 Substrate 的区块链、应用程序或生态系统组件的团队，从 Parity 的丰富经验和资源中受益，能够帮助他们取得成功。



Substrate Builders Program 委员会评估每个项目的不同愿景、目标和需求，并帮助他们取得成功制定一系列计划。该计划分为三个重大事件，每个重大事件持续三个月。



Parity 的 Runtime 工程师团队会定期检查阶段任务完成质量，为团队实施解决方案提供指导和反馈。在成功完成所有重大事件（或独立上线或通过成为平行链）后，该团队将从 Substrate Builders Program 毕业。



—



如果你有关于 Substrate 或在 Substrate 上构建项目的想法，Substrate Builders 计划可以支持您的项目直到项目上线，并将根据您的项目需求继续提供长期支持。