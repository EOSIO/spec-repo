---
eep: Tagged Data
title: <EEP title>
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

Contracts may tag serialized data with custom types to identify it and aid decoding.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

Contracts may pack arbitrary data in the `bytes` type when communicating with each other
and with the outside, but `bytes` doesn't indicate what the type of the encoded data is.
This EEP proposes a new `eosio_tagged_data` struct which pairs the raw data with a type.
This type references contract ABIs to enable customization.

## Motivation
<!--The motivation is critical for EEPs that want to change the EOSIO protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the eep solves. EEP submissions without sufficient motivation may be rejected outright.-->

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current EOSIO platforms.-->

### Definition

```c++
struct eosio_tagged_data {
    name            eosio_tag_abi;
    name            eosio_tag_type;
    bytes           eosio_tag_raw;
};
```

The type of the tagged data is defined by the type `eosio_tagged_data_*`, where `*` is the
name in `eosio_tag_type`. This type is defined in the ABI belonging to the `eosio_tag_abi`
account. If `eosio_tag_type` contains any periods ('.'), then these are replaced by
underscores ('_').

Type names which begin with `e.` are reserved.

### JSON

`eosio_tagged_data` has 2 alternative JSON representations. ABI serializers use the first form (raw)
when they don't have enough information to produce the second form. The second form may be more
convenient for applications and users. The `eosio_tag_` prefix helps tools find `eosio_tagged_data`
embedded in JSON so they can transform the content.

```json
{
    "eosio_tag_abi":    "abi.account",
    "eosio_tag_type":   "type.name",
    "eosio_tag_raw":    "data in hex form"
}
```

```json
{
    "eosio_tag_abi":    "abi.account",
    "eosio_tag_type":   "type.name",
    "eosio_tag_json":   data in JSON form
}
```

## Copyright
