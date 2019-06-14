# Get Current Producer

## Simple Summary

Provide the current producer to contracts

## Abstract

The [Subjective Data](eep-draft_subjective_data.md) proposal gives contracts a way
to obtain subjective data, but doesn't provide a way for contracts to check its
integrity. Contract authors could watch the behavior of their contracts over time for
malicious producer behavior, but they don't have a reliable way to restrict their contracts
from relying on data from bad producers. This EEP gives contracts this ability.

## Specification

This consensus upgrade adds this intrinsic:

```c++
name get_current_producer();
```

* When a transaction is speculatively executed, this returns the producer that
  is scheduled to produce at the current position.
* When a block is being produced, this returns the producer that is producing it.
* What a block is being validated, this returns the producer that produced the block.

## Copyright
