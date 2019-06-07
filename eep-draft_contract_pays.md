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
are transient; CPU and NET replenish over a 24-hour period. A user can spend
all their resources on a set of transactions, then 24 hours later do it again.
Even though these costs are low on a per-user basis, they can add up quickly
for application providers. e.g. if a provider wants to sponsor 1000 accounts
which need Xpeek CPU and Ypeek NET, then the provider needs to stake
1000 * Xpeek CPU and 1000 * Ypeek NET. Under this model, users can use these
resources for other apps, not just yours. In addition, most of this will
likely go unused.

[ONLY_BILL_FIRST_AUTHORIZER](https://github.com/EOSIO/eos/issues/6332) (Nodeos 1.8)
partially addresses this by charging only the first authorizer of a transaction.
This allows application
providers to cover costs from a common pool by cosigning each user
transaction. There is a downside: providers need to maintain
automated transaction cosigning services online. This has potential cost
issues and security issues.

This proposal adds a new capability: contracts can cover transaction
costs without cosigning. They can also limit resource consumption to 
prevent abuse. Thanks to the [Query Consumption](eep-draft_query_consumption.md)
and [Subjective Data](eep-draft_subjective_data.md)
proposals, contracts also can track resource consumption on transactions.
This proposal has the advantages of ONLY_BILL_FIRST_AUTHORIZER without
the hassle of cosigning.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current EOSIO platforms.-->

This consensus upgrade adds these intrinsics:

```c++
bool accept_charges(
    uint32_t max_net_usage_words,   // Maximum NET usage to charge
    uint32_t max_cpu_usage_ms       // Maximum CPU usage to charge
);

void get_accept_charges(
    name*     contract,
    uint32_t* max_net_usage_words,
    uint32_t* max_cpu_usage_ms,
);

void get_num_actions(
    uint32_t* num_actions,
    uint32_t* num_context_free_actions,
);
```

If a contract is the first one to call `accept_charges` during a transaction, then that contract's account will be billed
for the transaction, up to the limits specified. If the transaction exceeds these limits, then it will abort.
It will also abort if the contract's account doesn't have enough resources to cover the transaction.
The contract may call this intrinsic multiple times, within a single action or across multiple actions; this
enables the contract to adjust the limits. The original NET and CPU limits in the transaction have no effect
once a contract accepts the charges.

If multiple contracts call `accept_charges`, then the first one is the one charged. `accept_charges` returns true to that
contract, but false to the others. If the first one calls it multiple times, within the same action or across
multiple actions, it will return true each time.

`get_accept_charges` queries whether any contracts have accepted charges. If a contract has accepted charges,
then this fills the arguments with the details. If no contract has accepted charges, then it fills the arguments
with 0.

`get_num_actions` returns the number of non-context-free actions and the number of context-free actions in
the original transaction. This does not include any inline actions. This intrinsic helps contracts protect
themselves against paying for other contracts' actions; see the example use cases in
[Contract Authentication](eep-draft_contract_trx_auth.md) for advice on how to do this check.

`accept_charges` has no effect and always returns false during deferred transactions.

[Contract Authentication](eep-draft_contract_trx_auth.md) builds on this proposal to add an additional
capability to `accept_charges`.

## Copyright
