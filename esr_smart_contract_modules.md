 # Smart Contract Module Extensions

## Simple Summary

As of EOSIO.CDT v1.7.0, smart contract developers must specify the smart contract "module" in a few ways.
The main way of specifying this promotes all declarations in one monolithic C++ class and penalizes 
good code structuring.  These extensions aim to reduce the redundancy of specifying these "modules".

## Abstract
Smart contract developers must specify the smart contract "module" in a few ways.
```c++
class [[eosio::contract]] test : public eosio::contract { ... };
```
This will automatically use the name of the class as the smart contract "contract module".
Or,
```c++
class [[eosio::contract("overriden_name")]] test : public eosio::contract { ... };
```
which will override the name to what the developer specifies.  This tells the compiler and linker which
`actions` and `tables` are to be associated with the smart contract in question.  This allows the ABI
generator, code generator, compiler warnings, etc. to know what to filter out and allows users to 
include header files like `eosio.token.hpp` and not inherit those actions or tables into their ABI/code.
This has an issue, in that it promotes an anti-pattern for C++ developers to place all of their code into
one monolithic class.  To counter-act that the `[[eosio::contract]]` attribute can be used on action declarations
and table declarations to "attach" them to a specific smart contract module.
```c++
struct [[eosio::table, eosio::contract("test")]] sometable { ... }; 
```
This can be a bit too much syntax for what should be a valid normal use case.  This ESR proposes changing the name 
of the attribute `eosio::contract` to `eosio::app`, this helps eliminate the confusion of users because of the use of `contract` in too
many contexts. In addition to this the set of EOSIO.CDT attributes will supply the attributes with and without 
the `eosio` namespacing.
To help with good structuring, the `eosio::app` and `app` attributes will be allowed to be attached to a namespace,
this would allow all enclosed actions and tables to be a part of that module.

## Specification

The attributes to be added are:
   * *`eosio::app`
   * `app`
   * `action`
   * `table`

The `eosio::contract` attribute will be deprecated.

The developer can define their smart contract "module" by tagging the namespace with the `app` or `eosio::app` attributes.
```c++
namespace [[app("test")]] test_app {

class test : public eosio::contract {
   [[action]]
   void foo(...);
   struct [[table]] taba {
      ...
   };
};

} // namespace eosio
```

This will tell the compiler and linker that any `action`s or `table`s declared in source files enclosed by the tagged namespace
should be used/included.

The same rules for attaching the attribute to classes for smart contracts can be used and overriding the contract can be used
too (if still needed).

```c++
namespace [[app("test")]] test_app {
class [[app("overridden_app")]] test : public eosio::contract {
   ...
};
}
```

This allows for backward compatibility with the existing system until the deprecated `[[eosio::contract]]` attribute is removed. 

If the same namespace is used in multiple places and is declared with multiple "app" names, then the compiler will generate an error.

```c++
namespace [[app("test")]] test_app {
}
namespace [[app("foo")]] test_app {
}
```
This will generate a compiler error for conflicting app declarations.

But,
```c++
namespace [[app("test")]] test_app {
}
namespace test_app {
}
```
is perfectly valid and will use the app name from the single use of the attribute.
