---
title: 'Polkadot Consensus Part 1: Introduction'
src: https://polkadot.network/blog/polkadot-consensus-part-1-introduction/
author: Joe Petrowski
snapshot-date: 2019-12-18
---

# Polkadot Consensus Part 1: Introduction

![](https://polkadot.network/content/images/2019/12/consensus-1@2x-2.png)

**This series will be a discussion about security and consensus in Polkadot. In part 1, we will define some terms before getting into the details of how Polkadot creates and secures blocks.**

Consensus algorithms help a network of computers operate like a single computer. In practice, this means that almost every computer in the network must agree on some initial state and then agree on a log of deterministic operations to the initial state, such that they arrive at the same final state.

While blockchains bring some interesting tools to this domain, this coordination problem is nothing new. It originated in aerospace, where computers on satellites or high-altitude airplanes might behave arbitrarily due to the inhospitable nature of space. Imagine that you have a network of flight computers and you want to know what direction your airplane is going. It shouldn’t matter which computer in the network you ask, you should always get the same response.

What does this have to do with blockchain? We want a network of computers to agree on some value. That value could be the balance of an account, the outcome of a vote or the execution result of a smart contract.

In fact, some pre-existing consensus algorithms resembled blockchains. In a [2001 lecture](https://ttv.mit.edu/videos/16444-practical-byzantine-fault-tolerance), MIT professor Barbara Liskov talked about batching transactions to improve Practical Byzantine Fault Tolerance (PBFT) performance, well before Bitcoin existed.

> **“Imagine a very busy primary who is getting hit with request after request after request; it doesn’t actually start the protocol for each request. Instead, it collects a batch of requests and does one protocol for the bunch of them. … It isn’t really necessary for everybody to send a reply to the client. It’s OK if all but one of them send digests of the reply because this will be sufficient to allow the client to tell whether it has identical replies.”**

PBFT provided a set of rules to agree on state changes — even a batch (read: block) — of state changes.

# Breaking Down Blockchain Consensus

In a distributed system like a blockchain, you need to answer a few questions:

1. Who can propose the next change?
2. Which set of changes is final?
3. What happens if someone breaks the rules?

It’s important to make these distinctions early because many blockchain consensus protocols unite them into one. Proof of work, for example, uses the proof to select the proper author of a block; the longest chain to decide which chain is final; and the cost of making that proof as the punishment for breaking the rules. In Polkadot, these questions are all answered in isolation.

Non-blockchain systems still answer these questions. For example, one could make the assumption that all computers run the same software. In most cases, this is fine. If Boeing makes an airplane, it’s safe to assume that they program all the computers on it.

In a public network, however, we can’t make such an assumption. Blockchains let us chisel down some of our network assumptions by using economics. All consensus systems have notions of “good” and “bad” behavior. The intrinsic economic properties in blockchains allow us to reward good behavior or punish bad behavior. A proof-of-stake network uses economics as the direct means of securing its consensus.

Security in a blockchain system is a measure of the difficulty of breaking consensus. In proof of authority, security is the difficulty to take control of the authorities. In proof of work, security is the cost to acquire and operate enough hash power to create a longer chain than the network. And in proof of stake, security is the value staked and the value at risk.

Members of Parity Technologies and Web3 Foundation developed and implemented a library of algorithms to tackle consensus and security. In this series, we will start with [GRANDPA](https://polkadot.network/polkadot-consensus-part-2-grandpa/), our finality algorithm, because all block production algorithms must respect this finality. Then we will move on to [BABE](https://polkadot.network/polkadot-consensus-part-3-babe/), our block-production engine, and discuss how we add blocks to the chain. Finally, we will end with a discussion on [how we use economics](https://polkadot.network/polkadot-consensus-part-4-security/) to secure GRANDPA and BABE.
