---
eep: <to be assigned>
title: Query Resource Consumption
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

Allow contracts to query non-subjective resource consumption (NET and RAM).

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

Contracts don't have access to resource consumption data. e.g. they can't examine
transaction NET and CPU costs and they can't easily track their own RAM consumption.
This prevents them from accurately billing users for this resource usage, something
they may need to do if they decide to
[Pay Transaction Costs](eep-draft_contract_pays.md).

This proposal defines a set of intrinsics for accessing non-subjective resource
consumption. A follow-up proposal, [Subjective Data](eep-draft_subjective_data.md)
adds intrinsics for accessing subjective resource consumption.

This consensus upgrade adds the following intrinsics:
* `get_ram_usage` returns the amount of RAM used by the receiver
* `get_trx_net_bill` returns the current NET charges for the transaction

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current EOSIO platforms.-->

```c++
uint32_t get_ram_usage();
uint32_t get_trx_net_bill();
```

`get_ram_usage` returns the sender's current RAM usage, in bytes. It aborts the transaction if it's deferred.

`get_trx_net_bill` returns the transaction's current NET usage, in words. It aborts the transaction if it's deferred.

## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

## Backwards Compatibility
<!--All EEPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The EEP must explain how the author proposes to deal with these incompatibilities. EEP submissions without a sufficient backwards compatibility treatise may be rejected outright.-->

## Test Cases
<!--Test cases for an implementation are mandatory for EEPs that are affecting consensus changes. Other EEPs can choose to include links to test cases if applicable.-->

## Implementation
<!--The implementations must be completed before any EEP is given status "Final", but it need not be completed before the EEP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->

## Copyright
