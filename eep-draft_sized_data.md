# ABI 1.2: Sized Data in ABIs

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the EEP.-->

This proposal gives ABIs a way to describe sized data.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

ABI 1.1 supports storing size-prefixed data in the `string` and `bytes` types, but doesn't provide a way to describe the contents
of those types. Likewise, it supports a set of fixed-size types suitable for storing content (`checksum160`, `checksum256`,
and `checksum512`), but doesn't provide a way to describe content or a way to choose other sizes. This proposal adds a way to
describe the content of sized storage, and to specify arbitrary fixed sizes.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current EOSIO platforms.-->

### ABI

The new `#` type suffix describes sized data. It has two forms: `#` and `#n`, where `n` is an integer greater than 0.
Type `foo#` has a varuint size prefix followed by the bytes of `foo`. Type `foo#32` has the bytes of `foo` 0-padded
to 32 bytes.

### CDT

The CDT defines two new wrappers for sized data:

```c++
template<typename T>
struct sized_data {
    T value{};
};

template<typename T, int Size>
struct fixed_sized_data {
    T value{};
};
```

`sized_data` causes the CDT to serialize `value` as if the ABI type had a `#` suffix. It adds the suffix to the ABI.
Likewise, `fixed_sized_data` causes the CDT to serialize `value` as if the ABI type had a `#n` suffix, where `n=Size`.
It adds this suffix to the ABI.

## Backwards Compatibility
<!--All EEPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The EEP must explain how the author proposes to deal with these incompatibilities. EEP submissions without a sufficient backwards compatibility treatise may be rejected outright.-->

ABI serializers which don't support 1.2 will error out when they encounter `#` in type names. Either they
won't recognize the `#`, or they'll think it's part of a type name which isn't defined.

It's possible that some contract ABIs included `#` in their type names. ABI 1.2 serializers will malfunction with these.

## Copyright
