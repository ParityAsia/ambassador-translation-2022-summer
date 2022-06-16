---
title: 'XCM Part II: Versioning and Compatibility'
src: https://polkadot.network/blog/xcm-part-two-versioning-and-compatibility/
author: Gavin Wood
snapshot-date: 2022-06-16
---

# XCM Part II: Versioning and Compatibility

In the first article I wrote on XCM, I introduced its basic architecture, goals and how it could be used for some simple use cases. Here we will move on to inspect one interesting aspect of XCM in depth: how XCM can change over time without introducing breakage between the very networks it is meant to connect.

Having a common language solves an awful lot of problems with human interaction. It allows us to work together, resolve conflicts and record information for later use. But language is only as useful as the concepts which it is able to express, and in an ever-changing world a language must change and adapt its conceptual repertoire or risk falling into disuse.

Unfortunately, changing a language too abruptly compromises its primary purpose — facilitating communication between people. Since languages must change, there must be ways of managing these alterations without making new forms unintelligible to the uninitiated. One very useful invention in this regard was the dictionary for helping to document and archive the conceptual palette of a language at one time so that future generations be better able to comprehend historic texts. An edition of a dictionary could be thought of as formalised “version” of a language.

Times may change but the problems remain eerily familiar. As I explained in the previous article, XCM is nothing but a language, albeit a very specialised one. It is a means for consensus systems to talk to one another, and as the needs for this XCM evolve at the breakneck speed of the crypto industry and the Polkadot ecosystem in particular, then there must be some means to ensure that these changes do not compromise the original goal of XCM: interoperability. We now need to solve not just interoperability in consensus space, but also in consensus time.

## 🔮 Versioning

Since we expect the language of XCM to change over time while very much being in use, one very simple precaution to take is to ensure that we identify which version of XCM we are communicating prior to the actual message content. We do this by using a number of version-wrapper types, so named because they wrap up an XCM message or a component thereof by a version. In Rust code, this looks very simple:

```rs
pub enum VersionedXcm {
    V0(v0::Xcm),
    V1(v1::Xcm),
    V2(v2::Xcm),
}
```

When sent “over the wire” (or, rather, between consensus systems), XCM is always placed in this versioned container. This ensures that systems too old to be able to interpret the message can safely receive them and recognise that the message’s format is unsupported by them. It also allows newer systems to recognise and accordingly interpret older messages.

Not just XCM messages are versioned; in the XCM codebase we also version MultiLocation, MultiAsset, as well as its associated types. This is because they may need to be stored and later interpreted when the chain’s XCM logic has been upgraded. Without versioning, we might otherwise attempt to interpret an old MultiLocation as a new one and find that it is incomprehensible (or worse, comprehensible but different to the original meaning).

## 💬 Compatibility & Translation

Versioning is a first step and ensures that we can identify the edition of the language which is being used. It does not ensure that we can interpret it and certainly does not ensure that it is the same edition we preferentially use. This is where compatibility comes in. By “compatibility” we mean the ability to continue to interpret and express ourselves in a version of XCM which is not our preferred version.

If we expect to be able to upgrade our network and its version of XCM at a schedule of our choosing, then this compatibility becomes rather important, since we may want to communicate with other networks who have not yet upgraded or, indeed, have already upgraded. This can be broken down into backward compatibility and forward compatibility. Basically speaking, backward compatibility is the ability of an upgraded system to continue to function in a legacy world, and forward compatibility is the ability of a legacy system to continue to function in an upgraded world.

In our case we would like to have both, however there are practical limitations: where a new version of XCM provides for capabilities that did not exist in prior versions, it is unrealistic to expect older systems to be able to interpret these messages. It would be a bit like trying to translate the term “social media” into Latin and then expecting it to be understood at face value by Julius Caesar. Some concepts simply cannot be expressed in a legacy context.

Similarly, significant changes to XCM might result in capabilities being removed from its conceptual model. This happens less often, but is similar to the problem of translating certain archaic terms into modern day equivalents. Interestingly enough, the archaic meaning of “dot” might be an example here (it used to mean a rather particular form of financial endowment).

