# Synchronous Calls

## Simple Summary

Allow contracts to synchronously call into each other (read-only)

## Abstract

Contracts currently communicate by sending inline actions and by reading each others'
tables. Inline actions are asynchronous; if contract A sends an inline action to
contract B, then B will receive it after A terminates. This complicates two-way
communications, especially for queries. Contracts may work around this by reading
each others' tables. This creates a compatibility problem; if a contract updates
its table format, then the existing contracts which rely on it will break.
This proposal defines a mechanism that allows contracts to synchronously call into
each other to for read-only queries. It doesn't support mutating operations;
we may address that in the future.

## Specification

### CDT Support (callee)

To define a synchronous function which can be called by other contracts, define a member
function on a contract with the `eosio::sync_ro_func` attribute. The attribute has a string
argument which specifies the function name. The function may get the calling contract using
the contract's `get_caller` member function. To aid callers, also define a wrapper.
Synchronous functions may not modify system or database state.

```c++
// This is an example only. A future ESR will define a new token
// standard which differs from this.

class [[eosio::contract]] token: public contract {
  public:
    [[eosio::sync_ro_func("get.balance")]] asset get_balance(
        name account, symbol sym)
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
bool call_sync_readonly(
    name        contract,
    name        function,
    const char* args,
    size_t      args_size,
    size_t*     result_size
);

void get_sync_result(
    char*       result,
    size_t      result_size
);
```

`call_sync_readonly` calls into another contract. It returns false if a maximum call depth was
reached, if the callee doesn't have a synchronous call entry point, if the callee didn't call
`return_sync`, or if the callee did anything which would have aborted the transaction. If
it returns true, then it fills `result_size`.

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
synchronous call aborts if `return_sync` isn't called or if the contract uses any state-modifying intrinsics
(e.g. database modification). The transaction aborts if the contract calls `return_sync` while it's
not handling a synchronous call. `return_sync` stops execution of the callee.

### ABI

This ESR does not define ABI support for synchronous calls. A future ESR may address this.
