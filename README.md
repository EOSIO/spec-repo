# Welcome to the EOSIO Specifications Repository

![EOSIO Labs](https://img.shields.io/badge/EOSIO-Labs-5cb3ff.svg)

Scope: The  EOSIO Specifications Repository aims to provide high level design details
for many of the ideas discussed in the
[Strategic Vision Document](https://eos.io/strategic-vision/)

Goal: We aim to communicate our technical design choices and seek feedback from
the community on our approach. However, we may not be able to respond to every
element of feedback or synthesize all the feedback into a consensus approach.

Details: The software design docs are being released as part of EOSIO Labs.
The design docs are forward looking guesses when they are published on how we
might want to realize the feature/functionality needs. We make no guarantees
about implementing it as designed or even keeping the specs updated to match
the eventual design choices. Interested members of the community can engage
with us on the technical design details of how we are thinking about
implementing various pieces of functionality.

Repo mechanics and feedback: Each spec will be a separate markdown file
similar to [this one](esr_enhanced_token.md).
Community members can create issues to provide feedback on the various
specifications. Issues can be created using the provided issue template.

# About EOSIO Labs

EOSIO Labs repositories are experimental.  Developers in the community are 
encouraged to use EOSIO Labs repositories as the basis for code and concepts
to incorporate into their applications. Community members are also welcome
to contribute and further develop these repositories. Since these repositories
are not supported by Block.one, we may not provide responses to issue reports,
pull requests, updates to functionality, or other requests from the community.

# Documents

Key:
* `.` = Still being drafted
* `R` = Ready for feedback

In dependency order:
* `R` [Smart Contract Modules](esr_smart_contract_modules.md)
* `R` [Deprecate Deferred](esr_deprecate_deferred.md)
* `R` [Sized Data](esr_sized_data.md) (ABI 1.2)
* `R` [Tagged Data](esr_tagged_data.md)
* `.` [Flexible Notifications](esr_flexible_notify.md)
* `R` [Synchronous Calls](esr_synchronous_calls.md)
* `R` [Query Consumption](esr_query_consumption.md)
* `R` [Subjective Data](esr_subjective_data.md)
* `R` [Get Producer](esr_get_producer.md)
* `R` [Contract Pays](esr_contract_pays.md)
* `R` [Contract Authentication](esr_contract_trx_auth.md)
* `R` [Forwarding Authorizations](esr_contract_fwd_auth.md)
* `R` [Enhanced Token](esr_enhanced_token.md)
* `R` [Configurable WASM Limits](esr_configurable_wasm_limits.md)

Not in dependency order:
* `R` [Key-Value Database](esr_key_value_database.md)
  * `R` [Key-Value Database Intrinsics](esr_key_value_database_intrinsics.md)
* `R` [Regions](esr_regions.md)

Deprecations:
* `require_recipient`: [Flexible Notifications](esr_flexible_notify.md)
* Deferred Transactions: [Deprecate Deferred](esr_deprecate_deferred.md)

Practices to Consider (PTC):
* `.` [Alternative Uses for WebAuthN Signatures](ptc_webauthn_alt_use.md)

## Contributing

[Contributing Guide](./CONTRIBUTING.md)

[Code of Conduct](./CONTRIBUTING.md#conduct)

## Important

All material is provided subject to [this important notice](./IMPORTANT.md) and you must familiarize yourself with its terms. The notice contains important information, limitations and restrictions relating to our software, publications, trademarks, third-party resources and forward-looking statements. By accessing any of our material, you accept and agree to the terms of the notice.
