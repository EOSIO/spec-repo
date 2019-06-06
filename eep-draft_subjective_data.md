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

Give contracts access to subjective data. This includes CPU charges for the current
transaction, wall-clock time, and a random number generator.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

Contracts don't have access to subjective data. e.g. they can't examine CPU costs. They
don't have access to wall-clock time. They don't have access to an unpredictable entropy
source. They could, if BPs recorded all subjective data they passed to the contracts.

This proposal defines a mechanism for BPs to record the subjective data they provide to
contracts. It also defines a set of intrinsics for accessing this data.

## Overview

There is 1 consensus upgrade in this proposal, which adds the following:
* A block extension which records subjective data provided to contracts
* A new intrinsic `get_trx_cpu_bill` that returns the current CPU charges for the transaction
* A new intrinsic `get_wall_time` that returns the wall-clock time with microsecond accuracy
* A new intrinsic `get_random` that returns a random number, assuming you trust the producers

There are some constraints on these functions (e.g. `get_wall_time` is non-decreasing during a single
transaction), but this doesn't propose a way to prevent BPs from manipulating this data to their advantage
(e.g. `get_random`).

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current EOSIO platforms.-->

### Block extension

When it's producing a block, nodeos will record subjective data that it provides to contracts into
a block extension. This allows other nodes to provide this data to the contracts while applying
blocks.

### Primitives

All of these primitives abort the transaction when used inside a deferred transaction. They charge NET
since they increase block size.

```c++
uint32_t get_trx_cpu_bill();
uint32_t get_wall_time();
void get_random(char* dest, size_t size);
```
`get_trx_cpu_bill` returns the current CPU charges for the transaction. During validation, nodeos
verifies the following:
* The result is non-decreasing across a transaction.
* The result is <= the final billed amount.

`get_wall_time` returns the wall-clock time with microsecond accuracy. During validation, nodeos
verifies the following:
* The result is non-decreasing across a block.
* The result is >= (the block time of the current block - 0.5s)
* The result is <= (the block time of the current block + 0.5s)

`get_random` fills a buffer with random data. Even though the reference implementation of nodeos
fills this with random data, there are no guarantees that individual BPs can not change this.
[Get BP](eep-draft_get_bp.md) proposes a potential counter-measure against BPs manipulating
subjective data.

## Copyright
