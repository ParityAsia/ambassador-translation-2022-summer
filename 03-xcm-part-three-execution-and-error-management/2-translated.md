---
title: 'XCM Part III: 执行和错误管理'
src: https://github.com/Akagi201/rust-docs/blob/master/polkadot/polkadot-blog-xcm-p3.md
author: Akagi201 (https://github.com/Akagi201)
snapshot-date: 2022-06-17
---

# XCM Part III: 执行和错误管理

> 原文链接：<https://polkadot.network/blog/xcm-part-three-execution-and-error-management/>
>
> 翻译：[Akagi201](https://github.com/Akagi201)

![xcm_cover_p3](assets/xcm_cover_p3.jpeg)

在我写的关于 XCM 的前两篇文章（[第一部分](https://polkadot.network/blog/xcm-the-cross-consensus-message-format/)，[第二部分](https://polkadot.network/blog/xcm-part-two-versioning-and-compatibility/)）中，我介绍了其设计和版本结构的基本情况。在这篇文章中，我们将更深入地了解它的基本设计和执行模型。由于 XCM 是基于 XCVM 的指令集，这是一个非常高级的虚拟机，这相当于熟悉了这个机器架构。

XCVM 是一个非常高层次的、非图灵完全的虚拟机。它是基于寄存器的（而不是基于堆栈的），并有几个特殊用途的寄存器，其中大部分存放高度结构化的数据。与通用处理器不同，XCVM 的寄存器不能自由地设置为任意的值，而是有严格的机制来管理它们如何变化。除了与本地链状态进行交互的某些手段（如我们已经看到的 `WithdrawAsset` 和 `DepositAsset` 指令），没有额外的"内存"。没有循环的可能性，也没有明确的分支指令。

我们已经介绍了其中的两个寄存器：持有寄存器 (Holding Register)，它能够暂时持有一个或多个资产，可以通过从本地链中提取资产，或者通过从可信的外部来源（例如另一个链）接收资产来填充；以及原点寄存器 (Origin Register)，在执行之初，它持有当前 XCM 执行所来自的共识系统的位置，并且只能被突变到内部位置或完全清除掉。

在其他的寄存器中，有三个与异常/错误管理有关，两个与跟踪执行权重有关。我们将在本文中了解所有这些寄存器的情况。

## 🎬 执行模型

如前所述，没有明确的条件性指令或循环原语，使得同一指令可以重复执行一次以上。这使得预先确定一个程序的控制流变得相当简单。鉴于我们想确定一条 XCM 消息在执行前可以利用多少执行时间（在 Substrate/Polkadot 中被称为权重），这一特性非常有用。

我们期望执行 XCM 的大多数共识平台将需要在开始执行之前确定最坏情况下的执行时间。这是由于区块链通常需要确保单个区块的处理时间不超过某个预先确定的限制，以免导致整个系统停滞。此外，如果系统需要支付手续费，那么它必须在支付的工作量之前发生，而且这种支付必须涵盖最坏情况下的执行时间。

由于这种图灵完备性，允许图灵完备语言的系统（例如以太坊）实际上无法直接从程序中计算出最坏情况的执行时间。他们通过要求用户预先确定程序的执行资源来解决这个问题，然后在执行过程中对其进行计量，如果超过了所支付的金额，就中断它。有时，在交易执行之前事情就发生了变化，权重变得不正确。令人高兴的是，像 XCVM 这样的虚拟机不是图灵完备的，它可以避免这种计量和权重规定的需要。

## 🏋️‍♀️ 权重

权重通常表示为一个有代表性的硬件执行给定操作所需的整数皮秒 (picosecond)。正如我们在 `BuyExecution` 指令中所看到的，XCVM 在处理某些指令时包括了执行时间/权重的概念。

没有计量权重，但为了允许 XCVM 程序最终花费的时间少于最坏情况下的权重预测，我们有一个寄存器，称为剩余权重寄存器 (Surplus Weight Register)。大多数指令不接触它，因为我们可以准确预测它们将使用多少权重。然而，在某些情况下，最坏情况下的权重预测是高估的，只有在执行时我们才知道是多少。在对 XCM 消息的权重进行高估的同时，跟踪原始权重的高估量，并从账目中减去，使链上的区块执行时间的配额得到优化。

因此，剩余权重寄存器对我们的区块执行时间核算很有用，但并不能单独解决另一个问题，即确保支付的金额不是高估的。为此，我们需要一个与 `BuyExecution` 相配套的指令，该指令将任何剩余的权重予以退还。当然，这个指令是存在的，叫做 `RefundSurplus`。它还利用了第二个寄存器，称为 `Refunded Weight Register`，以确保同一剩余权重不会被多次退还。

## 😱 流动控制和异常情况

到目前为止，在我们对 XCVM 的处理中，还有两个寄存器比较隐蔽，但还是有必要了解一下。首先是程序寄存器 (Programme Register)，它存储当前执行的 XCVM 程序。其次，是程序计数器 (Programme Counter)，它存储当前执行的指令索引。当程序寄存器被改变时，它被重置为零，在每个成功执行的指令结束时，它被增加一。

处理"例外"情况的能力对于编写健壮的代码至关重要。当远程系统发生了你没有预料到的事情（或确实无法预料到的事情），那么你需要某种方式来管理它，即使只是向原系统发送一份报告，说明这一点。

虽然 XCVM 指令集不包括任何明确的通用分支指令，但它的执行模型中确实有一个通用的异常处理框架。XCVM 还包括两个代码寄存器，每个寄存器都保存着像程序寄存器一样的 XCVM 程序。这两个寄存器被称为附录寄存器 (Appendix Register) 和错误处理器寄存器 (Error Handler Register)。如果你熟悉几种流行语言中的 `try/catch/finally` 异常处理系统，那么接下来的内容可能会让人很有印象。

如前所述，XCVM 程序的执行是按照其中的每条指令一步步进行的。当它按照这些指令执行到程序结束时，会发生两种情况之一：要么成功到达程序的终点，要么发生错误。在第一种成功执行的情况下，错误寄存器被清零，其权重被添加到剩余权重寄存器中。附录寄存器也被清空，其内容被放入程序寄存器中。如果程序寄存器是空的，那么我们就停止执行。否则程序计数器被重置为零。简单地说，我们扔掉当前的程序和错误处理程序，开始执行附录程序（如果有）。

这个功能本身并不那么有用，但如果与错误情况下发生的情况结合起来，就会很有用。在这里，任何尚未执行的指令的权重被添加到剩余权重寄存器。错误处理寄存器被清除，其内容被放入程序寄存器，程序计数器被权重为零。简单地说，我们扔掉当前的程序，开始执行错误处理程序。因为我们没有清除附录寄存器，所以除非它被错误处理程序重置，否则一旦成功完成，它就会执行。

由于它的组成结构，它允许错误处理程序的任意"嵌套"：如果需要，错误处理程序也可以有错误处理程序，附录可以有自己的附录。

有两条指令允许对这些寄存器进行操作。`SetAppendix `和 `SetErrorHandler`。如你所料，其中一条设置附录寄存器，另一条设置错误处理程序寄存器。每一个的预测权重都比其参数的权重多出一小部分。然而，当执行时，将被替换的寄存器中的 XCM 信息的权重被添加到剩余权重寄存器中，使任何未使用的附录或错误处理程序的权重被回收。

## ☄️ 抛出错误

有时，确保一个错误发生并定制该错误的某些方面可能是有用的。这是在编写测试代码时使用的，但它最终可能会在实际运行的链中找到用途也不是不可能。这可以在 XCVM 中通过指令 `Trap` 来实现，它总是导致错误发生。被抛出的错误类型与 `Trap` 名称相同。指令和错误都带有一个整数参数，允许某种形式的信息在错误抛出者和外部观察者之间传递。

下面是一个简单的例子：

```rust
WithdrawAsset((Here, 10_000_000_000).into()),
BuyExecution {
    fees: (Here, 10_000_000_000).into(),
    weight: Unlimited,
},
SetErrorHandler(Xcm(vec![
    RefundSurplus,
    DepositAsset {
        assets: All.into(),
        max_assets: 1,
        beneficiary: Parachain(2000).into(),
    },
])),
Trap(0),
DepositAsset {
    assets: All.into(),
    max_assets: 1,
    beneficiary: Parachain(3000).into(),
},
```

`Trap` 导致最后的 `DepositAsset` 被跳过，取而代之的是错误处理程序的 `DepositAsset` 运行，将 1 DOT（减去执行成本）置于 parachain 2000 的所有权之下。我们总是倾向于在错误处理程序代码的开头使用 `RefundSurplus`，因为如果它正在运行，我们知道很可能所使用的预测权重（从而购买的权重）是一个高估的。

## 🗞 错误报告

能够引入代码来处理错误是非常有用的，但有一个经常被要求的功能是能够将 XCM 消息的结果反馈给原发件人。我们在上一篇文章中见到了 `QueryResponse` 指令，它允许一个共识系统向另一个共识系统报告一些信息，剩下的就是能够以某种方式将 XCM 的结果插入 `QueryResponse` 中，并将其发送给期望被告知结果的人。

事实证明，有一条指令可以做到这一点，名为 `ReportError`。它通过使用一个我们还没有接触过的寄存器：错误寄存器 (Error Register) 来工作。错误寄存器是一个可选的类型（它可以被设置或清除）。如果它被设置了，那么它持有两个信息：一个数字索引和一个 XCM 错误类型。

它的运行机制极其简单。首先，当一条指令产生错误时，它总是被设置；错误类型被设置为该错误的类型，数字索引被设置为程序计数器寄存器的值。其次，只有当 `ClearError` 指令被执行时，它才会被清除。这条指令是无懈可击的指令之一 -- 它本身是不允许导致错误发生的。仅此而已 -- 当错误发生时它被设置，当你发出适当的指令时被清除。

现在我们应该清楚地了解 `ReportError` 指令的工作原理：它只是用错误寄存器的内容组成一个 `QueryResponse` 指令，并将其发送到一个特定的目的地。当然，在它之前发生的任何错误都会导致指令被跳过，因为执行时首先跳到错误处理寄存器的代码，然后再跳到附录寄存器的代码。然而，解决这个问题的方法很简单：将 `ReportError` 放在附录中，将确保它被执行，无论主代码是否导致执行错误。

让我们看一下一个简单的例子。我们将把一笔资产（1 DOT）从中继链传送到 Statemint（parachain 1000），在那里购买一些执行时间，然后用 Statemint 作为储备，我们将资产存入 parachain 2000。原始的（不报错的）信息会是这样的：

```rust
WithdrawAsset((Here, 10_000_000_000).into()),
InitiateTeleport {
    assets: All.into(),
    dest: Parachain(1000).into(),
    xcm: Xcm(vec![
        BuyExecution {
            fees: (Parent, 10_000_000_000).into(),
            weight: Unlimited,
        },
        DepositReserveAsset {
            assets: All.into(),
            max_assets: 1,
            dest: ParentThen(Parachain(2000)).into(),
            xcm: Xcm(vec![
                BuyExecution {
                    fees: (Parent, 10_000_000_000).into(),
                    weight: Unlimited,
                },
                DepositAsset {
                    assets: All.into(),
                    max_assets: 1,
                    beneficiary: Parent.into(),
                },
            ]),
        },
    ]),
}
```

在基本的错误报告中，我们会使用以下方式来代替：

```rust
WithdrawAsset((Here, 10_000_000_000).into()),
InitiateTeleport {
    assets: All.into(),
    dest: Parachain(1000).into(),
    xcm: Xcm(vec![
        BuyExecution {
            fees: (Parent, 10_000_000_000).into(),
            weight: Unlimited,
        },
        SetAppendix(Xcm(vec![
            ReportError {
                query_id: 42,
                dest: Parent.into(),
                max_response_weight: 10_000_000,
            },
        ])),
        DepositReserveAsset {
            assets: All.into(),
            max_assets: 1,
            dest: ParentThen(Parachain(2000)).into(),
            xcm: Xcm(vec![
                BuyExecution {
                    fees: (Parent, 10_000_000_000).into(),
                    weight: Unlimited,
                },
                SetAppendix(Xcm(vec![
                    ReportError {
                        query_id: 42,
                        dest: Parent.into(),
                        max_response_weight: 10_000_000,
                    },
                ])),
                DepositAsset {
                    assets: All.into(),
                    max_assets: 1,
                    beneficiary: ParentThen(Parachain(2000)).into(),
                },
            ]),
        },
    ]),
}
```

正如你所看到的，唯一的变化是引入了两条 `SetAppendix` 指令，确保 Statemint 和 parachain 2000 中的错误或缺失都将报告给中继链。这假定中继链已经将自己设置为能够识别和处理来自 Statemint 和 parachain 2000 的查询 ID 为 42、权重限制为 1000 万的 `QueryResponse` 消息。令人高兴的是，这确实是 Substrate 所支持的东西，但现在已经超出了范围。

## 🪤 资产陷阱

当在处理资产的程序中出现错误时（因为大多数程序经常需要用 `BuyExecution` 来支付执行费用），那么就会产生很大的问题。可能会出现 `BuyExecution` 指令本身出错的情况，也许是因为权重限制不正确或用于支付的资产不足。也可能是资产被送到了一个无法有效处理的链上。在这些情况下，以及其他许多情况下，信息的 XCVM 执行结束时，资产仍在持有寄存器中，与其他寄存器一样是短暂的，我们希望能被遗忘。

团队和他们的用户会很高兴地知道，Substrate 的 XCM 允许链完全避免这种损失🎉。该机制分两步运作。首先，当持有寄存器中的任何资产被清空时，不会被完全遗忘。如果 XCVM 停止时 Holding Register 不是空的，那么就会发出一个事件，其中包含三条信息：Holding Register 的值；Origin Register 的原始值；以及这两条信息的哈希值。然后，Substrate 的 XCM 系统会将这个哈希值放入存储器。该机制的这一部分被称为 "资产陷阱"(Asset Trap)。

## 🎟 索赔系统

该机制的第二步是能够要求持有寄存器 (Holding Register) 中以前的一些内容。这实际上并不是通过任何专门设计的东西来实现的，而是通过一个我们还没有见过的叫做 `ClaimAsset` 的通用指令。下面是它在 Rust 中的声明方式：

```rust
pub enum Instruction {
    /* snip */
    ClaimAsset { assets: MultiAssets, ticket: MultiLocation },
    /* snip */
}
```

这个指令的名字似乎让人想起我们遇到的某些其他 "资金" 指令，如 `WithdrawAsset` 和 `ReceiveTeleportedAsset`。如果是这样，那么有一个很好的理由：它是和其他指令一样，它试图将资产（由这里的`assets`参数给出）放入持有寄存器中。与 `WithdrawAsset` 不同的是，`ClaimAsset` 会减少一个账户在链上的资产余额，而 `ClaimAsset` 则是寻找这些资产的有效索赔，无论原点寄存器的值是多少。为了帮助系统找到有效的索赔，可以通过`ticket`参数提供信息。如果找到一个有效的索赔，那么它将从链中删除，并将资产添加到持有寄存器中。

现在，究竟什么构成了索赔，完全由链本身决定。不同的链可能支持不同种类的索赔，而 Substrate 允许你轻松地组成它们。但是，正如你可能猜到的那样，有一种特殊的 claim 是可以随时使用的，那就是先前放弃的持有寄存器内容。

因此，让我们来看看这在实践中是如何运作的。假设我们的用户的 parachain 2000 向 Statemint 发送了一条消息，其中它从其主权账户中提取了 0.01DOT 以支付费用，并通知它有 100 个单位的自己的本地代币的储备资产转移到 Statemint 的主权账户。它可能看起来像这样：

```rust
WithdrawAsset((Parent, 100_000_000).into()),
BuyExecution {
    fees: (Parent, 100_000_000).into(),
    weight: Unlimited,
},
SetAppendix(Xcm(vec![
    ReportError {
        query_id: 42,
        dest: ParentThen(Parachain(2000)).into(),
        max_response_weight: 10_000_000,
    },
    RefundSurplus,
])),
ReserveAssetDeposited((ParentThen(Parachain(2000)), 100).into()),
DepositAsset {
    assets: All.into(),
    max_assets: 2,
    beneficiary: ParentThen(Parachain(2000)).into(),
}
```

假设 0.01DOT 是足够的费用，并且 Statemint 支持 parachain 2000 的本地资产的链上存款（以及使用 parachain 2000 作为它的储备），那么这应该是很好的工作。然而，也许 Statemint 还没有被设置为识别 parachain 2000 的本地资产。在这种情况下，`DepositAsset` 将不知道如何处理该资产并相应地抛出一个错误。在执行附录，通知 parachain 2000 这一失败后，我们将只剩下 parachain 2000 的 100 个单位的本地资产，以及持有寄存器中可能的一些 DOT。让我们假设这些费用只达到 0.005DOT，剩下 0.005DOT。

然后，Statemint 的 XCM pallet 会在这些新的可索赔资产上记录一个事件，类似于：

```rust
Event::AssetsTrapped(
    /* snipped hash */,
    ParentThen(Parachain(2000)),
    vec![
        (Parent, 50_000_000).into(),
        (ParentThen(Parachain(2000)), 100),
    ].into(),
)
```

一条信息将被发回给 parachain 2000，看起来像是：

```rust
QueryResponse {
    query_id: 42,
    response: ExecutionResult(Err((4, AssetNotFound))),
    max_weight: 10_000_000,
}
```

Parachain 2000 将在以后的某个阶段（也许一旦它确定 Statemint 能够接受其本地资产的存款），能够以相当简单的方式收回这 100 个单位。

```rust
ClaimAsset {
    assets: vec![
        (Parent, 50_000_000).into(),
        (ParentThen(Parachain(2000)), 100),
    ].into(),
    ticket: Here,
}
BuyExecution {
    fees: (Parent, 50_000_000).into(),
    weight: Unlimited,
},
DepositAsset {
    assets: All.into(),
    max_assets: 2,
    beneficiary: ParentThen(Parachain(2000)).into(),
}
```

在这种情况下，没有通过`ticket`参数提供特殊信息来帮助定位索赔。这对资产陷阱索赔来说通常没有问题，尽管对其他类型的索赔可能有必要使用它。

## 🏁 总结

那么现在就到这里吧 -- 我希望这篇文章能帮助你进一步了解 XCM 的底层虚拟机，以及它如何帮助你管理和恢复异常情况。本系列的下一篇文章将介绍 XCM 的未来发展方向，以及如何对该格式提出改进建议，并深入探讨 Substrate 的 XCM Rust 实现，以及我们如何利用它来为链提供轻松解释 XCM 的能力。
