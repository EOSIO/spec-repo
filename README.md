# Welcome to the EOSIO Specifications Lab

Scope: The  EOSIO Specifications Lab aims to provide high level design details
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
similar to [this one](https://github.com/EOSIO/eep-proposal-staging/blob/d79edc748696daa8b9bc8cacc5281c144cbd4ec9/eep-draft_enhanced_token.md).
Community members can create issues to provide feedback on the various
specifications. Issues can be created using the provided issue template.

# Documents

Key:
* `.` = Still being drafted
* `R` = Ready for feedback

In dependency order:
* `R` [Deprecate Deferred](eep-draft_deprecate_deferred.md)
* `R` [Sized Data](eep-draft_sized_data.md) (ABI 1.2)
* `R` [Tagged Data](eep-draft_tagged_data.md)
* `.` [Flexible Notifications](eep-draft_flexible_notify.md)
* `R` [Synchronous Calls](eep-draft_synchronous_calls.md)
* `R` [Query Consumption](eep-draft_query_consumption.md)
* `R` [Subjective Data](eep-draft_subjective_data.md)
* `R` [Get Producer](eep-draft_get_producer.md)
* `R` [Contract Pays](eep-draft_contract_pays.md)
* `R` [Contract Authentication](eep-draft_contract_trx_auth.md)
* `R` [Forwarding Authorizations](eep-draft_contract_fwd_auth.md)
* `.` [Enhanced Token](eep-draft_enhanced_token.md)

Not in dependency order:
* `R` [Key-Value Database](eep-draft_key_value_database.md)
* `R` [Regions](eep-draft_regions.md)

Deprecations:
* `require_recipient`: [Flexible Notifications](eep-draft_flexible_notify.md)
* Deferred Transactions: [Deprecate Deferred](eep-draft_deprecate_deferred.md)
