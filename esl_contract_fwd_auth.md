# Forwarding Authorizations Between Contracts

## Simple Summary

Contracts can attest user authentication to other contracts. This enables contract-defined accounts to use other contracts.

## Abstract

The [Contract Authentication](esl_contract_trx_auth.md) proposal allows contracts to define their own account
structures and authorize transactions from those accounts. It mentions that contracts may attest authorizations
to other contracts, but doesn't indicate how. This EEP proposes a protocol to support this, along with CDT enhancements
to simplify implementation.

## Specification

### Account Existence Check

The sender must define this action:

```c++
[[eosio::action("eosio.chkact")]] void check_account(checksum256 account);
```

This action asserts that `account` is valid. It should do nothing if the account
exists, or abort the transaction if the account doesn't exist. If this check
isn't possible, e.g. accounts on other chains, then this action should be
a no-op.

### CDT Support (sender)

`action_wrapper` will have a new constructor which accepts subaccount identities:

```c++
template<...>
struct action_wrapper {
    ...
    template<typename T>
    action_wrapper(name receiver, const std::vector<fixed_sized_data<T, 32>>& idents);
    ...
}
```

This lets the receiving contract know which identities this contract is attesting to
when executing the inline action. Example use:

```c++
// This is an example only. A future EEP will define a new token
// standard which may differ from this.

token::transfer2("eosio.token"_n, idents).send(from, to, amount, memo);
```

This doesn't forward any native authorities to the inline action; this prevents the
receiver from charging RAM to the sender.

The sender may use any type to represent identities, as long as it serializes into 32 bytes
or less. The `fixed_sized_data` wrapper enforces that limit and 0-pads any unused bytes. See
[Sized Data](esl_sized_data.md).

### CDT Support (receiver)

The CDT will add these new methods to the `contract` base class. These functions check
subaccount authorization:

```c++
struct subaccount {
    name        contract;
    checksum256 ident;
};

class contract {
    bool has_subaccount_auth(subaccount sub) const;
    void require_subaccount_auth(subaccount sub) const;
};
```

Contracts may use `has_subaccount_auth` and `require_subaccount_auth` to verify a subaccount's authority
is present similar to the way they currently use `has_auth` and `require_auth`.

The CDT will automatically dispatch actions which use the new authority system without any changes to the
contract source, assuming the contract is using the CDT's automatic dispatcher.

### Protocol (sender)

This pseudo-function describes the action in this protocol:

```c++
eosio.action(
    name                action,     // action name
    vector<checksum256> idents,     // identities
    bytes               payload     // action payload
);
```

The sender should send an inline action `eosio.action` with:
* The appropriate `action` and `payload`
* `idents` attesting to the authenticated account(s)

We recommend that senders avoid providing any native authorities to the inline action, even their own.
This will prevent the receiver from charging RAM to the sender.

### Protocol (receiver)

Contracts opt in to supporting subaccounts from other contracts by implementing the `eosio.action`
action. Contracts shouldn't implement this action manually; they should let the CDT handle this
task. The typical implementation:

* Uses `get_sender` to determine which contract is attesting to the identities and stores this value 
  somewhere for later use.
* Asserts that `get_sender` returned non-0.
* Stores `idents` somewhere for later use.
* Dispatches `payload` to the appropriate function chosen by `action`.

Each subaccount is identified by the tuple `(get_sender(), ident)`. Having `get_sender()`
in the tuple guarantees that contracts don't tread on each other's account space.
Actions may check whether a particular subaccount `(x, y)` authorized the action by verifying that `x`
matches the value returned by `get_sender` and `y` is in `idents`. This check replaces 
`require_auth` and `has_auth`.

The contract shouldn't use `require_auth` and `has_auth` when processing `eosio.action`.
The receiving contract must pay for any RAM it uses.

## Copyright
