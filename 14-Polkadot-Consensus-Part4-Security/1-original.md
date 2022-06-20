---
title: 'Polkadot Consensus Part 4: Security'
src: https://polkadot.network/blog/polkadot-consensus-part-4-security/
author: Joe Petrowski
snapshot-date: 2019-12-18
---

# Polkadot Consensus Part 4: Security

![](https://polkadot.network/content/images/2019/12/unnamed.png)

*This is part 4 in our Polkadot consensus series. See [part 1](https://polkadot.network/polkadot-consensus-part-1-introduction/) for the introduction, [part 2](https://polkadot.network/polkadot-consensus-part-2-grandpa/) for a discussion about GRANDPA, and [part 3](https://polkadot.network/polkadot-consensus-part-3-babe/) for a discussion about about BABE.*

So far, we have discussed how BABE creates blockchain candidates and GRANDPA finalizes them. We know that we need more than two-thirds of validators to properly follow the protocol. But how many validators are there? How are they chosen? Why should they follow the rules?

### Elections and Eras

For validators to know that a block has more than two-thirds agreement, GRANDPA needs to know how many validators there are in total. The chain’s governance process sets (and can change) the number, but the goal is to have at least 1,000 validators running BABE and GRANDPA in Polkadot.

Once we know how many validators there will be in the set, we hold an election to decide who gets to be a validator. Just as BABE breaks time into epochs, GRANDPA breaks time into eras. At the end of each era, rewards for the past era are paid, and an election takes place for the next era. Eras are planned to be about 24 hours long.

Polkadot uses nominated proof of stake (NPoS) to elect validators and Phragmen’s method to perform the election. [The introduction](https://polkadot.network/polkadot-consensus-part-1-introduction/) discussed that security in a proof-of-stake network is related to the value at risk. Users signal their intent to participate in the security of the network by locking funds, called staking.

While the number of validators is limited, the number of people who can participate in the security of the network by staking is not. If you are not a validator, you can still participate by nominating. Nominators stake their funds and select up to 16 validators that they trust to validate on their behalf. Nominators share in the rewards, but also the punishment, of the validators that they back.

One of Polkadot’s objective functions is having an evenly staked validator set. Rewards are paid based on performance, not based on stake, so a nominator will get a higher rate of return by staking behind smaller validators.

We optimize this stake distribution using Phragmen’s method. Prior to an election, there is a list of accounts wishing to become validators. Each validator has a list of potential nominators. Phragmen’s method will first pick the winners by figuring out the combination that will lead to having the most value at stake. Once it knows the set, it will apply the nominations in such a way that will lead to the most evenly staked set. This outcome will lead to the highest security for the network and most rewards for the nominators.

### Rewards

Rewards are the primary incentive for people to run validators on the network. As discussed in parts 2 and 3, validators run the BABE and GRANDPA protocols to create and finalize the Polkadot blockchain.

Unlike other proof-of-stake protocols, Polkadot determines rewards based on validator activity and not based on how much each validator has at stake. Validators accumulate points based on their activity. Points are primarily assigned for signing validity statements and producing blocks that are included in the canonical chain. Some additional points are issued for the production of blocks that do not end up in the canonical chain.

Points do not have a corresponding dot value until the end of the era, when we know the total number of points issued in the era. The total reward of an era is divided among validators based on how many points each one accumulated relative to the total. Then, the reward is divided among each validator’s nominators.

Following the reward system is simple. By running the standard client and having a high-availability network architecture, validators will be able to properly follow the protocols and earn points.

### Discipline and Punish

Rewards provide the reason to stake, but the network must make sure that stakers follow the rules. Slashing is the punishment for failing to follow the protocol. The security of the network requires that the punishment for attempting an attack is so great as to prevent the attack from occurring.

Infractions range from plainly lackadaisical to downright mendacious. The most basic requirement for a validator is to be online and available. A validator proves its availability either by authoring a block or by sending a heartbeat message to the network. Slashes for being offline are quite low because every system can experience periodic downtime within reason. As long as a validator takes proper care of their infrastructure configuration, though, downtime should be a rare event, and the slash is small enough to recover from.

A more serious transgression is equivocation. Equivocation can occur in both BABE and GRANDPA. In BABE, equivocation is producing two blocks in the same slot. In GRANDPA, it is sending pre-vote or pre-commit messages for two chains that conflict with each other in the same round. Equivocation comes with a severe slash. If too many validators equivocate, it would be impossible to select a single, canonical chain.

Some behavior can be slashed up to 100% of the amount at stake. These actions would be in extreme cases, such as casting pre-votes or pre-commits for a chain that conflicts with the chain that is already final. The network considers such behavior is an attack because it is attempting to revert finalized blocks.

### Super-linear Slashing

You may have already noticed that rewards are not related to stake, so if you have enough funds to run two validators, you could double your rewards.

This behavior is encouraged. We expect single entities — be they large holders or staking-as-a-service providers — to run multiple validators. There is nothing Polkadot can do to prevent certain entities from acquiring large amounts of stake and running validators. To protect against a single entity gaining too much power, Polkadot can make them increase their value at risk should they try anything nefarious.

Polkadot uses super-linear slashing. As the number of validators who commit an offense increases, so does the percentage of the slash. For example, if a single validator equivocates, it could be due to a poor infrastructure setup. However, if 30% of the validators equivocate in one round, it is more likely to be a coordinated attack and the slash will be more severe.

![img](https://lh6.googleusercontent.com/OyrrlOe5_zGiJdR5ZKVPP0zG4eyX6gaZBTuVOYoUrsbqpw94eCuO142kL4GFdIbWLQS6848j7vsaEVRZGwCUKYIgFKhLiguMTL0QNRmLfV0pS_UJeNNIwjWLJ3gackMeZq_hWHFq)*As more validators equivocate, the slashing severity increases. When over 33% of validators equivocate, an event that would halt the network, the offenders are slashed 100%.*

As a single entity adds more validators to the network, it will have to ensure that the validators are not dependent on each other or any centralized service.

### Shared Security

Security in proof-of-stake networks depends on economics, so there can only exist a limited amount of security in the world because economic value is, by definition, limited. As the number of blockchains increases due to scaling issues on single chains, their economic value — and therefore their security — gets spread out over multiple chains, leaving each one weaker than before.

Smart contracts that execute in a shared execution environment, like the Ethereum Virtual Machine, can interact without trust bounds. With Polkadot, the logic interface moved from a single execution environment in a blockchain to the blockchain’s logic itself.

But when considering how to make chains interact while breaking out of trust bounds, one must realize that trust does not come from executing in the same environment. Trust comes from operating under the same economic and state transition guarantees.

Polkadot introduces a shared security model so that chains can interact with others while knowing full well that their interlocutors have the same security guarantees as their own chain. Bridge-based solutions — where each chain handles its own security — force the receiver to trust the sender.  Polkadot’s security model provides the necessary guarantees to make cross-chain messages meaningful without trusting the security of the sender.

The Relay Chain blocks mostly consist of proofs of validity from parachains, meaning that when the Relay Chain validates a parachain’s state transition and includes a proof in the finalized Relay Chain, the parachain’s block is also final. To revert the parachain’s block, an attacker would have to revert the entire Polkadot system, including every single parachain. Security doesn’t compete; it adds.

This system of sharing security by sharing state in the Relay Chain obviates the need for parachains to even provide their own security and validator community. Polkadot’s Relay Chain provides this economic guarantee so that chains in the Polkadot ecosystem can focus on the logic that drives their application.

To learn more about consensus and economics in Polkadot, visit the [Polkadot Wiki](https://wiki.polkadot.network/docs/en/).
