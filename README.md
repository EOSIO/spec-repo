Proposals for eventual inclusion in [EEPs](https://github.com/eoscanada/EEPs)

* `.` = In progress
* `R` = Ready for review

In dependency order:
* `R` [Deprecate Deferred](eep-draft_deprecate_deferred.md)
* `R` [Sized Data](eep-draft_sized_data.md)
* `R` [Flexible Notifications](eep-draft_flexible_notify.md)
* `.` [Synchronous Calls](eep-draft_synchronous_calls.md)
* `R` [Query Consumption](eep-draft_query_consumption.md)
* `R` [Subjective Data](eep-draft_subjective_data.md)
* `R` [Get Producer](eep-draft_get_producer.md)
* `R` [Contract Pays](eep-draft_contract_pays.md)
* `R` [Contract Authentication](eep-draft_contract_trx_auth.md)
* `R` [Forwarding Authorizations](eep-draft_contract_fwd_auth.md)
* `.` [Enhanced Token](eep-draft_enhanced_token.md)

Not in dependency order:
* `R` [Enhanced Database](eep-draft_enhanced_database.md)
* `R` [Regions](eep-draft_regions.md)

Deprecations:
* `require_recipient`: [Flexible Notifications](eep-draft_flexible_notify.md)
* Deferred Transactions: [Deprecate Deferred](eep-draft_deprecate_deferred.md)

Todo:
* Switch proposals to use `#`. Check if `extendable_variant` still needed.
* RAM market issues with subaccounts
  * Contracts need a signal from system contract for RAM market ops. Need delta_currency,
    delta_ram, request_id. CDT enhancements to aid tracking requests.
  * subaccounts may need a way to move their RAM between contracts without excessive fees.
    * alternative: contracts have their own RAM submarkets.
  * Contracts can use `get_ram_usage` to track subaccount RAM usage, but there's a really
    tricky part to account for: table charges.
* Undecided addition: `extendable_variant`.
