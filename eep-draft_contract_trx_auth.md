---
eep: <to be assigned>
title: Contract-Defined Transaction Authentication
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

Contracts can define their own accounts and authenticate transactions for those accounts. This makes lightweight accounts more useful, enables more-flexible account structures, and helps accounts from other chains interact with eosio-based chains.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

EOSIO blockchains have a flexible and powerful account structure, but we see some areas to improve:
* Developers desire a way to onboard users at a much smaller individual cost (RAM, net, and cpu per user). 
* IBC mechanisms need a way to support accounts from other chains.
* Users need a convenient way to limit risk. e.g. allowing frequent small transfers while limiting large transfers.

What if contracts could define their own account structures? What if these contracts could authenticate 
transactions from those accounts? What if they could do their own resource billing? What if these 
contracts didn’t have to be privileged? This proposal gives contracts this capability.

## Overview

There are 3 consensus upgrades in this proposal:
* A new intrinsic `require_key` that asserts that a given public key was recovered from transaction signatures.
* Allow unused signatures. Nodeos will no longer reject transactions with signatures not used by the legacy authorizations.
* Allow a transaction without legacy authorizations, if a contract claims responsibility for it. This builds on the **** proposal.

## Example Use Case: Basic
* A user without a native account creates a transaction with this action:
```
    account:        "gamecontract"
    name:           "playgame1"
    data:           Arguments needed by game, including a user name.
    authorization:  empty
```
* The user signs the transaction with a key that gamecontract knows about
* The user pushes the transaction
* gamecontract calls `accept_charges` before leeway expires
* gamecontract uses `require_key` to authenticate the user
* gamecontract verifies that no other actions are present in the transaction to protect itself from paying for other contracts
* gamecontract executes the game logic

## Example Use Case: Authorization Manager
* A user with an account managed by "myauthmgr" creates a transaction with this action:
```
    account:        "myauthmgr"
    name:           "execute"
    data:           actions with authorizations
    authorization:  empty
```
* The user signs the transaction with a key that myauthmgr knows about
* The user pushes the transaction
* myauthmgr calls `accept_charges` before leeway expires
* myauthmgr checks the claimed authorizations and uses `require_key`
* myauthmgr verifies that no other native actions are present in the transaction
* myauthmgr updates `accept_charges` using the user’s available resources
* myauthmgr executes inline actions specified by the user. It includes an authorization attestation in each inline action. The contracts receiving these actions rely on the attestation.
* myauthmgr uses an inline action to check resource usage and updates the user’s resource consumption after the user’s actions have executed.




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
