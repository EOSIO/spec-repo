---
eep: <to be assigned>
title: Synchronous Calls
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

Allow contracts to synchronously call into each other (read-only)

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

## Motivation
<!--The motivation is critical for EEPs that want to change the EOSIO protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the eep solves. EEP submissions without sufficient motivation may be rejected outright.-->

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current EOSIO platforms.-->

### CDT Support (caller)

To declare a synchronous function, declare a function with an `eosio::sync_ro_call`
attribute. The first function argument is the contract to call. The return type allows the
caller to return values to the callee. The CDT will generate the function's body. e.g.:

```c++
// These are examples only. A future EEP will define a new token
// standard which differs from this.

[[eosio::sync_ro_call("get.balance")]] asset get_balance(
    name contract, name account, symbol sym);
```

The function name must follow the rules for eosio names. If the function has a non-compliant
name, then pass a corrected name to the attribute's argument.

To use the synchronous function, call it.

```c++
auto balance = get_balance("eosio.token"_n, account, symbol("SYS", 4));
```

### CDT Support (callee)

To define a synchronous function which can be called by other contracts, define a member
function on a contract with the `eosio::sync_ro_func` attribute. The attribute has a string
argument which specifies the function name. The function's first argument is the caller.
Synchronous functions may not modify system or database state.

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
};
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

`handle_sync` should fetch the arguments using `get_sync_args`, dispatch it to the appropriate function,
then use `return_sync` to return the result. `handle_sync` should assert that `function` is known. The
transaction aborts if `return_sync` isn't called or if the contract uses any state-modifying intrinsics
(e.g. database modification). The transaction also aborts if the contract calls `return_sync` while it's
not handling a synchronous call. `return_sync` stops execution of the callee.

A new consensus parameter will limit the maximum nesting depth of `handle_sync`.

### ABI

This EEP does not define ABI support for synchronous calls. A future EEP may address this.

## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

## Backwards Compatibility
<!--All EEPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The EEP must explain how the author proposes to deal with these incompatibilities. EEP submissions without a sufficient backwards compatibility treatise may be rejected outright.-->

## Test Cases
<!--Test cases for an implementation are mandatory for EEPs that are affecting consensus changes. Other EEPs can choose to include links to test cases if applicable.-->

## Implementation
<!--The implementations must be completed before any EEP is given status "Final", but it need not be completed before the EEP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->

## Copyright
