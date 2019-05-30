---
eep: <to be assigned>
title: Contract Access To Subjective Data
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

Give contracts access to subjective data. This includes NET and CPU charges for the current
transaction, wall-clock time, and a random number generator.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

Contracts don't have access to subjective data. e.g. they can't examine NET and CPU costs. They
don't have access to wall-clock time. They don't have access to an unpredictable entropy
source. They could, if the BP recorded all subjective data it passed to the contracts.

This proposal defines a mechanism for BPs to record the subjective data they provide to
contracts. It also defines a set of intrinsics for accessing this data.

## Overview

There is 1 consensus upgrade in this proposal, which adds the following:
* A block extension which records subjective data provided to contracts
* A new intrinsic `get_resource_usage` that returns the current NET and CPU charges for the transaction
* A new intrinsic `get_wall_time` that returns the wall-clock time with millisecond accuracy
* A new intrinsic `get_random` that returns a random number

There are some constraints on these functions (e.g. `get_wall_time` is non-decreasing during a single
transaction), but this doesn't propose a way to prevent BPs from manipulating this data to their advantage
(e.g. `get_random`).

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
