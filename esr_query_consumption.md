# Query Resource Consumption

## Simple Summary

Allow contracts to query non-subjective resource consumption (NET and RAM).

## Abstract

Contracts don't have access to resource consumption data. e.g. they can't examine
transaction NET and CPU costs and they can't easily track their own RAM consumption.
This prevents them from accurately billing users for this resource usage, something
they may need to do if they [Pay Transaction Costs](esr_contract_pays.md).

This proposal defines a set of intrinsics for accessing non-subjective resource
consumption. A follow-up proposal, [Subjective Data](esr_subjective_data.md)
adds intrinsics for accessing subjective resource consumption.

This consensus upgrade adds the following intrinsics:
* `get_ram_usage` returns the amount of RAM used by the receiver
* `get_trx_net_bill` returns the current NET charges for the transaction
* `sync_context_free` synchronizes context-free actions to enable the contract to get accurate
  resource consumption

## Specification

This consensus upgrade adds these intrinsics:

```c++
uint32_t get_ram_usage();
uint32_t get_trx_net_bill();
void     sync_context_free();
```

`get_ram_usage` returns the sender's current RAM usage, in bytes. It aborts the transaction if it's deferred.

`get_trx_net_bill` returns the transaction's current NET usage, in words. It aborts the transaction if it's deferred.

`sync_context_free` synchronizes context-free actions to enable the contract to get accurate results from
`get_trx_net_bill` and `get_trx_cpu_bill` (see [Subjective Data](esr_subjective_data.md)).
The transaction aborts if:

* Any contracts create inline context-free actions after `sync_context_free` has been called. e.g. contracts may no
  longer send [events](esr_flexible_notify.md) after `sync_context_free`.
* The caller hasn't accepted charges. Only the first contract to call [accept_charges](esr_contract_pays.md) may
  use `sync_context_free`.
* The transaction is deferred.
