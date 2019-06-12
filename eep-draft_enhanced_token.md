---
eep: <to be assigned>
title: Enhanced Token
author: <a list of the author's or authors' name(s) and/or username(s), or name(s) and email(s), e.g. (use with the parentheses or triangular brackets): FirstName LastName (@GitHubUsername), FirstName LastName <foo@bar.com>, FirstName (@GitHubUsername) and GitHubUsername (@GitHubUsername)>
discussions-to: <URL>
status: Draft
type: <Standards Track (Core, Networking, Interface)  | RFC | Meta>
category (*only required for Standard Track): <Core | Networking | Interface>
created: <date created on, in ISO 8601 (yyyy-mm-dd) format>
requires (*optional): <EEP number(s)>
replaces (*optional): <EEP number(s)>
---

<!--You can leave these HTML comments in your merged EEP and delete the visible duplicate text guides, they will not appear and may be helpful to refer to if you edit it again. This is the suggested template for new EEPs. Note that an EEP number will be assigned by an editor. When opening a pull request to submit your EEP, please use an abbreviated title in the filename, `eep-draft_title_abbrev.md`. The title should be 44 characters or less.-->

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the EEP.-->

This discussion piece covers some of the ideas we have for a new token standard.
This idea may or may not be implemented.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

`eosio.token` is the defacto token standard on eosio blockchains. There are many deployments of this contract on
different accounts handling a variety of tokens. There are also customized versions of this contract, and
alternative implementations of its interface.

We would like an updated token standard to include support for the following:

* [Subaccounts](eep-draft_contract_fwd_auth.md). This will aid cross-chain and [named-region](eep-draft_regions.md) use of
  tokens. It will also allow low-overhead contract-defined accounts to hold tokens.
* Support for `extended_asset` in the interface. Some contracts hold tokens issued by other contracts on behalf of
  their users. If the token interface uses `extended_asset` instead of `asset`, then these contracts can expose the same
  interface.
* A replacement for `memo` which supports ABI-defined binary data. This will allow users to attach structured data to actions
  that contracts can process in a standard way. It provides an alternative to the deposit-and-spend pattern without the downsides
  of parsing that existing contracts which avoid deposit-and-spend currently use.
* Ability for non-standard actions to adjust balances without causing wallets and block explorers to break. They will be able
  to notify affected accounts via [Flexible Notifications](eep-draft_flexible_notify.md).
* No requirements on table structures. This will improve flexibility in developing custom token contracts. It will also allow
  token contracts to freely move to [Enhanced Database Support](eep-draft_enhanced_database.md) in the future.

We expect to cover balance query issues in a future EEP.

## Specification

### Accounts

`token_account` represents accounts in actions and in notifications:

```c++
struct native_account {
    name account;
};

struct local_account {
    fixed_sized_data<local_account_type, 32> account;
};

struct foreign_account {
    name        contract;
    checksum256 ident;
};

using token_account_variant = variant<
    native_account, local_account, foreign_account>;

using token_account = sized_data<token_account_variant>;
```

`token_account` supports 3 different types of accounts:

* Native accounts are the original type of eosio account. `has_auth` and `require_auth`
  checks the authentication of these.
* Local accounts are defined by the specific token contract handling the request. The
  token contract:
  * Defines what `local_account_type` is and includes its definition in its ABI; this
    may differ between token contracts.
  * Defines how local types are authenticated.
* Foreign accounts are accounts defined by other contracts. They handle the
  [Forwarding Authorizations](eep-draft_contract_fwd_auth.md) specification.

Even though a token contract needs to define all three options in its ABI to be compatible,
it doesn't need to support all three.
* If it doesn't support native accounts, then it may assert when native accounts are used.
* If it doesn't need local accounts, then it may typedef `local_account_type` to `checksum256`
  and assert when they are used.
* If it doesn't support foreign accounts, then it may assert when they are used.

If a foreign account's contract field matches the token contract, then the token contract
should treat it like a local account.

`token_account` has a size prefix to aid future extensions. These extensions are reserved
for future EEPs.

### Memo replacement

Contracts need a way to pass structured data through token actions. Actions and
notifications in this token standard include the `data` field:

```c++
optional<eosio_scoped_data> data
```

`data` contains structured data for contracts and off-chain apps to process, e.g. purchase
details or an exchange account name. `memo` contains free-form text; contracts and off-chain
applications shouldn't expect it to contain meaningful data.

### Token Lifetime Actions (local)

These optional actions manage lifetime of local tokens. Local tokens are ones that originate
in the current contract.

```c++
void create2(
    account authorizer,
    account issuer,
    asset maximum_supply,
    string memo,
    optional<eosio_scoped_data> data);

void issue2(
    account authorizer,
    asset quantity,
    string memo,
    optional<eosio_scoped_data> data);

void retire2(
    account authorizer,
    asset quantity,
    string memo,
    optional<eosio_scoped_data> data);
```

