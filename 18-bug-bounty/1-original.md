---
title: 'Parity Bug Bounty Program'
src: 'https://www.parity.io/bug-bounty/'
author:
snapshot-date: 2022-06-21
---

# Parity Bug Bounty Program

## Help Us Make Parity More Secure!

We work hard to make sure the systems we build are bug-free, but acknowledge that we might not catch them all. We call on our community and all bug bounty hunters to help identify bugs in the protocols and software. If you discover a bug, we appreciate your cooperation in responsibly investigating and reporting it to us so that we can address it as soon as possible.

Our Parity Bug Bounty Program allows us to recognise and reward members of the Parity community for helping us find and address significant bugs, in accordance with the terms of the Parity Bug Bounty Program set out below.

We want to remind all hunters that Parity’s main projects are blockchain-related source code (located in our Github repositories) and associated released binaries, and not websites or services in any form. This is the reason for our Bug Bounty Program covering only the former, and not the latter.

## What's In Scope? (Hint: Not Our Website!)

If you've found a potential bug in Substrate, Polkadot, or associated build and deployment infrastructure, then we want to hear from you!

Parity welcomes vulnerability reports that demonstrate security flaws in:

* Substrate: Any bugs which can be used to bring down or take control of Substrate clients without direct access to the machine, including bugs in Substrate pallets and Substrate primitives.

* Smoldot: Any bugs which can be used to bring down or take control of Smoldot light clients without direct access to the machine.

* Polkadot
    * Client: Any bugs which can be used to bring down or take control of Parity Polkadot client without direct access to the machine.
    * Runtimes: Any bugs that compromise the intended behavior of the various Parity-built blockchain runtimes (Kusama, Polkadot, etc).

* Parity releases pipeline: any bugs which could be used to enable an attacker to inject malicious code into our distributed binaries, or be used to halt Parity’s release process or add malicious/unintended functions to the released binaries.

* Production infrastructure: publicly-available infrastructure Parity runs for production-grade networks (in contrast to testnets), especially parts which are critical for network’s well-being or safety of funds.

* Cryptography code: any bugs relating to cryptography, encryption, decryption, and signing of messages (this includes account creation and recovery).

* Client Application Security: bugs which can allow users to obtain privileges not intended for them.

Please note that where the scope of this policy includes third-party code this should not be taken as an indication that we are legally or otherwise responsible for that code, its security, quality or your rights in respect of that code.

## Exclusions — What's NOT in Scope

Did you find a bug in our open source blockchain code or related infrastructure? Great! Tell us about it!

Most other things are not in scope, though. Specifically:

* The Parity websites, including https://parity.io and https://substrate.dev (including subdomains), and any others we might make in the future, are not in scope.

* Bugs which have already been submitted by another user or are already known to the Parity team or have already been publicly disclosed.

* Bugs in third-party tools and services we’re using (but we would be glad to connect you with the security team of the corresponding project).

* Parity Technologies’ development team, Parity Technology employees and any other person employed or providing services in any way to the company, directly or indirectly, are not eligible for rewards.

* Anything that contravenes the spirit or letter of this Program.

## Be Nice and So Will We!

Responsible investigation and reporting includes, but isn't limited to, the following:

* You should use your best effort not to access, modify, delete, or store user data or Parity’s data. Instead, use your own accounts or test accounts for security research purposes.

* Don’t defraud, harm, or violate the privacy of Parity Technologies Ltd or its users during your research; you should make a good faith effort to not interrupt or degrade our services.

* Don't target our physical security measures, or attempt to use social engineering, spam, distributed denial of service (DDOS) attacks, etc.

* Initially report the bug only to us and not to anyone else.

* Keep the details of any suspected bug confidential.

* After reporting a suspected bug, give us a reasonable amount of time to fix the bug before disclosing it to anyone else, and seek our approval before disclosing it to anyone else. An uncoordinated public disclosure may lead to your submission being disqualified from the Program.

* Please don’t make repeat submissions of low quality, rejected or automated vulnerability reports.In general, please investigate and report bugs in a way that makes a reasonable, good faith effort not to be disruptive or harmful to us or our users. Otherwise your actions might be interpreted as an attack rather than an effort to be helpful.

## Is My Bug Eligible?

To be eligible for a reward under this Program:

* Play by the rules - this includes complying with the spirit and letter of this policy as well as any other applicable laws or agreements.

* The security bug must be original and previously unreported. Duplicate submissions made within 72 hours of each other will split the bounty between reporters. If duplicate submissions are of unequal quality, the split will be at the level of the lesser report, and the greater report will receive a pro-rated additional bounty on top of the split.

* The security bug must be a part of Parity’s code, not the code of a third party. We will pay bounties for vulnerabilities in third-party libraries (for instance, libp2p) incorporated into shipped client code utilized by Parity if both of the following two conditions are met:

    * the bug leads to an exploitable vulnerability in Parity's software in particular, and
    * is not actively maintained by another commercial entity with a separate bug bounty program.

* You must not have written the buggy code or otherwise been involved in contributing the buggy code to the Parity project.

