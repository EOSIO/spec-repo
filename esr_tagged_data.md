# Tagged Data

## Simple Summary

Contracts may tag serialized data with custom types to identify it for decoding.

## Abstract

Contracts may pack arbitrary data in the `bytes` type when communicating with each other
and with the outside, but `bytes` doesn't indicate what the type of the encoded data is.
This ESR proposes a new `eosio_tagged_data` struct which pairs the raw data with a type.
This type references contract ABIs to enable customization.

## Specification

### Definition

```c++
struct eosio_tagged_data {
    name            eosio_tag_abi;
    name            eosio_tag_type;
    bytes           eosio_tag_raw;
};
```

The ABI type `eosio_tagged_data_*` describes the content of the raw data, where `*` is the
name in `eosio_tag_type`. This type is in the ABI belonging to the `eosio_tag_abi`
account. If `eosio_tag_type` contains any periods ('.'), then these are replaced by
underscores ('_').

Type names which begin with `e.` are reserved.

### CDT Support

To define a type which can appear in tagged data, use the `eosio::include_in_abi` tag and
inherit from `tagged_base`:

```c++
[[eosio::contract("contract name"), eosio::include_in_abi()]]
struct transfer_data: tagged_base<"transfer.dat"_n> {
    name    from;
    name    to;
    asset   amount;
};
```

This adds a struct named `eosio_tagged_data_transfer_dat` to the contract's ABI.

`tagged_variant` aids decoding tagged data. It deserializes to the appropriate type when
there's a match and falls back to `eosio_tagged_data` when there isn't a match. When
`tagged_variant` is an action argument or a struct member, the CDT fills its type in
as `eosio_tagged_data` in the ABI.

```c++
template<typename... Ts>
struct tagged_variant {
    variant<Ts..., eosio_tagged_data> value;
};
```

Example use:

```c++
using balance_change_data = tagged_variant<
    transfer_data,
    issue_data,
    open_data,
    close_data
>;
```

### JSON

`eosio_tagged_data` has 2 alternative JSON representations. ABI serializers use the first form (raw)
when they don't have enough information to produce the second form. The second form may be more
convenient for applications and users. The `eosio_tag_` prefix helps tools find `eosio_tagged_data`
embedded in JSON so they can transform the content between the two forms.

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
