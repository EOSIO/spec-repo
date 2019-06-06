---
eep: <to be assigned>
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

This discussion piece covers one of the ideas we have for running contracts in parallel.
This idea may or may not be implemented.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

eosio blockchains have a much higher transaction throughput than other public blockchains which
execute smart contracts. Performance has improved greatly since the initial release and continues
to advance, even though the transactions have a serial execution model. This serial model will
eventually limit further improvement; eosio needs a parallel model to move forward. This EEP
covers one of the ideas we're considering.

## Alternatives

Some of the potential approaches to parallel scaling include:

* Execute transactions from a single chain in parallel after checking for conflicts (e.g.
  accessing the same data)
* Side chains and sister chains. Applications which deal with multiple chains need some way
  of coordinating between the chains.
  * Proof-based IBC. Contracts verify proofs about events which happen on other chains. This can
    rely on the Merkle roots embedded in block headers.
  * Oracle-based IBC. Contracts may opt in to trusting oracles which attest to events on other
    chains.
* Named Shards. This EEP uses the term "shards" to mean side chains which share a common block
  log with a main chain. The block log keeps the shards synchronized with each other and may
  help form the communication channel between them.

This EEP discusses named shards.

## Description

* Named shards / regions
  * Alternatives for communication. Identify pros and cons of each:
    * Communication protocol (e.g. maybe a message queue)
    * Ability to prove an inline action in another shard
  * BPs don't have to process every shard every block. only if there's a transaction.

fork together; don't need to wait for irreversible
can replay just a subset of shards

## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

## Backwards Compatibility
<!--All EEPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The EEP must explain how the author proposes to deal with these incompatibilities. EEP submissions without a sufficient backwards compatibility treatise may be rejected outright.-->

## Test Cases
<!--Test cases for an implementation are mandatory for EEPs that are affecting consensus changes. Other EEPs can choose to include links to test cases if applicable.-->

## Implementation
<!--The implementations must be completed before any EEP is given status "Final", but it need not be completed before the EEP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->

## Copyright