* You must be old enough to be eligible to participate in and receive payment from this Program in your jurisdiction, or otherwise qualify to receive payment, whether through consent from your parent or guardian or some other way.

* You must not be an employee, contractor, or otherwise have a business relationship with the Parity or any of its subsidiaries.

* If you inadvertently access, modify, delete, or store user data, we ask that you notify Parity immediately at bugbounty@parity.io and delete any stored data after notifying us.

* We might be prevented by law from paying you. For example, if you happen to live in a country on a sanctions list that applies to us. In this case, if we can we would be happy to make a donation to a well-established charity of your choice.

* You must not either directly or indirectly exploit the security vulnerability for your own gain or incite, encourage or assist anyone else in doing so.

* Before sharing any part of the security issue with a third party, you must give us a reasonable amount of time to address the security issue.

* To the extent that you propose a fix that includes code we will ask you to sign our standard contributor license agreement in respect of that fix so that we can deploy it going forwards.

Do not threaten or attempt to extort Parity. We reserve the right to disqualify individuals from the Program if they threaten to withhold the security issue from us or if you threaten to release the vulnerability or any exposed data to the public or any third party or otherwise act in a malicious, disrespectful, or disruptive manner.

We want your bugs! But please note that it's entirely at our discretion to decide whether a bug is significant enough to be eligible for reward. Our lawyer made us write this. You understand.

## Ka-Ching! How We Pay You 😀

Bug Bounty Hunter Program rewards are at the sole discretion of Parity Technologies.

* The minimum reward for eligible bugs is the equivalent of 100 USD in KSM.
* Rewards over the minimum are at our discretion, but we will pay significantly more for particularly serious issues, i.e. that the identified issue could put a significant number of users at risk of severe damage, monetary or otherwise.
* Each bug will only be considered for a reward once.
* Bounty eligible bug hunters will be asked to send their proof of identity and KSM address to be rewarded.

## How to Report a Bug

* Is there a bug in our website? It's not eligible! Scroll back up to learn what is in scope — namely, our open source blockchain technology: Substrate, Polkadot, and associated infrastructure.
* Still want to report? Send your bug to bugbounty@parity.io, including the information below:
    * step by step details to reproduce
    * affected components
    * potential impact
* Please be as detailed as possible. The easier it is for us to reproduce your bug, the faster we can fix it — and the faster we can pay you! Try to include as much information in your report as you can, including a description of the bug, its potential impact, and steps for reproducing it or proof of concept.
* Please add a Github link to the repo you’ve found a bug in right in the email title — this will help our laborious robots to route your email accordingly.
* Please allow two business days for us to respond before taking any further action.

Once the issue has been submitted, our team will review the information, assign a severity level (that may or may not be similar to your choice) and redirect this to one member of the Bug Bounty Program team, who will contact you with more details on the next steps.

## What Our Lawyers Want You to Know!

The Parity Bug Bounty Program is a discretionary rewards program for our active community to encourage and reward those who are helping to improve the systems we build. It is not a competition. We can cancel the Program at any time and awards are at the sole discretion of Parity Technologies development team. All Bug Bounty awards are subject to compliance with local laws, rules, and regulations. We are not able to issue awards to individuals who are on sanctions lists or who are in countries on sanctions lists. You are responsible for all taxes payable in connection with the receipt of any rewards. All rewards are subject to the laws of England and Wales. Finally, your testing must not violate any law or compromise any data — or funds — that are not yours.

We will do our best to respond to your submission as quickly as possible, keep you updated on the fix, and award a bounty where appropriate. If you follow these guidelines in discovering and disclosing a vulnerability, we will not consider your actions as an attack and won’t take any legal action against you.

### Privacy

As part of participating in the Bug Bounty Program you will need to share with us personal data including your name, email address, ID information and photos, and a blockchain address. Parity Technologies is committed to protecting and respecting your privacy. To understand how Parity uses your personal data please see our privacy policy (https://www.parity.io/privacy/). If you want to contact us about this please email legal@parity.io.

### Governing Law and Jurisdiction

Any obligations arising out of or in connection with the Parity Bug Bounty Program or its subject matter will be governed by and construed in accordance with the law of England and Wales, and the courts of England and Wales shall have exclusive jurisdiction to settle any dispute or claim (including non-contractual disputes or claims) arising out of or in connection with the Parity Bug Bounty Program.

## Legal Safe Harbour

Parity strongly supports security research into Substrate and Polkadot and wants to encourage that research. If you conduct genuine, in-scope, bug hunting research in good faith and in accordance with this policy we will consider your actions to be legitimate and will not seek prosecution. But for the avoidance of doubt, this does not give you permission to act in any manner that is inconsistent with the law or might cause Parity to be in breach of any of its legal obligations.

We understand that many Parity systems and services are interconnected with third-party systems and services. While we can authorize your research on Parity’s systems and services we cannot authorize efforts on third-party products or guarantee they won’t pursue legal action against you.

**If you’re not sure whether your conduct complies with this policy, please contact us first at bugbounty@parity.io and we will do our best to clarify.**

## Got Questions? We Got Answers!

If you have a query or complaint about the Parity Bug Bounty Hunter Program, please contact us using the same bugbounty@parity.io email address.