Token contracts choose their own policies about who may create, issue, and retire tokens. Here
is a typical policy:

* The token contract owner authorizes `create2`. Alternatively, the winner of a symbol auction
  may authorize it.
* Only the issuer may authorize `issue2`. The newly-issued tokens go to the authorizer.
* Only the issuer may authorize `retire2`. The tokens are burned from the authorizer's account.

### Token Lifetime Actions (foreign)

This optional action registers a foreign token. A foreign token is a token which
is managed by another contract.

```c++
void regforeign(account authorizer, extended_symbol sym, unsigned_int version);
```

`version` indicates which protocol the other contract uses. This must be `0` for the previous
token standard or `1` for this one.

### Token Account Actions

These actions manage account lifetimes:

```c++
void open2(
    account authorizer,
    extended_asset max_fee,
    account owner,
    extended_symbol symbol);

void close2(
    account authorizer,
    account owner,
    eosio::extended_symbol symbol);
```

Token contracts choose their own policies about who may open and close accounts.
These actions may not be appropriate for some token contracts; those contracts
may implement the actions as no-ops. 

Token contracts which implement `open2` should take reasonable precautions against
opening non-existent accounts, e.g. by using `is_account` to check native accounts
and the `eosio.chkact` action
([Forwarding Authorizations](eep-draft_contract_fwd_auth.md)) to check foreign accounts.
`open2` may charge a fee up to `max_fee` to the authorizer to cover the costs of
opening the account.

Here is a typical policy:

* Any authorizer may open an account for another user. Fees are deducted from the
  authorizer's account, up to `max_fee`.
* Only the account owner may authorize closing the account.

### Token Transfer Action

This action transfers tokens:

```c++
void transfer2(
    account authorizer,
    account from,
    account to,
    extended_asset quantity,
    string memo,
    optional<eosio_scoped_data> data);
```

Token contracts choose their own policies about who may transfer tokens. Here is a typical policy:

* `authorizer` must be `from`.
* `to` must already have an open account that can hold the tokens.

### Notifications

Token contracts choose their own policies about when to send signals; they always send events.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current EOSIO platforms.-->

Consider having accounts opt-in to receiving notifications to save resources. Could cause problems for wallets and block explorers.

Reserve variant additions

Reserve binary_extensions of actions and signals

eep value of 0 to support app-specific

require rejection of unknown actions (new cdt feature)

sync functions

```c++
using account = variant<name, subaccount>;

struct create_data {
    string  memo;
};

struct issue_data {
};

struct transfer_data {
    account from;
    account to;
    string  memo;
};

template<auto eep, name::raw reason>
struct eep_reason {
    // returns array<char, ?>
    static constexpr auto abi_name() {
    }

    static constexpr int get_eep() {return eep;}
    static constexpr name get_reason() {return reason;}
};


[[eosio::include_in_abi]]
struct not_icky: eep_reason<34, "foo"_n> {
    // static constexpr int eep = 34;
    // static constexpr name reason = "foo";
    // static constexpr auto abi_name = f(epp, reason);


    name from;
    name to;
};

...
T::get_eep()

struct custom_data {
    unsigned_int    eep_number;
    name            reason;         // identifies struct in sender ABI: data_<eep>_<reason>
    bytes           payload;
};


struct transfer_in_data: transfer_data {};
struct transfer_out_data: transfer_data {};

// using balchg_data = extendable_variant<
//     create_data,
//     transfer_in_data,
//     transfer_out_data,
//     issue_data,
//     custom_data,
//     >;

using balchg_data = eep_variant<
    create_data,
    transfer_in_data,
    issue_data,
>;

[[eosio::signal]] void eosio.balchg(
    name receiver,
    account acc, extended_asset delta, int64_t new_bal, balchg_data data);
```

## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

## Backwards Compatibility
<!--All EEPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The EEP must explain how the author proposes to deal with these incompatibilities. EEP submissions without a sufficient backwards compatibility treatise may be rejected outright.-->

There doesn't appear to be a way to upgrade existing token contracts to a new standard without breaking the world:
* There isn't a safe and efficient way for an updated token contract to notify older contracts about transfers
  to them:
  * The existing notification can't represent subaccounts.
  * The existing notification can't reference tokens in another token contract's namespace
    (extended_asset support).
  * The existing notification can only be sent from the original `transfer` action, not any other actions.
* Wallets, block explorers, and even some non-token contracts make assumptions about the table structures
  of existing tokens. If a token contract migrated its table structures to support new features, the existing
  users would break.

Instead, we propose that:
* Only new token contracts implement a new token protocol
* New token contracts can hold tokens from the older standard on behalf of their users. This will allow existing
  tokens to flow through the new protocol.

## Test Cases
<!--Test cases for an implementation are mandatory for EEPs that are affecting consensus changes. Other EEPs can choose to include links to test cases if applicable.-->

## Implementation
<!--The implementations must be completed before any EEP is given status "Final", but it need not be completed before the EEP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->

## Copyright
