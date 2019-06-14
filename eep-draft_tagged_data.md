# Tagged Data

## Simple Summary

Contracts may tag serialized data with custom types to identify it and aid decoding.

## Abstract

Contracts may pack arbitrary data in the `bytes` type when communicating with each other
and with the outside, but `bytes` doesn't indicate what the type of the encoded data is.
This EEP proposes a new `eosio_tagged_data` struct which pairs the raw data with a type.
This type references contract ABIs to enable customization.

## Motivation

## Specification

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
