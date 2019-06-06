---
eep: <to be assigned>
title: Get Current BP
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

Provide the current BP to contracts

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

The [Subjective Data](eep-draft_subjective_data.md) proposal gives contracts a way
to obtain subjective data, but doesn't provide a way for contracts to check its
integrity. Contract authors could watch the behavior of their contracts over time for
malicious BP behavior, but they don't have a reliable way to restrict their contracts
from relying on data from bad BPs. This EEP gives contracts this ability.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current EOSIO platforms.-->

This consensus upgrade adds this intrinsic:

```c++
name get_current_bp();
```

* When a transaction is speculatively executed, this returns the BP that
  is scheduled to produce at the current position.
* When a block is being produced, this returns the BP that is producing it.
* What a block is being validated, this returns the BP that produced the block.

## Copyright
