# Get Current Producer

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the EEP.-->

Provide the current producer to contracts

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

The [Subjective Data](eep-draft_subjective_data.md) proposal gives contracts a way
to obtain subjective data, but doesn't provide a way for contracts to check its
integrity. Contract authors could watch the behavior of their contracts over time for
malicious producer behavior, but they don't have a reliable way to restrict their contracts
from relying on data from bad producers. This EEP gives contracts this ability.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current EOSIO platforms.-->

This consensus upgrade adds this intrinsic:

```c++
name get_current_producer();
```

* When a transaction is speculatively executed, this returns the producer that
  is scheduled to produce at the current position.
* When a block is being produced, this returns the producer that is producing it.
* What a block is being validated, this returns the producer that produced the block.

## Copyright
