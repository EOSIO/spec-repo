# Contract-Defined Transaction Authentication

## Simple Summary

Contracts can define their own accounts and authenticate transactions for those accounts. This makes lightweight accounts more useful, enables more-flexible account structures, and helps accounts from other chains interact with eosio-based chains.

## Abstract

EOSIO blockchains have a flexible and powerful account structure, but we see some areas to improve:
* Developers desire a way to onboard users at a much smaller individual cost (RAM, net, and cpu per user). 
* IBC mechanisms need a way to support accounts from other chains.
* Users need a convenient way to limit risk. e.g. allowing frequent small transfers while limiting large transfers.

What if contracts could define their own account structures? What if these contracts could authenticate 
transactions from those accounts? What if they could do their own resource billing? What if these 
contracts didn’t have to be privileged? This proposal, which builds on
the [Query Consumption](esr_query_consumption.md), [Subjective Data](esr_subjective_data.md),
and [Contract Pays](esr_contract_pays.md) proposals, grants contracts this capability.

## Overview

There are 3 changes in this consensus upgrade:
* A new intrinsic `require_key` asserts that a given public key was recovered from transaction signatures.
* Nodeos will no longer reject transactions with signatures not used by the native authorizations, if those keys are checked by `require_key`.
* Nodeos will allow transactions to omit native authorizations, if a contract accepts the charges using `accept_charges`.
  This builds on the [Contract Pays](esr_contract_pays.md) proposal.

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
* gamecontract verifies that no other actions are present in the transaction. This prevents it from paying for other contracts.
  It can do this by verifying `get_sender` returns 0, the number of non-context-free actions returned by `get_num_actions` is
  1, and the number of context-free actions returned by `get_num_actions` is 0.
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
* myauthmgr checks the claimed authorizations using `require_key`
* myauthmgr verifies that no other native actions are present in the transaction. This allows it to accurately track resource consumption.
  It can do this by verifying `get_sender` returns 0, the number of non-context-free actions returned by `get_num_actions` is
  1, and the number of context-free actions returned by `get_num_actions` is 0.
* myauthmgr updates `accept_charges` using the user’s available resources
* myauthmgr executes inline actions specified by the user. It includes an authorization attestation in each inline action.
  The contracts receiving these actions rely on the attestation. See [Forwarding Authentication](esr_contract_fwd_auth.md)
  for details on this step.
* Inside an inline action, myauthmgr calls `sync_context_free`, checks `get_trx_net_bill` and `get_trx_cpu_bill`, then updates the user’s
  resource consumption. This inline action executes last.

## Specification

### New Intrinsic

```c++
void require_key(char* pub, size_t publen);
```

This intrinsic verifies that a given public key was recovered from transaction signatures. It aborts the
transaction if either the key wasn't recovered, or if it's called from a deferred transaction.

Nodeos normally rejects transactions which have recovered keys not needed by the native authorizations. Nodeos
will no longer reject these transactions, if all the extra keys were referenced by `require_key`. It verifies
all keys are used to prevent a resource attack. If someone intercepted a transaction, added unneeded signatures,
then delivered that transaction to the scheduled producer before the original, that modified transaction would
spend more CPU than necessary. Since all recovered keys must be referenced, this attack won't work.

### Enhancement to accept_charges

[Contract Pays](esr_contract_pays.md) adds this intrinsic:

```c++
bool accept_charges(
    uint32_t max_net_usage_words,   // Maximum NET usage to charge
    uint32_t max_cpu_usage_ms       // Maximum CPU usage to charge
);
```

Nodeos normally rejects transactions which have no authorizations. After this consensus upgrade, nodeos
will no longer reject these transactions if a contract accepts the charges during the leeway.

No contract may accept charges during a deferred transaction.

## Copyright
