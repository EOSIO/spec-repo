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
similar to [this one](esl_enhanced_token.md).
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

## Contributing

[Contributing Guide](./CONTRIBUTING.md)

[Code of Conduct](./CONTRIBUTING.md#conduct)

## License

[MIT](./LICENSE)

## Important

Disclaimer: Block.one makes its contribution on a voluntary basis as a member of the EOSIO community and is not responsible for ensuring the overall performance of the software or any related applications. We make no representation, warranty, guarantee or undertaking in respect of the releases described here, the related GitHub release, the EOSIO software or any related documentation, whether expressed or implied, including but not limited to the warranties or merchantability, fitness for a particular purpose and noninfringement.  In no event shall we be liable for any claim, damages or other liability, whether in an action of contract, tort or otherwise, arising from, out of or in connection with the software or documentation or the use or other dealings in the software or documentation.  Any test results or performance figures are indicative and will not reflect performance under all conditions.  Any reference to any third party or third-party product, resource or service is not an endorsement or recommendation by Block.one.  We are not responsible, and disclaim any and all responsibility and liability, for your use of or reliance on any of these resources. Third-party resources may be updated, changed or terminated at any time, so the information here may be out of date or inaccurate.  Any person using or offering this software in connection with providing software, goods or services to third parties shall advise such third parties of these license terms, disclaimers and exclusions of liability.  Block.one, EOSIO, EOSIO Labs, EOS, the heptahedron and associated logos are trademarks of Block.one. Other trademarks referenced herein are the property of their respective owners.  Please note that the statements herein are an expression of Block.one’s vision, not a guarantee of anything, and all aspects of it are subject to change in all respects at Block.one’s sole discretion. We call these “forward looking statements”, which includes statements in this document, other than statements of historical facts, such as statements regarding EOSIO’s development, expected performance, and future features, or our business strategy, plans, prospects, developments and objectives. These statements are only predictions and reflect Block.one’s current beliefs and expectations with respect to future events; they are based on assumptions and are subject to risk, uncertainties and change at any time.  We operate in a rapidly changing environment. New risks emerge from time to time. Given these risks and uncertainties, you are cautioned not to rely on these forward-looking statements. Actual results, performance or events may differ materially from what is predicted in the forward-looking statements. Some of the factors that could cause actual results, performance or events to differ materially from the forward-looking statements include, without limitation: market volatility; continued availability of capital, financing and personnel; product acceptance; the commercial success of any new products or technologies; competition; government regulation and laws; and general economic, market or business conditions.  All statements are valid only as of the date of first posting and Block.one is under no obligation to, and expressly disclaims any obligation to, update or alter any statements, whether as a result of new information, subsequent events or otherwise.  Nothing herein constitutes technological, financial, investment, legal or other advice, either in general or with regard to any particular situation or implementation. Please consult with experts in appropriate areas before implementing or utilizing anything contained in this document.