Therefore new versions of XCM are designed to be mostly compatible with both older and newer versions, but generally there will be XCM messages which simply don’t make sense in the alternative context and will not be translatable.

## 🗣 Practical Communication

As mentioned before, we ensure that all messages which exist independently include a version identifier. This means messages sent between systems or messages persisted in storage. It does not include all messages, locations and assets though — data which exists as a part of other data need not be versioned since its version can be inferred from its context.

Version identification and compatibility/translation is helpful for receiving messages from an older network or sending messages to a newer network, but — taken alone — is less useful when going the other way. This is because a legacy network receiving a message from an upgraded network does not itself have the logic to be able to translate the new XCM into some form it can interpret — rather, that logic exists only on the sending side which has the translation code able to re-express the new message in legacy terms.

It must therefore be the responsibility of the sending network to ensure that the message it sends is capable of being interpreted by the receiving network. In concrete terms, the version of XCM used for the message must be no newer than the version of XCM that the receiving network supports.

For this reason, the Polkadot and Kusama Relay Chains, Statemint, Statemine, Shell and any other chains based on Substrate/Frame and its XCM engine, all keep a registry of the XCM versions supported by the remote chains. Whenever an XCM message is sent by these chains, it first determines what version to send the message in by consulting its registry. It translates the message to the older of the sender and the receiver’s supported XCM versions. For chains that stay up to date, then most of the time these will be the same, latest released, version, making available the full feature set of XCM.

This registry would normally be dictated and upgraded by governance processes, which is a bit cumbersome and tedious, especially as the number of potential destinations grows. For this reason, version-tracking was introduced.

## 🤝 Version Negotiation

Version-tracking is the final piece in the puzzle of XCM’s versioning story. Its function is to remove any off-chain or governance processes needed for tracking the XCM version of potential destination chains. Instead, the process happens autonomously and on-chain.

Essentially it works by allowing one network to use XCM to query another for the latest version of XCM which it supports, and to be notified whenever this changes. Replies that come from this query allow the network in question to populate and maintain its version registry, ensuring that messages are sent with the latest comprehensible version possible.

Specifically, there are three valuable instructions in XCM: SubscribeVersion, allowing one to ask another to notify it of its XCM version now and as it changes; UnsubscribeVersion to cancel that request; and QueryResponse, a general means of returning some information from the responder network back to the initiating network. This is what they look like in Rust:

```rs
enum Instruction {
    SubscribeVersion {
        query_id: QueryId,
        max_response_weight: u64,
    },
    UnsubscribeVersion,
    /* snip */
}
```

So SubscribeVersion takes two parameters. The first, query_id is of type QueryId, which is simply an integer used to allow us to identify and distinguish between the responses that come back. All XCM instructions which result in a response being sent have a similar means to ensure that their response can be recognised and dealt with accordingly. The second parameter is called max_response_weight and is a Weight value (also an integer) indicating the maximum amount of computation time that the reply should take by us when it returns. Like the query_id, this will be placed into any response messages that this instruction generates and is needed to ensure that any weight unpredictable, variable weight costs can at least be limited to a maximum prior to execution. Without this we would be unable to get an upper limit on the time that the reply message might take to interpret and thus be unable to schedule it for execution.

UnsubscribeVersion is rather barren as an instruction, primarily because only one version subscription is allowed to be active for a given location at once. This means that cancellation can happen with nothing more to identify it than the contents of the Origin Register.

An illustration of the version registry and its usage. Here, Chain A (XCM version 2) negotiates with Chain E (XCM version 3) and ultimately sends a version 2 message, which E would automatically translate to version 3 prior to interpreting it.

## 👂 Replying

The third instruction to be aware of is QueryResponse, which is a very much general purpose instruction allowing one chain to reply to another, and in doing so, report some information. Here it is in Rust:

