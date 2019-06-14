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
similar to [this one](esl_enhanced_token.md).
Community members can create issues to provide feedback on the various
specifications. Issues can be created using the provided issue template.

# Documents

Key:
* `.` = Still being drafted
* `R` = Ready for feedback

In dependency order:
* `R` [Deprecate Deferred](esl_deprecate_deferred.md)
* `R` [Sized Data](esl_sized_data.md) (ABI 1.2)
* `R` [Tagged Data](esl_tagged_data.md)
* `.` [Flexible Notifications](esl_flexible_notify.md)
* `R` [Synchronous Calls](esl_synchronous_calls.md)
* `R` [Query Consumption](esl_query_consumption.md)
* `R` [Subjective Data](esl_subjective_data.md)
* `R` [Get Producer](esl_get_producer.md)
* `R` [Contract Pays](esl_contract_pays.md)
* `R` [Contract Authentication](esl_contract_trx_auth.md)
* `R` [Forwarding Authorizations](esl_contract_fwd_auth.md)
* `.` [Enhanced Token](esl_enhanced_token.md)

Not in dependency order:
* `R` [Key-Value Database](esl_key_value_database.md)
* `R` [Regions](esl_regions.md)

Deprecations:
* `require_recipient`: [Flexible Notifications](esl_flexible_notify.md)
* Deferred Transactions: [Deprecate Deferred](esl_deprecate_deferred.md)
