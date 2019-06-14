# ABI 1.2: Sized Data in ABIs

## Simple Summary

This proposal gives ABIs a way to describe sized data.

## Abstract

ABI 1.1 supports storing size-prefixed data in the `string` and `bytes` types, but doesn't provide a way to describe the contents
of those types. Likewise, it supports a set of fixed-size types suitable for storing content (`checksum160`, `checksum256`,
and `checksum512`), but doesn't provide a way to describe content or a way to choose other sizes. This proposal adds a way to
describe the content of sized storage, and to specify arbitrary fixed sizes.

## Specification

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

ABI serializers which don't support 1.2 will error out when they encounter `#` in type names. Either they
won't recognize the `#`, or they'll think it's part of a type name which isn't defined.

It's possible that some contract ABIs included `#` in their type names. ABI 1.2 serializers will malfunction with these.

## Copyright
