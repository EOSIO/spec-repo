# Named Regions

## Simple Summary

This discussion piece covers one of the ideas we have for running contracts in parallel.
This idea may or may not be implemented.

## Abstract

eosio blockchains have a much higher transaction throughput than other public blockchains which
execute smart contracts. Performance has improved greatly since the initial release and continues
to advance, even though the transactions have a serial execution model. This serial model will
eventually limit further improvement; eosio needs a parallel model to move forward. This ESR
covers one of the ideas we're considering.

## Alternatives

Some of the potential approaches to parallel scaling include:

* Execute transactions from a single chain in parallel while checking for conflicts (e.g.
  accessing the same data)
* Side chains and sister chains. Applications which deal with multiple chains need some way
  of coordinating between the chains.
  * Proof-based IBC. Contracts verify proofs about events which happen on other chains. This can
    rely on the Merkle roots embedded in block headers.
  * Oracle-based IBC. Contracts may opt in to trusting oracles which attest to events on other
    chains.
* Named Regions. This ESR uses the term "regions" to mean side chains which share a common block
  log. The block log keeps the regions synchronized with each other and may help form the
  communication channel between them.

This ESR discusses named regions.

## Discussion

### Base Properties

Suppose we have a system with these properties:

* A single block holds the transactions for a set of named regions. This allows the
  regions to operate in lock-step, even during forks.
* Regions can't access each others' state; this allows them to execute in parallel.

To reduce load on validating nodes, and to limit network needs for these nodes, let's
add these properties:

* A node can choose to validate a particular subset of the regions and ignore the rest.
* Subsets of regions can be extracted from blocks.
* If a region doesn't have any transactions for a particular block and it doesn't have
  any incoming messages to queue (see IRC), then that region doesn't have to execute.
  We could exempt a main region from this to enable bookkeeping activities, such as
  onblock.

This system would create, in effect, a system of side chains under the control of a
common set of producers. Each region could have its own system contract for managing
resources. To simplify implementation, each region could also have its own set of
base-level accounts. Contracts could provide account portability using the
[Contract Authentication](esr_contract_trx_auth.md) and
[Forwarding Authorizations](esr_contract_fwd_auth.md) proposals, combined
with Inter-Region Communication.

### Inter-Region Communication (IRC)

IBC protocols need to deal with forking issues. A chain that's listening for the events
of another chain needs to either wait for that event to become irreversible, or deal
with it being undone by a fork change. Since regions share a common block history,
they don't have to deal with this issue when communicating with each other. Instead,
a region can assume that if an event happened on a prior block, it won't be undone.
It can assume this because a fork change which undoes that event also undoes
the effects it had on the receiving region.

Here are some potential IRC approaches:

* Trusted off-chain oracles forward events between regions. They use TaPoS to keep
  an event from one fork affecting another. Contracts need to be careful which
  oracles they trust. If an oracle is compromised, then any contracts that depend
  on it can also be compromised.
* Untrusted off-chain oracles forward events, along with Merkle proofs. Contracts
  would have to consume resources verifying these proofs.
* A cache of recent actions (identified by hash) can be asserted to exist from
  other regions without need of Merkle proofs. To validate this, it is critical
  that producers run all regions, but non-producers can assume that all asserted
  hashes are valid and therefore run just a subset of the regions.
* The system could provide message queues. If a contract on region A posts a message
  to a contract on region B, that message would become available to B on the next block.
  This isn't a form of deferred transactions. Instead, B would have to poll for messages.
  Messages would be recorded in blocks to enable nodes to validate subsets of regions.
* A single transaction could have multiple actions, where each action specifies which
  region it operates on. If an action on any region fails, the whole transaction would
  be rejected. The content of the whole transaction would be available to each action,
  allowing actions on one region to verify the appropriate action on another is taking
  place. A downside is that each transaction forms a synchronization point between
  regions, probably limiting performance. A potential way to counter this is to have
  multiple speculative transactions executing in parallel within each region, which
  are committed only if they don't have conflicting state access.

## Copyright
