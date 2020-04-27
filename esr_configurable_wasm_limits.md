# Configurable WASM limits

## Simple Summary

This ESR defines a set of intrinsics to allow limits on wasm to be configurable.
It also changes the default constraints to be more consistent.

## Intrinsics

```c++
uint32_t get_wasm_parameters_packed( void* buffer, uint32_t buffer_size, uint32_t max_version);
void set_wasm_parameters_packed( const void* buffer, uint32_t buffer_size);
```

These intrinsics may be used by a privileged contract only.  `max_version` has
no effect, but should be 0.  Future protocol features should preserve the current
behavior of `get_wasm_parameters_packed` when `max_version == 0`.

## Fields

### version

Must be 0.

### max_mutable_global_bytes

The maximum total size (in bytes) used for mutable globals.  i32 and f32 consume 4 bytes and
i64 and f64 consume 8 bytes.  const globals are not included in this count.
The default is 1024.

### max_table_elements

The maximum number of elements of a table.  The default is 1024.

### max_section_elements

The maximum number of elements in each section.  The default is 8192.  The minimum value is 4.

### max_linear_memory_init

The size (in bytes) of the range of memory that may be initialized.  Data segments may use the range [0, max_linear_memory_init).
The default is 64 KiB.

### max_func_local_bytes

The maximum total size (in bytes) used by parameters and local variables in a function.
The default is 8 KiB.  The minimum value is 8.

### max_nested_structures

The maximum nesting depth of structured control instructions.  The function itself
is included in this count.  The default is 1024.  The minimum value is 1.

### max_symbol_bytes

The maximum size (in bytes) of names used for import and export.  The default is 8192.  The minimum value is 32.

### max_module_bytes

The maximum total size (in bytes) of a wasm module.  The default is 20 MiB.  The minimum value is 256.

### max_code_bytes

The maximum size (in bytes) of each function body.  The default is 20 MiB.  The minimum value is 32.

### max_pages

The maximum number of 64 KiB pages of linear memory that a contract can use.
Enforced when an action is executed.  The initial size of linear memory is
also checked at setcode.  The default is 528, which corresponds to 33 MiB.
Increasing this may cause degraded performance.  The minimum value is 1.

### max_call_depth

The maximum number of functions that may be on the stack.  Enforced when an action is executed.
The default is 251.  The minimum value is 2.

## Layout

All fields are 32-bit unsigned little endian integers.  Limits which are
greater than those imposed by the WASM specification or other limits
will have no effect.

[Example: Setting `max_pages` > 65536 does not permit a contract to use more than 4 GiB of linear memory]
[Example: max_module_bytes = 100 KiB, max_code_bytes = 20 MiB is permitted]

```
+-------+---------------------------+---------------------------+
| byte  |   0  |   1  |   2  |   3  |   4  |   5  |   6  |   7  |
+-------+---------------------------+---------------------------+
| field |          version          | max_mutable_global_bytes  |
+-------+---------------------------+---------------------------+
| byte  |   8  |   9  |  10  |  11  |  12  |  13  |  14  |  15  |
+-------+---------------------------+---------------------------+
| field |     max_table_elements    |   max_section_elements    |
+-------+---------------------------+---------------------------+
| byte  |  16  |  17  |  18  |  19  |  20  |  21  |  22  |  23  |
+-------+---------------------------+---------------------------+
| field |   max_linear_memory_init  |   max_func_local_bytes    |
+-------+---------------------------+---------------------------+
| byte  |  24  |  25  |  26  |  27  |  28  |  29  |  30  |  31  |
+-------+---------------------------+---------------------------+
| field |   max_nested_structures   |     max_symbol_bytes      |
+-------+---------------------------+---------------------------+
| byte  |  32  |  33  |  34  |  35  |  36  |  37  |  38  |  39  |
+-------+---------------------------+---------------------------+
| field |      max_code_bytes       |     max_module_bytes      |
+-------+---------------------------+---------------------------+
| byte  |  40  |  41  |  42  |  43  |  44  |  45  |  46  |  47  |
+-------+---------------------------+---------------------------+
| field |         max_pages         |       max_call_depth      |
+-------+---------------------------+---------------------------+
```
