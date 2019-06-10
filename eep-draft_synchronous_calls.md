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

### CDT Support (callee)

### Intrinsics (caller)

Callers use these intrinsics to make the call and get the result. Callers shouldn't use these directly;
they should let the CDT handle this task.

```c++
size_t call_sync(
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

`call_sync` calls into another contract. It aborts the transaction if the callee doesn't have
a synchronous call entry point, if contract is already in the call stack, if the result from a
previous `call_sync` call hasn't been fetched using `get_sync_result`, or if the current
transaction is deferred. It returns the size of the result.

`get_sync_result` returns the result of the previous asynchronous call. It aborts if `call_sync`
hasn't been called, or if `get_sync_result` has already been called.

### Intrinsics and Entry (callee)

Contracts opt in to being called by implementing the following entry point. Contracts shouldn't
implement this directly; they should let the CDT handle this task.

```c++
void handle_sync(
    name        function,
    size_t      args_size
);
```

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
then use `return_sync` to return the result. The transaction aborts if `return_sync` isn't called or
if the contract uses any state-modifying intrinsics (e.g. database modification). The transaction also
aborts if the contract calls `return_sync` while it's not handling a synchronous call. `return_sync`
stops execution of the callee.

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
