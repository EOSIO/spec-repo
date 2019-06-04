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

This notification protocol is more flexible than `require_recipient`. It supports both
contract-to-contract signals and contract-to-outside events.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

The `require_recipient` notification protocol has these limitations:
* A notification is identical to the original action
  * This prevents augmenting the notification with auxillary data (e.g. new balances after a transfer)
  * This prevents generating multiple types of notifications from a single action
* If a contract sends a `require_recipient` to itself, then the notification is silently dropped.
  This complicates contracts which embed other contract code (e.g. eosio.token) but need to listen
  for notifications from the embedded contract.

This new notification protocol builds on [get_sender](https://github.com/EOSIO/eos/issues/7028)
(Nodeos 1.8) and has these properties:
* Contracts may send a variety of signals and events in response to a single action. These may contain any data that inline actions can handle.
* A contract may send signals to any contract or non-contract account, including itself. 
* A contract may send cheap context-free events to off-chain systems.
* Receivers may not charge RAM to the sender.
* Receiving contracts may authenticate signals using `get_sender`.
* Off-chain processes may authenticate signals and events by looking at `creator_action_ordinal` in the notification's action trace.

This EEP is a variation on signals and slots, similar to boost::signal2 or Qt. A major difference is that both sides must opt in to a pairing.
The sending contract chooses which contracts may opt into receiving, and the receiving contracts opt in to which contracts they listen to.
Once this system is implemented, `required_recipient` will be deprecated.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current EOSIO platforms.-->

### CDT Support: Sender

To define a signal, declare a function with an `eosio::signal` attribute. The first argument is the receiver.
The CDT will generate the function's body. e.g.:

```c++
// These are examples only. A future EEP will define a new token standard which differs
// from this.

[[eosio::signal]] void transferout(
    name receiver, name from, name to, asset amount, string memo);

[[eosio::signal]] void transferin(
    name receiver, name from, name to, asset amount, string memo);
```

The signal name must follow the rules for eosio names. To send a signal, call its function.

```c++
// This is an example only. A future EEP will define a new token standard which differs
// from this.

class token: public contract {
    ...
    [[eosio::action]] void transfer( 
        name from, name to, asset quantity, string memo) {
        ...
        transferout("alice"_n, "alice"_n, "bob"_n, asset("1.0000 SYS"), "A transfer");
        transferin("bob"_n, "alice"_n, "bob"_n, asset("1.0000 SYS"), "A transfer");
    }
};
```

To define an event, declare a function with an `eosio::event` attribute. There is no receiver argument;
events always go to `eosio.null`. The CDT will generate the function's body. e.g.:

```c++
[[eosio::event]] void gamestatus(name game, map<name, uint32_t> current_positions);
```

```c++
class game: public contract {
    ...
    [[eosio::action]] void moveto(name game, name player, uint32_t position) {
        ...
        gamestatus(game, current_positions);
    }
};
```

### CDT Support: Receiver

To receive a signal, define a member function on a contract with the `eosio::slot` attribute.
The slot attribute has a string argument which specifies the sender and slot.
e.g. `"eosio.token::transferin"`. It can use a wildcard for the sender: `"*::transferin"`.

```c++
// This is an example only. A future EEP will define a new token standard which differs
// from this.

class [[eosio::contract]] exchange: public contract {
  public:
    void [[eosio::slot("eosio.token::transferin"]] transferin(
        name receiver, name from, name to, asset amount, string memo)
    ) {
        // Handle the notification here
    }
};
```

### Protocol: Sender

```c++
eosio.signal(
    name  signal,   // signal name
    bytes payload   // signal payload
);

eosio.event(
    name  event,    // event name
    bytes payload   // event payload
);
```

Contracts send signals and events via the above actions. Contracts shouldn't send these manually;
they should let the CDT handle this task (above). The typical implementation:

* For signals, sends an inline action `eosio.signal` to the receiver.
* For events, sends a context-free inline action `eosio.event` to `eosio.null`.
* Doesn't include any authorizations. This prevents the receiver from charging RAM to the sender.

### Protocol: Receiver

Contracts opt-in to receiving signals by implementing the `eosio.signal` action. Contracts
shouldn't implement this action manually; they should let the CDT handle this task (above).
The typical implementation:

* Uses `get_sender` to determine which contract is sending the notification.
* Asserts that `get_sender` returned non-0.
* Dispatches `payload` to the appropriate function chosen by `notification`.

The contract shouldn't use `require_auth` and `has_auth` when processing `eosio.notify`; these would abort
the transaction. The receiving contract must pay for any RAM it uses.

### ABI Support

The ABI of the sender includes a struct definition for each notification. The struct name is
`eosio.notify.name`, where `name` is the name of the notification.

## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

## Backwards Compatibility
<!--All EEPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The EEP must explain how the author proposes to deal with these incompatibilities. EEP submissions without a sufficient backwards compatibility treatise may be rejected outright.-->

CDT versions 1.7 - yyy produce automatic dispatchers which assert on unknown actions, including `eosio.notify`.
If a contract sends a notification to a receiver built with those CDT versions, and that receiver uses the
automatic dispatcher, the whole transaction will abort.

## Test Cases
<!--Test cases for an implementation are mandatory for EEPs that are affecting consensus changes. Other EEPs can choose to include links to test cases if applicable.-->

## Implementation
<!--The implementations must be completed before any EEP is given status "Final", but it need not be completed before the EEP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->

## Copyright