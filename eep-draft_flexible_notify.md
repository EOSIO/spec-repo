---
eep: Flexible Notifications
title: <EEP title>
author: <a list of the author's or authors' name(s) and/or username(s), or name(s) and email(s), e.g. (use with the parentheses or triangular brackets): FirstName LastName (@GitHubUsername), FirstName LastName <foo@bar.com>, FirstName (@GitHubUsername) and GitHubUsername (@GitHubUsername)>
discussions-to: <URL>
status: Draft
type: <Standards Track (Core, Networking, Interface)  | RFC | Meta>
category (*only required for Standard Track): <Core | Networking | Interface>
created: <date created on, in ISO 8601 (yyyy-mm-dd) format>
requires (*optional): <EEP number(s)>
replaces (*optional): <EEP number(s)>
---

<!--You can leave these HTML comments in your merged EEP and delete the visible duplicate text guides, they will not appear and may be helpful to refer to if you edit it again. This is the suggested template for new EEPs. Note that an EEP number will be assigned by an editor. When opening a pull request to submit your EEP, please use an abbreviated title in the filename, `eep-draft_title_abbrev.md`. The title should be 44 characters or less.-->

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the EEP.-->

This inline-action-based notification protocol is more flexible than `require_recipient`.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

The `require_recipient` notification protocol has these limitations:
* A notification is always identical to an original action
  * This prevents augmenting the notification with auxillary data (e.g. new balances after a transfer)
  * This prevents generating multiple types of notifications from a single action
* If a contract sends a `require_recipient` to itself, then the notification is silently dropped. 
This complicates contracts which embed other contract code (e.g. eosio.token) but need to listen 
for notifications from the embedded contract.

This new notification notification protocol builds on [get_sender](https://github.com/EOSIO/eos/issues/7028)
and has these properties:
* Contracts may send a variety of notifications in response to a single action. These may contain any data that inline actions can handle.
* A contract may send notifications to any contract or non-contract account, including itself. 
* Receivers may not charge RAM to the sender.
* Receiving contracts may authenticate these notifications using `get_sender`.
* Off-chain processes may authenticate these notifications by looking at `creator_action_ordinal` in the notification's action trace.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current EOSIO platforms.-->

## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->
## Backwards Compatibility
<!--All EEPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The EEP must explain how the author proposes to deal with these incompatibilities. EEP submissions without a sufficient backwards compatibility treatise may be rejected outright.-->
## Test Cases
<!--Test cases for an implementation are mandatory for EEPs that are affecting consensus changes. Other EEPs can choose to include links to test cases if applicable.-->

## Implementation
<!--The implementations must be completed before any EEP is given status "Final", but it need not be completed before the EEP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->

## Copyright
