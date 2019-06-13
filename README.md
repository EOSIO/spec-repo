# EOSIO Specs Labs

Proposals for eventual inclusion in [EEPs](https://github.com/eoscanada/EEPs)

* `.` = In progress
* `R` = Ready for review

In dependency order:
* `R` [Deprecate Deferred](eep-draft_deprecate_deferred.md)
* `R` [Sized Data](eep-draft_sized_data.md) (ABI 1.2)
* `.` [Scoped Data](eep-draft_scoped_data.md)
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
* Use `eosio_scoped_data`
* RAM market issues with subaccounts
  * Contracts need a signal from system contract for RAM market ops. Need delta_currency,
    delta_ram, request_id. CDT enhancements to aid tracking requests.
  * subaccounts may need a way to move their RAM between contracts without excessive fees.
    * alternative: contracts have their own RAM submarkets.
  * Contracts can use `get_ram_usage` to track subaccount RAM usage, but there's a really
    tricky part to account for: table charges.

## Contributing

[Contributing Guide](./CONTRIBUTING.md)

[Code of Conduct](./CONTRIBUTING.md#conduct)

## License

[MIT](./LICENSE)

## Important

Disclaimer: Block.one makes its contribution on a voluntary basis as a member of the EOSIO community and is not responsible for ensuring the overall performance of the software or any related applications. We make no representation, warranty, guarantee or undertaking in respect of the releases described here, the related GitHub release, the EOSIO software or any related documentation, whether expressed or implied, including but not limited to the warranties or merchantability, fitness for a particular purpose and noninfringement.  In no event shall we be liable for any claim, damages or other liability, whether in an action of contract, tort or otherwise, arising from, out of or in connection with the software or documentation or the use or other dealings in the software or documentation.  Any test results or performance figures are indicative and will not reflect performance under all conditions.  Any reference to any third party or third-party product, resource or service is not an endorsement or recommendation by Block.one.  We are not responsible, and disclaim any and all responsibility and liability, for your use of or reliance on any of these resources. Third-party resources may be updated, changed or terminated at any time, so the information here may be out of date or inaccurate.  Any person using or offering this software in connection with providing software, goods or services to third parties shall advise such third parties of these license terms, disclaimers and exclusions of liability.  Block.one, EOSIO, EOSIO Labs, EOS, the heptahedron and associated logos are trademarks of Block.one. Other trademarks referenced herein are the property of their respective owners.  Please note that the statements herein are an expression of Block.one’s vision, not a guarantee of anything, and all aspects of it are subject to change in all respects at Block.one’s sole discretion. We call these “forward looking statements”, which includes statements in this document, other than statements of historical facts, such as statements regarding EOSIO’s development, expected performance, and future features, or our business strategy, plans, prospects, developments and objectives. These statements are only predictions and reflect Block.one’s current beliefs and expectations with respect to future events; they are based on assumptions and are subject to risk, uncertainties and change at any time.  We operate in a rapidly changing environment. New risks emerge from time to time. Given these risks and uncertainties, you are cautioned not to rely on these forward-looking statements. Actual results, performance or events may differ materially from what is predicted in the forward-looking statements. Some of the factors that could cause actual results, performance or events to differ materially from the forward-looking statements include, without limitation: market volatility; continued availability of capital, financing and personnel; product acceptance; the commercial success of any new products or technologies; competition; government regulation and laws; and general economic, market or business conditions.  All statements are valid only as of the date of first posting and Block.one is under no obligation to, and expressly disclaims any obligation to, update or alter any statements, whether as a result of new information, subsequent events or otherwise.  Nothing herein constitutes technological, financial, investment, legal or other advice, either in general or with regard to any particular situation or implementation. Please consult with experts in appropriate areas before implementing or utilizing anything contained in this document.
