---
eep: <to be assigned>
title: Enhanced Database
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

This discussion piece covers one of the ideas we have for potential database enhancements. This may
or may not be implemented.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

## Motivation
<!--The motivation is critical for EEPs that want to change the EOSIO protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the eep solves. EEP submissions without sufficient motivation may be rejected outright.-->

Nodeos offers contracts a flexible database structure, but we see some areas for potential improvement:
* Primary keys are always 64 bits. Contracts which need larger primary keys can emulate them by combining
  primary keys with scopes, or by using secondary indexes. These approach works in some cases, but not others.
* Secondary indexes are limited to a fixed set: uint64_t, uint128_t, float64_t, float128_t, and checksum256. Contracts
  which need more than 256 bits in a secondary key can emulate them by combining the secondary key with scope
  or by hashing the secondary key, but these create their own set of problems since scopes aren't iterable and hashes
  don't preserve order. In addition, checksum256's implementation includes an optimization that affects sort order.
  This optimization leaked through to contracts and to the RPC API in non-obvious ways.
* The scope concept can confuse developers. It can help organize data, but it can't be iterated. It can act like an
  extension to the primary key, but it also affects secondary keys. It can help organize RAM charges if a contract
  enforces scope=payer, but contributes to non-obvious RAM-charging behavior in other cases. Scope is always present
  even when contract authors don't desire it.
* The database API gives contracts the ability to charge RAM to users without granting users the ability to force
  contracts to release their RAM. Developers who intend to return RAM to users sometimes find they can't because
  of the way table billing, separate from row billing, works. All this gives users a disincentive to try new contracts.
* RAM billing for tables, separate from rows, is not obvious to contract developers. Many don't even know it's there.
* Both nodeos and the CDT's multi_index have considerable overhead to maintain C++'s iterator abstraction. This overhead
  exists even when contracts don't use multi_index's iterators directly.
* The current RAM billing model is based on an implementation detail: well-behaved nodes use physical RAM to back
  contracts' database rows. This partly explains the current RAM prices.
* Contracts can freely read other contracts' rows, creating compatibility headaches during upgrades.
* Contract developers struggle to write code which wipes tables correctly when schemas change, especially when secondary
  indexes are present or when scope values aren't fixed.

## Ideas
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current EOSIO platforms.-->

### Key-value store with arbitrary-sized keys

Consider a set of primitives that look like this. These definitions use C++ types for ease of discussion; the
low-level primitives would be different. There may be performance gains by supporting a different iteration model
than the functions below imply; we can consider that later.

```c++
// Add (key,value) to database. Overwrites existing entry, if any.
void db_set_kv(bytes key, bytes value);

// Remove key from database, if it exists
void db_remove(bytes key);

// Get value for a specific key, if it exists
optional<bytes> db_get_v(bytes key);

// Returns a key. End is represented by an empty optional.
optional<bytes> db_lower_bound(bytes key);

// Returns a key. End is represented by an empty optional.
optional<bytes> db_upper_bound(bytes key);

// Returns a key. End is represented by an empty optional.
optional<bytes> db_next_key(bytes key);

// Returns a key. End is represented by an empty optional.
optional<bytes> db_prev_key(bytes key);
```

Some potential ideas we could apply to the above:
* Only charge RAM to the contract, not to users.
* Maybe allow this model to go to SSD. In this case, there could be a new resource type to charge,
  instead of RAM.
* Prevent contracts from reading other contracts' data
* No scopes or tables at this level of abstraction

This model gives contracts the ability to build higher-level abstractions on top:
* A table model which supports
  * any-size or even variable-size primary keys
  * any-size or even variable-size secondary keys
  * new types for keys (e.g. strings)
* A multi_index compatibility layer
* A file-system like abstraction

### Table abstraction

Contracts can build abstractions on top of a key-value store by partitioning the key space. Suppose a
contract wants a table model. The key could contain, in order:

* 0x01 to indicate a table
* 8 bytes to indicate table name (big-endian)
* 8 bytes to indicate index name (big-endian). 0 for primary index
* transformed key (below)

In the primary index, the value could contain the table data. In secondary indexes, the value could contain the
primary key. If there's multiple secondary indexes, then it's probably in the contract's interest to choose a
small (e.g. uint32_t or uint64_t) primary key.

The CDT could provide this table abstraction layer to prevent contract authors from having to implement it.
It could provide both a full table model and a simplified STL-like container which only supports a single key.

### Key transformations

The key-value store could provide a lexicographical ordering of uint8_t on the keys. The contract can
create an ordering on top by transforming its keys. Example transforms:

* uint?_t: Convert to big-endian
* int?_t: Invert the MSB then convert to big-endian
* strings: Convert 0x00 to (0x00, 0x01). Append (0x00, 0x00) to the end. This transform allows arbitrary-length strings.
* case-insensitive strings: Convert to upper-case, then apply the above transform. Assumes ASCII.
* floating-point:
  * There's some bit manipulations, followed by an endian conversion
  * limitations:
    * Positive 0 and Negative 0 map to the same value
    * NaN's and inf's end up with an unusual ordering
* struct or tuple: transform each field in order. Concatenate results.

The CDT would provide functions to handle this conversion.

## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

## Backwards Compatibility
<!--All EEPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The EEP must explain how the author proposes to deal with these incompatibilities. EEP submissions without a sufficient backwards compatibility treatise may be rejected outright.-->

## Test Cases
<!--Test cases for an implementation are mandatory for EEPs that are affecting consensus changes. Other EEPs can choose to include links to test cases if applicable.-->

## Implementation
<!--The implementations must be completed before any EEP is given status "Final", but it need not be completed before the EEP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->

## Copyright
