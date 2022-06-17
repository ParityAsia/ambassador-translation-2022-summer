---
title: 'XCM Part III: Execution and Error Management'
src: https://polkadot.network/blog/xcm-part-three-execution-and-error-management/
author: Gavin Wood
snapshot-date: 2022-06-17
---

# XCM Part III: Execution and Error Management

In the first two articles (Part I, Part II) I wrote on XCM, I introduced the basics of its design and versioning structure. In this article, we will take a deeper look into its underlying design and execution model. Since XCM is based around the instruction set of the XCVM, a very high-level virtual machine, this amounts to becoming familiar with this machine architecture.

The XCVM is a very high level, non-Turing complete virtual machine. It is register-based (rather than stack-based) and has several special-purpose registers, most of which hold highly structured data. Unlike general-purpose processors, the XCVM’s registers are not free to be set to arbitrary values, but have strict mechanics governing how they may change. Beyond certain means of interacting with the local chain state (such as the WithdrawAsset and DepositAsset instructions which we have already seen) there is no additional “memory”. There is no possibility of looping and no explicit branch instructions.

We have already been introduced to two of the registers: the Holding Register, which is able to temporarily hold one or more assets and may be populated through withdrawing an asset from the local chain, or else through taking receipt of an asset from a trusted external source (e.g. another chain); and the Origin Register, which at the beginning of execution holds the location of the consensus system from which current XCM execution originated, and may only be mutated into an interior location or cleared entirely.

Of the other registers, three are concerned with exception/error management and two with tracking execution weight. We will learn about all of them in this article.

## 🎬 Execution Model

As mentioned already, there are no explicitly conditional instructions or looping primitives which make it possible to re-execute the same instruction more than once. This makes it fairly trivial to predetermine a programme’s control flow. This property is useful given that we want to determine how much execution time (known as weight throughout Substrate/Polkadot) an XCM message could utilise prior to the point of execution.

Most consensus platforms which we expect to execute XCM will need to be able to determine a worst-case execution time prior to the commencement of execution. This is due to blockchains typically needing to ensure that individual blocks do not take longer to process than some predetermined limit lest it cause the system as a whole to stall. Additionally, if fee payment is needed by the system, then it must necessarily happen prior to the workload that the payment is being taken for and it’s important that this payment cover the worst-case execution time.

Systems that allow for Turing-complete languages (e.g. Ethereum) cannot actually calculate the worst-case execution time from the programme directly owing to this Turing-completeness. They get around this by requiring the user to predetermine the execution resources of the programme and then by metering it as it executes and interrupting it should it exceed the amount that was paid for. Sometimes things change before the transaction gets executed and the weight becomes incorrect. Happily, virtual machines such the XCVM which are not Turing-complete can avoid the need for this metering and weight-prescription.

## 🏋️‍♀️ Weight

Weight is typically represented as the integer number of picoseconds which a piece of representative hardware would take to execute the given operation. As we have seen with the BuyExecution instruction, the XCVM includes this concept of execution time/weight when dealing with certain instructions.

There is no metering of weight, but to allow for the possibility of an XCVM programme ultimately taking less than the worst-case weight prediction, we have a register called the Surplus Weight Register. Most instructions don’t touch it since we can accurately predict how much weight they will use. However, there are occasionally circumstances where the worst-case weight prediction is an over-estimate and only at the time of execution do we know by how much. While accounting for block execution time with an over-estimate of the weight of the XCM message, tracking the amount by which the original weight is an overestimate and subtracting it from the accounts allows the chain to optimise its quota of block execution time.

So the Surplus Weight Register is useful for our block execution time accounting, but doesn’t alone solve the other issue of ensuring the amount paid is not an over-estimate. For this, we need a companion instruction to BuyExecution, which takes any surplus weight and refunds it. Naturally, this instruction exists and is called RefundSurplus. There is a second register which it utilises called the Refunded Weight Register, ensuring that the same surplus weight is not refunded multiple times.

## 😱 Flow Control and Exceptions

Two more registers have been rather implicit in our treatment of the XCVM so far, but are nonetheless important to know about. Firstly, there is the Programme Register which stores the currently executing XCVM programme. Secondly, there is the Programme Counter, which stores the currently executing instruction index. This gets reset to zero when the Programme Register is changed and is incremented by one at the end of every successfully executed instruction.

The ability to handle the possibility of an “exceptional” circumstance is crucial in writing robust code. When something happens on a remote system which you didn’t expect (or indeed could not have predicted), then you need some way of managing it, even if it is simply to send a report back to the origin stating as much.

