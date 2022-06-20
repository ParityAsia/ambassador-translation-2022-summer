---
title: 'Polkadot Consensus Part 3: BABE'
src: https://polkadot.network/blog/polkadot-consensus-part-3-babe/
author: Joe Petrowski
snapshot-date: 2019-12-18
---

# Polkadot Consensus Part 3: BABE

![](https://polkadot.network/content/images/2019/12/unnamed-1.png)

***This is part 3 in our Polkadot consensus series. See [part 1](https://polkadot.network/polkadot-consensus-part-1-introduction/) for the introduction and [part 2](https://polkadot.network/polkadot-consensus-part-2-grandpa/) for a discussion about GRANDPA.\***

Blind Assignment for Blockchain Extension (BABE) is a block production engine that was inspired by Ouroboros Praos, another proof-of-stake protocol. It can be used on its own because it provides probabilistic finality, or it can be coupled with a finality gadget like GRANDPA.

BABE is a slot-based algorithm. It breaks time into epochs, with each epoch being broken into slots. In Polkadot, each slot is six seconds long, our target block time. BABE will select an author (or several) to author a block in each slot.

![img](https://lh6.googleusercontent.com/40vQ5xCzLj6lNbQtlU0sc12djJzSBJjGGCT26Gjz0zuqOTnp5dUKfm0gBrNN0oLVqCCZMbav-5e925tEo1er3Vu6TeM-2oAep7tGqryWbVMOzUCoNCS171_cHIx3jiLs4M02MISC)*Time in BABE is broken into epochs. Each epoch is a set of slots.*

One way to assign authors to these slots would be simply to take turns. However, in a round-robin pattern, adversaries always know who the next author is and can use that information to coordinate attacks. Ideally, nobody knows who the slot author is until he or she proves it.

Each slot can have a primary and secondary author (or “slot leader”). *Primary* slot leaders are assigned randomly. Because the function is random, however, sometimes there are slots without a leader. In order to ensure a consistent block time, BABE uses a round-robin system to assign *secondary* slot leaders.

### Primary Slot Leaders

Primary leadership is granted based on the evaluation of a verifiable random function (VRF). There’s a lot of hype around random numbers in blockchain. To make a long story short, a lot of applications depend on random number generation, but it can be difficult to find randomness that everyone agrees is random (and not gamed such that the generator benefits) when all on-chain operations must be deterministic and verifiable.

VRFs generate a pseudo-random number along with a proof that it was properly generated. They take some parameters, including a private key, as input. Our VRF takes an epoch random seed (agreed upon in advance by all nodes), a slot number and the author’s private key. Because no two nodes have the same private key, each node can generate a unique, pseudo-random value for each slot

Each author evaluates its VRF for each slot in an epoch. For each slot whose output is below some agreed-upon threshold, the validator has the right to author a block in that slot. Because of the random slot assignment process, it’s possible to have slots without a primary as well as slots with multiple primaries. We will discuss how we deal with that later.

![img](https://lh5.googleusercontent.com/riRgJd3Ig7VDEododtnDJlSy-EnmejVujOniszdL_KSYcUBKbuOloEXgGZofLN5WbrAA0igpRMZ0BjaD5mVtGVFoVzpeVd4_4a1-1MZpG0lwqM9Z0aKSC_7642VxpbMl4Ju_QzFe)*The VRF in BABE takes an epoch randomness, slot number and validator private key as input and outputs a value for each slot in an epoch. When a block author’s output is below the network’s threshold, it produces a block as a primary block leader for that slot.*

### Secondary Slot Leaders

To deal with empty slots, BABE uses a round-robin fallback. Every slot has a secondary leader. If nobody claims that they are the primary at the beginning of the slot, then the secondary will author a block. This fallback will make sure that every slot has a block author and helps to guarantee a consistent block time.

### Bringing BABE and GRANDPA Together

Up to now, we have GRANDPA finalizing chains and BABE creating new blocks. Some of BABE’s chains may have forks since a single slot can have multiple leaders.

The first rule of choosing the best chain to extend is simple: BABE must build on a chain that has been finalized by GRANDPA. This is one of the requirements to use GRANDPA.

A second, more subtle, requirement for using GRANDPA is that the block production algorithm must have a way of choosing the “best” chain. This property leads to BABE having probabilistic finality (thus, you could use it without GRANDPA).

The best chain in BABE is simply the one with the most blocks authored by primaries.

![img](https://lh5.googleusercontent.com/49HOLTl2c1qQZfWCn5ud_zKT2_9zgp0AdQrr_RBMGHeAZXfPXK3UlGJmIHZ1p43oyrJZHy6HBC8XoHGnnwYycLyP4YSuxi1vZCFixJ4IUHqB1RRsLiGBaSKQUvqXe680GP4dXISE)*An example of BABE’s fork choice rule for choosing the best chain.*

Forks are common in BABE. As discussed in the GRANPA article, block production is *O(n)*, meaning that the author just has to broadcast its block to everyone but does not need everyone to send a message to everyone (like in GRANDPA). So, not everyone will have the same view of the non-finalized chain (yellow blocks in the image).

This system lets us produce blocks in an efficient way and lets GRANDPA finalize a set of them.

### Wait, Whose Clock?

We are assigning slots based on time, but we don’t have a single view of time. Every computer has its own clock. We can’t use a centralized time service (called NTP servers) because that is a single point of attack. An attacker could attack the NTP server, either cutting it off or taking control to get up to more unscrupulous behavior like sending different times to different nodes.

If you’re interested, consider this scenario:

> I receive a message from you saying “it’s 8:42:00.” My clock says it is 8:42:03. One of three things is possible:
>
> \1. Our clocks are in sync, it just took 3 seconds for the network to deliver your message.
>
> \2. It actually took 1 second to deliver your message. Our clocks are out of sync by 2 seconds.
>
> \3. You’re lying to me, that’s not what your clock said.

![img](https://lh6.googleusercontent.com/jj9KK5gvVt2J4-Rencpp5iCVcf2iFqXQzH3s_lFRof-FNgzKBO76fPIhX4thKwF6D0ZOu4IEbDudC_W-Q1Dip3zxuwOIGlfHUUk1emrcZ_XsXVRn78iNRGzDQYQasa3_IKg2qEnk)

> Now imagine that I receive this message when my clock says 8:41:59. If I believe that you are honestly telling me what your clock said, then I know that we are out of sync and that I must set my clock forward. I still don’t know the time it took to deliver via the network, so I don’t know how much out of sync we are.

BABE aligns slot numbers to an individual computer’s clock using *relative time.* When a node receives a block, it checks the reception time and the slot number associated with the block. It then adds the number of slots to each block’s time to forecast future slots and uses the median value from its data. Remember, validators know the slot numbers for which they will author in advance, so they can check incoming blocks against this.

![img](https://lh6.googleusercontent.com/VBu_4PlCcgWyQO5El4xSXEJvuATjyUpjhLYNGnVoZItP697sjo_x6Fqwxzr8LV_31KutvMCqyzr34TGzRFEAtMU50q-699zDYRbA2HjGIb44Ovk93hlS9PoywkyRtUqykoU7XQI5)*Block authors in BABE use the reception time for blocks to create a view of the network’s time. They project the reception times into the future, based on the slot time, to determine when they should author and propose a block.*

So far we have discussed how chains are made (BABE) and how they are finalized (GRANDPA). The next problem we have to solve is, *how do we get people to run these protocols in the correct way?* The [final part](https://polkadot.network/polkadot-consensus-part-4-security/) of this series will discuss how the runtime provides incentives to run BABE and GRANDPA and the punishments for errors.
