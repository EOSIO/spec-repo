# Synchronous Calls

## Simple Summary

Allow contracts to synchronously call into each other (read-only)

## Abstract

## Specification

### CDT Support (callee)

To define a synchronous function which can be called by other contracts, define a member
function on a contract with the `eosio::sync_ro_func` attribute. The attribute has a string
argument which specifies the function name. The function's first argument is the caller.
To aid callers, also define a wrapper. Synchronous functions may not modify system or
database state.

```c++
// This is an example only. A future EEP will define a new token
// standard which differs from this.

class [[eosio::contract]] token: public contract {
  public:
    [[eosio::sync_ro_func("get.balance")]] asset get_balance(
        name caller, name account, symbol sym)
    ) {
        ...
        return amount;
    }
    using get_balance_ro =
        eosio::readonly_wrapper<"get.balance"_n, &token::get_balance>;
};
```

### CDT Support (caller)

To call a synchronous function, include the callee's header and use its `readonly_wrapper`:

```c++
auto balance = token::get_balance_ro("eosio.token"_n).call(account, symbol("SYS", 4));
```

### Intrinsics (caller)

Callers use these intrinsics to make the call and get the result. Callers shouldn't use these directly;
they should let the CDT handle this task.

```c++
size_t call_sync_readonly(
    name        contract,
    name        function,
    const char* args,
    size_t      args_size
);

void get_sync_result(
    char*       result,
    size_t      result_size
);
```

`call_sync_readonly` calls into another contract. It aborts the transaction if the callee doesn't have
a synchronous call entry point, if contract is already in the call stack, if the result from a
previous `call_sync_readonly` call hasn't been fetched using `get_sync_result`, or if the current
transaction is deferred. It returns the size of the result.

`get_sync_result` returns the result of the previous synchronous call. It aborts if `call_sync_readonly`
hasn't been called, or if `get_sync_result` has already been called.

A new consensus parameter will limit the maximum nesting depth of `call_sync_readonly`.

### Intrinsics and Entry (callee)

Contracts opt in to being called by implementing the following entry point. Contracts shouldn't
implement this directly; they should let the CDT handle this task.

```c++
extern "C" void handle_sync_readonly(
    name        caller,
    name        function,
    size_t      args_size
);
```

The system initializes the WASM linear memory before calling this entry point. It does this
for each call.

The callee has the following intrinsics available:

```c++
void get_sync_args(
    char*       args,
    size_t      args_size
);

[[noreturn]] void return_sync(
    const char* result,
    size_t      result_size
);
```

`handle_sync_readonly` should fetch the arguments using `get_sync_args`, dispatch it to the appropriate function,
then use `return_sync` to return the result. `handle_sync_readonly` should assert that `function` is known. The
transaction aborts if `return_sync` isn't called or if the contract uses any state-modifying intrinsics
(e.g. database modification). The transaction also aborts if the contract calls `return_sync` while it's
not handling a synchronous call. `return_sync` stops execution of the callee.

### ABI

This EEP does not define ABI support for synchronous calls. A future EEP may address this.

## Copyright
