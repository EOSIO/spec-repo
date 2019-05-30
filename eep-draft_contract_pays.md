---
eep: <to be assigned>
title: Contracts Paying Transaction Costs
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

Contracts can pay transaction costs (NET and CPU). This allows users with little or no resources to use
applications without requiring those applications to cosign the original transaction.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

EOSIO blockchains charge users NET and CPU costs on transactions. These costs
are temporary; CPU and NET replenish over a 24-hour period. A user can spend
all their resources on a set of transactions, then 24 hours later do it again.

Even though these costs are low on a per-user basis, they can add up quickly
for application providers. e.g. if a provider wants to sponsor 1000 accounts
which need Xpeek CPU and Ypeek NET, then the provider needs to stake
1000 * Xpeek CPU and 1000 * Ypeek NET. Most of this will go unused.

[ONLY_BILL_FIRST_AUTHORIZER](https://github.com/EOSIO/eos/issues/6332)
charges only the first authorizer of a transaction. This allows application
providers to cover costs from a common pool by cosigning each user
transaction. Unfortunately there is a downside: providers need to maintain
automated transaction cosigning services online. This has potential cost
issues and security issues.

This proposal adds a new capability: contracts can cover transaction
costs without cosigning. They can also track and limit resource
consumption to detect and prevent abuse. This gains the advantages of
ONLY_BILL_FIRST_AUTHORIZER without the hassle of cosigning.

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
