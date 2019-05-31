Proposals for eventual inclusion in [EEPs](https://github.com/eoscanada/EEPs)

In dependency order:
* [Flexible Notifications](eep-draft_flexible_notify.md)
* [Subjective Data](eep-draft_subjective_data.md)
* [Contract Pays](eep-draft_contract_pays.md)
* [Contract Authentication](eep-draft_contract_trx_auth.md)
* [Forwarding Authorizations](eep-draft_contract_fwd_auth.md)
* [Enhanced Token](eep-draft_enhanced_token.md)

Todo:
* A new intrinsic `get_ram_usage` that returns the amount of RAM used by the receiver
* A new intrinsic `get_trx_net_bill` that returns the current NET charges for the transaction
    Need to record a commitment to the initial NET
* RAM market issues with subaccounts
  * Contracts need a callback from system contract for RAM market ops. Need delta_currency,
    delta_ram, request_id. CDT enhancements to aid tracking requests.
  * subaccounts may need a way to move their RAM between contracts without excessive fees.
    * alternative: contracts have their own RAM submarkets.
  * Contracts can use `get_ram_usage` to track subaccount RAM usage, but there's a really
    tricky part to account for: table charges.