While the XCVM instruction set does not include any explicit general purpose branch instructions, it does have a general exception-handling framework built into its execution model. The XCVM includes two more code registers, each one holding an XCVM programme like the Programme Register. These two registers are called the Appendix Register and the Error Handler Register. If you’re familiar with the try/catch/finally exception system in several popular languages, then what is to follow might seem quite reminiscent.

As mentioned, an XCVM programme’s execution follows each instruction in it, step by step. As it follows these instructions to the end of the programme, one of two things will happen: either it will reach the end of the programme successfully, or an error will occur. In the first case of successful execution, the Error Register is cleared and its weight added to the Surplus Weight Register. The Appendix Register is also cleared and its contents are placed in the Programme Register. If the Programme Register is left empty, then we halt. Otherwise the Programme Counter is reset to zero. Put simply, we toss out the current programme and error handler and begin executing the appendix programme if there is one.

This functionality isn’t so useful on its own, but can be useful when combined with the what happens in the case of an error. Here, the weight of any instructions yet to be executed is added to the Surplus Weight Register. The Error Handler Register is cleared, its contents placed in the Programme Register and the Programme Counter reset to zero. Put simply, we toss out the current programme and begin executing the error handler. Because we do not clear the Appendix Register, then unless it gets reset by the error handler, then it will execute once that finishes successfully.

Owing to its compositional structure, it allows for arbitrary “nesting” of error handlers: error handlers can, if desired, also have error handlers and appendices can have their own appendices.

There are two instructions which allow these registers to be manipulated: SetAppendix and SetErrorHandler. As you might expect, one of them sets the Appendix Register and the other the Error Handler Register. The predicted weight of each of these is a small amount more than the weight of their parameter. However, when executed, the weight of the XCM message in the register which will be replaced is added to the Surplus Weight Register, allowing the weight of any unused appendix or error handler to be reclaimed.

## ☄️ Throwing Errors

Sometimes it might be useful to actually ensure that an error happens and customise some aspect of that error. This has been used while writing test code but it’s not impossible that it may eventually find use within a live chain. This can be done in the XCVM through the instruction Trap which always results in an error happening. The error type which gets thrown shares the name Trap. Both the instruction and the error carry an integer argument allowing some form of information to be passed between the error thrower and an external onlooker.

Here’s a trivial example:

```rs
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

The Trap causes the final DepositAsset to be skipped and the instead the error handler’s DepositAsset to run, placing the 1 DOT (minus execution cost) under the ownership of parachain 2000. We will always tend to use RefundSurplus at the beginning of an error handler code since if it is running we know it is likely that the predicted weight used (and thus weight purchased) is an over-estimate.

## 🗞 Error Reporting

Being able to introduce code to handle errors is very useful, but one oft-requested feature is to be able to report the outcome of an XCM message back to the original sender. We met the QueryResponse instruction in the previous article which allows one consensus system to report some information back to another, all that remains is to be able to somehow insert the outcome of the XCM into this QueryResponse and send it to whoever is expecting to be told of the result.

It turns out that there is precisely one instruction which does that named ReportError. It works by using a register we have not yet come across: the Error Register. The Error Register is an optional type (it may be either set or clear). If it is set, then it holds two pieces of information: a numeric index and an XCM error type.

It has extremely simple mechanics of operation. Firstly, it always becomes set whenever an instruction results in error; the error type is set to the type of that error, and the numeric index is set to the value of the Programme Counter Register. Secondly, it becomes cleared only when the ClearError instruction is executed. This instruction is one of the infallible instructions — it is never allowed to result in an error itself. That’s all — it gets set when an error happens and gets cleared when you issue the appropriate instruction.

It should now be clear to understand how the ReportError instruction works: it simply composes a QueryResponse instruction using the contents of the Error Register and sends it to a particular destination. Of course any error which happens before it would result in the instruction being skipped as execution jumps first to the Error Handler Register’s code and then to the Appendix Register’s code. However, the solution to this is trivial: placing ReportError in the appendix will ensure it is executed regardless of whether the main code resulted in an execution error.

Let’s take a look at a simple example. We will teleport an asset (1 DOT) from the Relay Chain over to Statemint (parachain 1000), buy some execution time there and then using Statemint as a reserve we’ll deposit the asset on parachain 2000. The original (non error-reporting) message would look like this:

```rs
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

With basic error reporting we would instead use this:

```rs
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

As you can see, the only change is the introduction of two SetAppendix instructions which ensures that the error or lack thereof within both Statemint and parachain 2000 will be reported to the Relay Chain. This assumes that the Relay Chain has set itself up to be able to recognise and handle QueryResponse messages originating from Statemint and parachain 2000 with query ID 42 and a weight limit of ten million. Happily, this is indeed something which Substrate supports well, but out of scope right now.

## 🪤 The Asset Trap

When errors occur during programs that deal with assets (as most do since they will often need to pay for their execution with BuyExecution), then it can be very problematic. There may be instances where the BuyExecution instruction itself results in error, perhaps because the weight limit was incorrect or the assets used for payment were insufficient. Or perhaps an asset gets sent to a chain which cannot deal with it in a useful way. In these cases, any many others, the message’s XCVM execution finishes with assets remaining in the Holding Register, which like the other registers are transient and we would expect to be forgotten about.

Teams and their users will be happy to know that Substrate’s XCM allows chains to avoid this loss entirely 🎉. The mechanism works in two steps. First, any assets in the Holding Register when it gets cleared do not get completely forgotten. If the Holding Register is not empty when the XCVM halts, then an event is emitted containing three pieces of information: the value of the Holding Register; the original value of the Origin Register; and the hash of these two pieces of information. Substrate’s XCM system then places this hash in storage. This part of the mechanism is called the Asset Trap.

## 🎟 The Claim System

The second step to the mechanism is being able to claim some previous contents of the Holding Register. This actually happens not through anything specially designed for this purpose but rather through a general purpose instruction that we have not yet met called ClaimAsset. Here’s how it’s declared in Rust:

```rs
pub enum Instruction {
    /* snip */
    ClaimAsset { assets: MultiAssets, ticket: MultiLocation },
    /* snip */
}
```

The name of this instruction might seem reminiscent of certain other “funding” instructions that we have met such as WithdrawAsset and ReceiveTeleportedAsset. If it does, then it’s for a pretty good reason: it is. Like the others, it attempts to place the assets (given by the assets argument here) into the Holding Register. Unlike e.g. WithdrawAsset which reduces an account’s on-chain asset balance, ClaimAsset looks for a valid claim for these assets available to the whatever the value is of the Origin Register. To help the system find the valid claim, information may be provided via the ticket argument. If a valid claim is found, then it is deleted from the chain and the assets added into the Holding Register.

Now, exactly what constitutes a claim is entirely up to the chain itself. Different chains may support different kinds of claim, and Substrate allows you to compose them easily. But, as you may guess, one particular kind of claim that comes ready to go, of course, is that of previously dropped Holding Register contents.

So let’s take a look at how this might work in practice. Suppose our user’s parachain 2000 sends a message into Statemint in which it withdraws 0.01 DOT from its sovereign account to pay for fees and also notifies it of a reserve-asset transfer of 100 units of its own native token to be placed into its sovereign account on Statemint. It might look something like this:

```rs
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

Assuming that 0.01 DOT is enough fees for this and that Statemint supports on-chain deposits of the native asset of parachain 2000 (as well as using parachain 2000 as a reserve for it), then this should work just fine. However, perhaps Statemint has not yet been set up to recognise parachain 2000’s native asset. In this case, the DepositAsset will not know what to do with the asset and accordingly throw an error. After executing the appendix which will notify parachain 2000 of this failure, then we will be left with the 100 units of parachain 2000’s native assets as well as potentially some DOT in the Holding Register. Let’s assume the fees only amounted to 0.005 DOT, leaving 0.005 DOT remaining.

Then there would be an event recorded by Statemint’s XCM pallet over these newly claimable assets, something like:

```rs
Event::AssetsTrapped(
    /* snipped hash */,
    ParentThen(Parachain(2000)),
    vec![
        (Parent, 50_000_000).into(),
        (ParentThen(Parachain(2000)), 100),
    ].into(),
)
```

A message would be sent back to parachain 2000 that looks like:

```rs
QueryResponse {
    query_id: 42,
    response: ExecutionResult(Err((4, AssetNotFound))),
    max_weight: 10_000_000,
}
```

Parachain 2000 would at some later stage (perhaps once it has determined that Statemint is able to accept deposits of its native asset), be able to reclaim those 100 units with a rather simple:

```rs
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

In this case, no special information is provided through the ticket argument to help locate the claim. This is usually fine for the Asset Trap claims, though it may be necessary to use it for other types of claims.

## 🏁 Conclusion

So that’s it for now — I hope this has been instrumental in helping you understand more about XCM’s underlying virtual machine and how it can help you manage and recover from unexpected situations. The next articles in this series will cover future directions in XCM and how improvements can be suggested to the format as well as take a deeper dive into Substrate’s XCM Rust implementation and how we can use it to furnish a chain with the ability to easily interpret XCM.
