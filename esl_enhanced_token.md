# Enhanced Token

## Simple Summary

This discussion piece covers some of the ideas we have for a new token standard.
This idea may or may not be implemented.

## Abstract

`eosio.token` is the defacto token standard on eosio blockchains. There are many deployments of this contract on
different accounts handling a variety of tokens. There are also customized versions of this contract, and
alternative implementations of its interface.

We would like an updated token standard to include support for the following:

* [Subaccounts](esl_contract_fwd_auth.md). This will aid cross-chain and [named-region](esl_regions.md) use of
  tokens. It will also allow low-overhead contract-defined accounts to hold tokens.
* Support for `extended_asset` in the interface. Some contracts hold tokens issued by other contracts on behalf of
  their users. If the token interface uses `extended_asset` instead of `asset`, then these contracts can expose the same
  interface.
* A replacement for `memo` which supports ABI-defined binary data. This will allow users to attach structured data to actions
  that contracts can process in a standard way. It provides an alternative to the deposit-and-spend pattern without the downsides
  of parsing that existing contracts which avoid deposit-and-spend currently use.
* Ability for non-standard actions to adjust balances without causing wallets and block explorers to break. They will be able
  to notify affected accounts via [Flexible Notifications](esl_flexible_notify.md).
* No requirements on table structures. This will improve flexibility in developing custom token contracts. It will also allow
  token contracts to freely move to an [Alternative Database](esl_key_value_database.md) in the future.

We expect to cover balance query issues in a future ESL.

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
  [Forwarding Authorizations](esl_contract_fwd_auth.md) specification.

Even though a token contract needs to define all three options in its ABI to be compatible,
it doesn't need to support all three.
* If it doesn't support native accounts, then it may assert when native accounts are used.
* If it doesn't need local accounts, then it may typedef `local_account_type` to `checksum256`
  and assert when they are used.
* If it doesn't support foreign accounts, then it may assert when they are used.

If a foreign account's contract field matches the token contract, then the token contract
should treat it like a local account.

`token_account` has a size prefix to aid future extensions of `token_account_variant`.
Extensions to `token_account_variant` are reserved for future ESLs.

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
    token_account               authorizer,
    token_account               issuer,
    asset                       maximum_supply,
    string                      memo,
    optional<eosio_scoped_data> data);

void issue2(
    token_account               authorizer,
    asset                       quantity,
    string                      memo,
    optional<eosio_scoped_data> data);

void retire2(
    token_account               authorizer,
    asset                       quantity,
    string                      memo,
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
    account         authorizer,
    extended_asset  max_fee,
    account         owner,
    extended_symbol symbol);

void close2(
    account         authorizer,
    account         owner,
    extended_symbol symbol);
```

Token contracts choose their own policies about who may open and close accounts.
These actions may not be appropriate for some token contracts; those contracts
may implement the actions as no-ops. 

Token contracts which implement `open2` should take reasonable precautions against
opening non-existent accounts, e.g. by using `is_account` to check native accounts
and the `eosio.chkact` action
([Forwarding Authorizations](esl_contract_fwd_auth.md)) to check foreign accounts.
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
    account                     authorizer,
    account                     from,
    account                     to,
    extended_asset              quantity,
    string                      memo,
    optional<eosio_scoped_data> data);
```

Token contracts choose their own policies about who may transfer tokens. Here is a typical policy:

* `authorizer` must be `from`.
* `to` must already have an open account that can hold the tokens.

### Notifications

There is a single event type that off-chain processes can watch for:

```c++
[[eosio::event("e.balchng")]] void balance_changed_event(
    token_account account, extended_asset delta, int64_t new_bal, eosio_tagged_data data);
```

This indicates `account`'s balance changed by `delta` and has `new_bal`. It includes
additional `data` about the action; see below. Token contracts send this event whenever
they modify an account's balance.

There's a single signal that contracts can listen to for notifications:

```c++
[[eosio::signal("e.balchng")]] void balance_changed_signal(
    name receiver, token_account account, extended_asset delta, int64_t new_bal, eosio_tagged_data data);
```

Token contracts choose their own policies about when to send signals. Here is the recommended policy:
* Notify the "to" account on transfers. If "to" is a foreign account, then notify the contract that
  authorizes for it.

## Notification Data

Here are definitions for the `data` field for notifications about the actions in this spec. Token
contracts may define additional structs for the `data` field for notifications about other actions.
The `e.` prefix is reserved for future specifications; custom structs should not use that prefix.

```c++
struct create_data: tagged_base<"e.create.dat"_n> {
    token_account               issuer;
    asset                       maximum_supply;
    string                      memo;
    optional<eosio_scoped_data> data;
};

struct issue_data: tagged_base<"e.issue.dat"_n> {
    asset                       quantity;
    string                      memo;
    optional<eosio_scoped_data> data;
};

struct retire_data: tagged_base<"e.retire.dat"_n> {
    asset                       quantity;
    string                      memo;
    optional<eosio_scoped_data> data;
};

struct transfer_data: tagged_base<"e.xfer.dat"_n> {
    account                     from;
    account                     to;
    extended_asset              quantity;
    string                      memo;
    optional<eosio_scoped_data> data;
};
```

## Todo

Reserve binary_extensions of actions and signals

Require rejection of unknown actions (new cdt feature)

Sync functions

## Backwards Compatibility

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

## Copyright
