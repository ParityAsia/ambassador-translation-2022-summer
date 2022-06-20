---
title: 'Polkadot Consensus Part 2: GRANDPA'
src: https://polkadot.network/blog/polkadot-consensus-part-2-grandpa/
author: Joe Petrowski
snapshot-date: 2019-12-18
---

# Polkadot Consensus Part 2: GRANDPA

![https://polkadot.network/content/images/2019/12/consensus-grandpa@2x-1.png]()

***This is part 2 in our Polkadot consensus series. See [part 1](https://polkadot.network/polkadot-consensus-part-1-introduction/) for the introduction.\***

In the introduction to this series, I outlined that a consensus algorithm helps a network of computers answer three questions. GRANDPA addresses the second.

1. Who can propose the next change?
2. ***\*Which set of changes is final?\****
3. What happens if someone breaks the rules?

GRANDPA is Polkadot’s finality gadget. Its purpose is to deterministically select the canonical chain. In other words, GRANDPA decides which set of changes is final.. It doesn’t produce blocks on its own; instead, GRANDPA validators import blocks from another block production module (which we will discuss in part 3).

One of the benefits of separating block production and safety¹ — besides being generally good engineering — is that GRANDPA doesn’t impose many constraints on the blocks that it imports. GRANDPA only requires that the block production system have eventual safety, follow the fork choice rule of GRANDPA and that a block’s header have a pointer to its parent block. That third property ensures that [light clients](https://www.parity.io/what-is-a-light-client/) can follow the chain.

# The GRANDPA Protocol

GRANDPA stands apart from other Byzantine fault-tolerant (BFT) blockchain algorithms in that validators vote on **chains**, not blocks. The protocol applies votes transitively and the GRANDPA algorithm finds the highest block number with a sufficient number of votes to be considered final. This process allows several blocks to be finalized in one round.

This last part is significant because it removes a bottleneck that hampers other blockchain finality gadgets. GRANDPA, like other PBFT derivatives, has **O(n²)** complexity. That is to say, if you double the number of nodes, you have to send four times the number of messages. Consensus systems that make block production part of the finality process make you send these messages for every single block. By isolating block production in another module, we can produce blocks in a much more efficient manner (**O(n)** for BABE) and finalize several of them together in one round.

To see an example of this, look at these log messages from a Kusama node:

```
Idle (24 peers), best: #664257 (0x706c…76b7), finalized #664253 (0xe4ab…4d2a)
Imported #664258 (0xee71…6321)
Idle (24 peers), best: #664258 (0xee71…6321), finalized #664256 (0x809a…a5d8)
```

Notice that in one round, GRANDPA finalized three blocks (664,254 to 664,256).

![img](https://polkadot.network/content/images/2019/12/image1.png)*This is an example of how GRANDPA can finalize multiple blocks in one round, as seen in the log messages above. The dark gray blocks on the left were previously final, and validators (gray dots on the right) have sent votes for the new round. Three more blocks had a supermajority and were finalized.*

# A GRANDPA Round

Voters perform the following to finalize new blocks:

1. A node that is designated as the “primary” broadcasts the highest block that it thinks could be final from the previous round.
2. After waiting for a network delay, each validator broadcasts a “pre-vote” for the highest block that it thinks should be finalized. If the supermajority of validators are honest, this block should extend the chain that the primary broadcast. This new chain could be several blocks longer than the last finalized chain.
3. Each validator computes the highest block that can be finalized based on the set of pre-votes. If the set of pre-votes extends the last finalized chain, then each validator will cast a “pre-commit” to that chain.
4. Each validator waits to receive enough pre-commits to form a commit message on the newly finalized chain.

A subtle but important distinction from other Byzantine fault-tolerant algorithms (like PBFT and Hotstuff) is that there is no view change on the critical path. Although the primary changes each round, this view change is only to start a new round in asynchronous network conditions so that in partially synchronous networks, the protocol will always advance, even without assigning a primary.

Steps in the protocol are completable when they have more than two-thirds of the pre-votes or pre-commits from validators. To have deterministic finality, the number of votes in the validator set must be limited. This is different to chains with probabilistic finality, which can have unlimited validator sets. The method for selecting the set of voters is logic that is defined outside of the GRANDPA protocol ([see part 4](https://polkadot.network/polkadot-consensus-part-4-security/)).

GRANDPA supports weighted voting. For example, you could implement GRANDPA on your chain where validators with more stake get more votes. In Polkadot, however, all validators have a single, equally weighted vote. This weighting was an economic decision to prevent small sets of nodes from gaining a large network share.

# Accountable Safety: When Things Go Wrong

GRANDPA has a feature called **accountable safety** to hold validators accountable for safety violations. A safety violation occurs when two blocks that are in different chains are finalized. Accountable safety is like an investigation after an accident.

But first, how did two conflicting chains reach finality to begin with? BFT systems are always built on the requirement that the maximum number of faulty validators is some fraction — in our case one-third — of the total validators. In order to finalize two conflicting chains, the validator set has failed to meet this requirement; at least one-third of the validators voted on these two chains.

![img](https://polkadot.network/content/images/2019/12/image3.png)*In this example, there are 10 validators, meaning 3 is the maximum number of faulty validators the system can withstand (f = (10 - 1) / 3). With 4 faulty validators (red) and a network partition, each group of honest validators (blue) can think a different block is final.*

Voting on two conflicting chains is called equivocating. It is a truth universally acknowledged, that equivocation is an affront to BFT systems. In GRANDPA, we can detect it.

First, we start asking nodes why they didn’t consider one block final when they voted to finalize the second block. Any honest validator should answer this with a set of pre-votes or pre-commits for the second round that have a supermajority for the second block.

If that’s the case, then we ask a second question: Which pre-votes for the first round have you seen? We are essentially asking them to rat on other validators and reveal all the votes they received from peers. Somewhere in the union of both sets you will discover the validators who voted for the two conflicting chains. Presumably, they will be heavily punished, but that is the business of the chain’s logic, not consensus.

If a safety fault occurs, then the network will have to have a hard fork to select which of the conflicting chains is final. With accountable safety, Polkadot can ensure that the validators who performed the attack are punished and do not remain in the validator set.

# How GRANDPA Helps Availability and Validity

Remember the log messages above? Notice that the finalized block is two blocks behind the **best** block. This lag is actually an advantage of keeping block production and finalization distinct.

```
Idle (24 peers), best: #664258 (0xee71…6321), finalized #664256 (0x809a…a5d8)
```

Blockchain interoperability systems, Polkadot included, have a data availability problem. Imagine that a single collator submits a block to a validator, but none of the other parachain collators have seen it. What would happen if the collator who submitted the block went offline? Validators have a responsibility to store complete blocks for a period of time so that any parachain collator can ask for the block.

Validators should execute blocks before voting for them, but we want to make sure they do so. A number of nodes, which we call fishermen, exist in Polkadot to execute blocks and report any validator misbehavior, e.g. proposing an invalid parachain block for inclusion in the Relay Chain.

We never want a situation where we finalize an invalid block or finalize a block that collators cannot reconstruct. By keeping finality a few blocks behind the chain tip, we can let fishermen verify that the blocks are correct and challenge the validators for block availability.

We’ve been discussing how to decide on the canonical chain, but where do these chain options come from? That is where BABE comes in. See [part 3](https://polkadot.network/polkadot-consensus-part-3-babe/) of this series.

[1] I prefer the term “safety” over “finality” because finalized blocks are not final in a laws-of-physics kind of way. They are final because GRANDPA said so. I much prefer the word “safe,” which comes with more reasonable expectations than “final.” For example, we consider air travel safe. But we all know that sometimes planes crash. Further, when a plane does crash, we have a process of investigation and legal recourse to hold certain parties accountable.
