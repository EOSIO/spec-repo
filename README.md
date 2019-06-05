Proposals for eventual inclusion in [EEPs](https://github.com/eoscanada/EEPs)

* `.` = In progress
* `R` = Ready for review

In dependency order:
* `R` [Flexible Notifications](eep-draft_flexible_notify.md)
* `R` [Query Consumption](eep-draft_query_consumption.md)
* `.` [Subjective Data](eep-draft_subjective_data.md)
* `R` [Contract Pays](eep-draft_contract_pays.md)
* `R` [Contract Authentication](eep-draft_contract_trx_auth.md)
* `.` [Forwarding Authorizations](eep-draft_contract_fwd_auth.md)
* `.` [Enhanced Token](eep-draft_enhanced_token.md)

Discussion pieces:
* `.` [Enhanced Database](eep-draft_enhanced_database.md)

Deprecations:
* `require_recipient`: [Flexible Notifications](eep-draft_flexible_notify.md)
* Deferred Transactions: <link>

Todo:
* Read-only queries: contracts calling other contracts
  * Consistency issue:
    * ? Allow either caller or callee to force fresh VM
    * Potential issue with multi_index caching rows
* RAM market issues with subaccounts
  * Contracts need a signal from system contract for RAM market ops. Need delta_currency,
    delta_ram, request_id. CDT enhancements to aid tracking requests.
  * subaccounts may need a way to move their RAM between contracts without excessive fees.
    * alternative: contracts have their own RAM submarkets.
  * Contracts can use `get_ram_usage` to track subaccount RAM usage, but there's a really
    tricky part to account for: table charges.
* Undecided addition. CDT: `sized_content<T>`, `extendable_variant`. ABI: '#'.
* Deprecation of deferred transactions and potential replacements
  * Inline transactions
  * Deprecate wait weights (chestnut-like replacements based on subaccounts)
  * Discuss problems with deferred transactions
* Named shards / regions
  * Alternatives for communication. Identify pros and cons of each:
    * Communication protocol (e.g. maybe a message queue)
    * Ability to prove an inline action in another shard
  * BPs don't have to process every shard every block
