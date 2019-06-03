---
eep: Forwarding Authorizations Between Contracts
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

Contracts can attest user authentication to other contracts. This enables contract-defined accounts to use other contracts.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

The [Contract Authentication](eep-draft_contract_trx_auth.md) proposal allows contracts to define their own account
structures and authorize transactions from those accounts. It mentions that contracts may attest authorizations
to other contracts, but doesn't indicate how. This EEP proposes a protocol to support this, along with CDT enhancements
to simplify implementation.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current EOSIO platforms.-->

### CDT Support

todo

serializer support for streaming into checksum256

```c++
namespace eosio {
    struct subaccount {
        name        contract;
        checksum256 ident;
    };
}
```

### Protocol (receiver)

```c++
eosio.action(
    name                action, // action name
    vector<checksum256> idents, // identities
    bytes               payload // action payload
);
```

Contracts opt-in to supporting subaccounts from other contracts by implementing the above action.
Contracts shouldn't implement this action manually; they should let the CDT handle this task (above).
The typical implementation:

* Uses `get_sender` to determine which contract is attesting to the identities and stores this value 
  somewhere for later use.
* Asserts that `get_sender` returned non-0.
* Stores `idents` somewhere for later use.
* Dispatches `payload` to the appropriate function chosen by `action`.

Each subaccount is identified by the tuple `(get_sender(), ident)`. Having `get_sender()`
in the tuple guarantees that contracts don't tread on each other's account space.
Actions may check whether a particular subaccount `(x, y)` authorized the action by verifying that `x`
matches the value returned by `get_sender` and `y` is in `idents`. This check replaces 
`require_auth` and `has_auth`.

The contract shouldn't use `require_auth` and `has_auth` when processing `eosio.action`.
The receiving contract must pay for any RAM it uses.

### Protocol (sender)

The sender should send an inline action `eosio.action` with:
* The appropriate `action` and `payload`
* `idents` attesting to the authenticated account(s)

It's up to the sender to define how it encodes identities into checksum256. It could use the CDT's
serialization facilities to pack the sender's internal identity format (e.g. `name`, struct, etc.)
into 256 bits. It could also hash the identities if they are larger than 256 bits.

We recommend that senders avoid providing any native authorities to the inline action, even their own.
This will prevent the receiver from charging RAM to the sender. 

### ABI Support

## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

## Backwards Compatibility
<!--All EEPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The EEP must explain how the author proposes to deal with these incompatibilities. EEP submissions without a sufficient backwards compatibility treatise may be rejected outright.-->

## Test Cases
<!--Test cases for an implementation are mandatory for EEPs that are affecting consensus changes. Other EEPs can choose to include links to test cases if applicable.-->

## Implementation
<!--The implementations must be completed before any EEP is given status "Final", but it need not be completed before the EEP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->

## Copyright