```rs
enum Instruction {
    QueryResponse {
        query_id: QueryId,
        response: Response,
        max_weight: u64,
    },
    /* snip */
}
```

We already know two of of the three parameters, since they get filled from the values provided in SubscribeVersion. The third is called response and contains the actual information we care about. It is placed in a new type Response, itself a union of several types of which one network might wish to use to inform another network. It looks like this in Rust:

```rs
pub enum Response {
    Null,
    Assets(MultiAssets),
    ExecutionResult(Result<(), (u32, XcmError)>),
    Version(XcmVersion),
}
```

For our present purposes, only the Version item is needed, though as we will see in forthcoming articles, other items are useful for other contexts.

## ⏱ Execution time

Generally, we do not require QueryResponse instructions to purchase their own execution time with BuyExecution since (assuming they are valid), it was the now-interpreting network which requested that they be sent in the first place. Similarly we consider SubscribeVersion to be something broadly in the common interest of both sender and receiver and so wouldn’t expect that it need be paid for. In any case, the payment would be rather difficult to calculate due to the asynchronous and unpredictable nature of the responses it would generate.

## 🤖 Automation

While these XCM instructions allow a network to use entirely on-chain logic to determine the latest version that their interlocutor supports, there is still the question of when to initiate this version-discovery “handshake”. It cannot generally be done when a channel for sending XCM is created since transport-channel creation is of a conceptually lower level to that of XCM, which is one (perhaps of many) data formats which may be sent over that channel. To muddy the waters here could compromise the independence of the layered design. Furthermore, some cross-consensus transport protocols are not channel-based at all which would preclude the possibility of version negotiation on their inception.

Within Substrate chains such as the Polkadot Relay Chain and Statemint, the solution is to initiate this version discovery process automatically when a message needs to be wrapped for sending but the latest version of the destination is unknown. This has the slight drawback that the first messages would be sent under a suboptimal XCM version, which would happen until the version response was received. If this were a practical problem, then governance could step in to force the initial version of XCM for that destination to be something different to the default (generally set to the earliest XCM version still to be expected in production).

## ⌨️ Code Compatibility within XCM

The final point to address with regards to versioning is code authoring. Quite different to the over-the-wire format of XCM, code compatibility deals with what must happen to the codebases of (Substrate-based) projects which use the Rust implementation of the XCM stack over time as it evolves.

Clearly the codebases which aim to use an evolving language to express ideas must change and adapt with the times. We already have the Semantic Versioning (SemVer) system which helps dictate what changes may happen over particular version changes. However, this is really useful when dealing with APIs and ABIs and less so when considering an overall data format or language. Thankfully, XCM is designed to have little need for SemVer.

We know that newer versions of the XCM software are capable of translating between new and old XCM messages as well as their internal datatypes like locations and assets. It is able to do this by keeping several versions of the XCM language in the XCM codebase at once. Rust’s module system makes this trivial, with a new XCM version simply corresponding to a new Rust module. If we review the Rust declaration of the VersionedXcm datatype (right at the beginning of this article), it is simply the tagged union of each of the specific versions of the underlying Xcm datatype, each found in their own module v0, v1, v2, &c.

Since the transactions and APIs which use XCM and its datatypes tend to use only the versioned variants which are equally constructible with old and new formats, the end result is that codebases can be updated to use the most recent XCM software (in Rust, this is known as a crate) with few or no changes to their code. Upgrading the XCM crate allows a network to better interoperate with other similarly upgraded networks, but upgrading any fragments of XCM language that the network uses need not happen until a later time.

This acts as, I would hope, a strong incentive for teams to keep their XCM crates updated and thus keep everything iterating and evolving quickly.

## 🏁 Conclusion

I hope this has enlightened you regarding XCM’s versioning system and how it can be used to keep a network of sovereign chains communicating as the language they use to communicate evolves at differing rates and times between networks, and without a significant operational overhead on the developer teams who maintain their logic.

In the next installment, we will take a much deeper look into one of the most interesting parts of XCM: its execution model and exception management capabilities.
