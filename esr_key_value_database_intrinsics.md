# Key-Value Store Intrinsics

## Simple Summary

This ESR defines a potential set of intrinsics to implement part of [Key-Value Database](esr_key_value_database.md).
It doesn't cover CDT enhancements.

## Database ID

The `db` argument indicates which database to operate on. There are initially 2 values for this:

* `"eosio.kvram"`: This database has low latency and consumes `RAM` resources.
* `"eosio.kvdisk"`: This database consumes a new `DISK` resource. It may have a higher capacity, but may also consume higher
       CPU to access than database `"eosio.kvram"`.

All intrinsics which have a `db` argument assert if it's out of range.

## Billing

For each entry in a contract's table, the kv database bills 112 bytes + the size of the key + the size
of the value to the contract.

## Temporary Data Buffer

Some intrinsics use a temporary data buffer. `kv_get_data` reads this buffer. The buffer is initially empty.

## Ordering

Keys are ordered lexicographically by `uint8_t`.

## Pointer Aliasing

Pointers are not allowed to alias for any kv intrinsic.
If two pointer arguments in a call to an intrinsic, refer to two memory ranges,
M<sub>1</sub> = [B<sub>1</sub>, E<sub>1</sub>) and M<sub>2</sub> = [B<sub>2</sub>, E<sub>2</sub>),
such that M<sub>1</sub> ∩ M<sub>2</sub> ≠ ∅, the transaction will be aborted.

## Non-Iterator Intrinsics

```c++
int64_t  kv_erase(uint64_t db, uint64_t contract, const char* key, uint32_t key_size);
int64_t  kv_set(uint64_t db, uint64_t contract, const char* key, uint32_t key_size, const char* value, uint32_t value_size);
bool     kv_get(uint64_t db, uint64_t contract, const char* key, uint32_t key_size, uint32_t* value_size);
uint32_t kv_get_data(uint64_t db, uint32_t offset, char* data, uint32_t data_size);
```

These intrinsics modify the database and support point lookups. Point lookups may use less CPU than iterator operations.

### kv_erase

`kv_erase` erases a key. If `contract` doesn't match the executing contract then it aborts the transaction. Else the
function succeeds, even if the key didn't exist.

If any iterators are positioned at the erased key, then `kv_erase` switches their status to `iterator_erased`.

`kv_erase` clears the temporary data buffer.

Returns the change in resource billing caused by the erase.  If the key existed,
this will always be negative.

### kv_set

`kv_set` sets a key-value pair. If `contract` doesn't match the executing contract then it aborts the transaction.
If the contract exceeds available resources, it will fail at the appropriate time, similar to existing database
primitives.

`kv_set` clears the temporary data buffer.

Returns the change in resource usage.

Consensus parameters limit the maximum key size and the maximum value size. `kv_set` aborts the transaction if
the contract exceeds these limits.

### kv_get

`kv_get` checks the existence of a key. If the key doesn't exist, it returns `false`, clears the temporary data buffer,
and sets `*value_size` to 0. If the key does exist, it returns `true`, stores the value into the temporary data buffer,
and sets `*value_size` to the value size. Use `kv_get_data` to retrieve the value.

If the contract doesn't exist, then this behaves as if the key doesn't exist.

### kv_get_data

```c++
uint32_t kv_get_data(uint64_t db, uint32_t offset, char* data, uint32_t data_size);
```

Fetches data from temporary buffer starting at `offset`. Copies up to `data_size` bytes into `data`.
Returns amount of data in temporary buffer.

`kv_get_data` doesn't modify `data` if `offset` is greater than the temporary buffer size.
It still aborts the transaction if `[data, data + data_size)` is out of the WASM's linear
memory range.

## Iterator Intrinsics

```c++
uint32_t kv_it_create(uint64_t db, uint64_t contract, const char* prefix, uint32_t size);
void     kv_it_destroy(uint32_t itr);
it_stat  kv_it_status(uint32_t itr);
int      kv_it_compare(uint32_t itr_a, uint32_t itr_b);
int      kv_it_key_compare(uint32_t itr, const char* key, uint32_t size);
it_stat  kv_it_move_to_end(uint32_t itr);
it_stat  kv_it_next(uint32_t itr, uint32_t* found_key_size, uint32_t* found_value_size);
it_stat  kv_it_prev(uint32_t itr, uint32_t* found_key_size, uint32_t* found_value_size);
it_stat  kv_it_lower_bound(uint32_t itr, const char* key, uint32_t size, uint32_t* found_key_size, uint32_t* found_value_size);
it_stat  kv_it_key(uint32_t itr, uint32_t offset, char* dest, uint32_t size, uint32_t* actual_size);
it_stat  kv_it_value(uint32_t itr, uint32_t offset, char* dest, uint32_t size, uint32_t* actual_size);
```

These intrinsics support iterating over ranges of key-value pairs. An iterator, during its lifetime:

* Stays within a single database ID
* Stays within a single contract's data
* Stays within the subset of keys that start with the prefix passed to `kv_it_create`

An iterator has a status (`it_stat`, 32 bits), which is one of the following:

| Name             | Value | Description |
|------------------|-------|-------------|
|`iterator_ok`     | 0     | Iterator is positioned at a key-value pair |
|`iterator_erased` | -1    | The key-value pair that the iterator used to be positioned at was erased |
|`iterator_end`    | -2    | Iterator is out-of-bounds |

### kv_it_create

```c++
uint32_t kv_it_create(uint64_t db, uint64_t contract, const char* prefix, uint32_t size);
```

`kv_it_create` creates an iterator. The returned handle:

* Starts at 1.
* Counts up by 1 at each call, assuming no destroyed iterators are available.
* If destroyed iterators are available, the most-recently-destroyed one is reinitialized and returned.

The prefix limits the range of keys that the iterator covers. If the prefix is empty, the iterator
covers the entire range of keys belonging to a contract within the database ID.

If the contract doesn't exist, then the iterator covers an empty range.

The newly-created iterator has `iterator_end` status.

A consensus parameter limits the number of available iterators for each database.
`kv_it_create` aborts the transaction if the contract exceeds this or if the iterator
handle is not representable as a uint32_t.

The size of the prefix is limited by the same consensus parameter that limits
the key size.  `kv_it_create` aborts the transaction if size exceeds this.

### kv_it_destroy

```c++
void     kv_it_destroy(uint32_t itr);
```

This destroys an iterator. It aborts the transaction if `itr` wasn't returned by a call to
`kv_it_create`, or if `itr` has already been destroyed.

### kv_it_status

```c++
it_stat  kv_it_status(uint32_t itr);
```

This returns the status of an iterator. It aborts the transaction if `itr` wasn't returned
by a call to `kv_it_create`, or if `itr` has been destroyed.

`kv_it_status` returns `iterator_ok`, `iterator_erased`, or `iterator_end`.

### kv_it_compare

```c++
int      kv_it_compare(uint32_t itr_a, uint32_t itr_b);
```

Compare the position of two iterators. It aborts the transaction if either iterator
wasn't returned by `kv_it_create`, was destroyed, or has status `iterator_erased`.
It also aborts if the iterators are from different databases or from different contracts.

| Return Value | Description |
|--------------|-------------|
| -1           | `itr_a`'s key is less than `itr_b`'s key |
| 0            | `itr_a`'s key is the same as `itr_b`'s key |
| 1            | `itr_a`'s key is greater than `itr_b`'s key |

If an iterator has status `iterator_end`, then it compares greater than iterators with status
`iterator_ok`. Two iterators with status `iterator_end` compare equal (0 return value).

### kv_it_key_compare

```c++
int      kv_it_key_compare(uint32_t itr, const char* key, uint32_t size);
```

Compare the position of an iterator to a provided key. It aborts the
transaction if `itr` wasn't returned by `kv_it_create`, was destroyed,
or has status `iterator_erased`.

| Return Value | Description |
|--------------|-------------|
| -1           | `itr`'s key is less than `key` |
| 0            | `itr`'s key is the same as `key` |
| 1            | `itr`'s key is greater than `key` |

If `itr` has status `iterator_end`, then it compares greater than `key`, no matter
what value `key` has.

### kv_it_move_to_end

```c++
it_stat  kv_it_move_to_end(uint32_t itr);
```

Move `itr` to out-of-bounds and return the new status (`iterator_end`). It aborts the
transaction if `itr` wasn't returned by `kv_it_create` or was destroyed.

### kv_it_next

```c++
it_stat  kv_it_next(uint32_t itr, uint32_t* found_key_size, uint32_t* found_value_size);
```

Move iterator to next position and return its new status. It aborts the
transaction if `itr` wasn't returned by `kv_it_create`, was destroyed,
or has status `iterator_erased`.

If `itr`'s status is `iterator_ok`, then this finds the
next non-deleted key in range. If found, the new status is `iterator_ok`. If
not found, the new status is `iterator_end`.

If `itr`'s status is `iterator_end` then this finds the first non-deleted key
in range. If found, the new status is `iterator_ok`. If not found, the new
status is `iterator_end`.

The size of the key and the size of the value at the iterator's new
position, or zero if the iterator is at end, will be written to
found_key_size and found_value_size.

`kv_it_next` never returns `iterator_erased`.

### kv_it_prev

```c++
it_stat  kv_it_prev(uint32_t itr, uint32_t* found_key_size, uint32_t* found_value_size);
```

Move iterator to previous position and return its new status. It aborts the
transaction if `itr` wasn't returned by `kv_it_create`, was destroyed,
or has status `iterator_erased`.

If `itr`'s status is `iterator_ok`, then this finds the previous non-deleted
key in range. If found, the new status is `iterator_ok`. If not found, the
new status is `iterator_end`.

If `itr`'s status is `iterator_end` then this finds the last non-deleted key
in range. If found, the new status is `iterator_ok`. If not found, the new
status is `iterator_end`.

The size of the key and the size of the value at the iterator's new
position, or zero if the iterator is at end, will be written to
found_key_size and found_value_size.

`kv_it_prev` never returns `iterator_erased`.

### kv_it_lower_bound

```c++
it_stat  kv_it_lower_bound(uint32_t itr, const char* key, uint32_t size);
```

Find the least non-deleted key which is >= the provided key and has the correct prefix.
Returns the new iterator status. It aborts the transaction if `itr` wasn't returned by
`kv_it_create` or was destroyed.

If a key is found, the new status is `iterator_ok`. If not found, the new status is
`iterator_end`. `kv_it_lower_bound` never returns `iterator_erased`.

The size of the key and the size of the value at the iterator's new
position, or zero if the iterator is at end, will be written to
found_key_size and found_value_size.

### kv_it_key

```c++
it_stat  kv_it_key(uint32_t itr, uint32_t offset, char* dest, uint32_t size, uint32_t* actual_size);
```

Fetch the key from the iterator and return the iterator's status. It aborts the transaction if
`itr` wasn't returned by `kv_it_create`, was destroyed, or has status `iterator_erased`.

This sets `*actual_size` to the size of the key and copies up to `size` bytes into `dest`.
If `itr`'s status is `iterator_end`, then this function behaves as if the key is empty.

`kv_it_key` doesn't modify `dest` if `offset` is greater than the iterator's key size.
It still aborts the transaction if `[dest, dest + size)` is out of the WASM's linear
memory range.

### kv_it_value

```c++
it_stat  kv_it_value(uint32_t itr, uint32_t offset, char* dest, uint32_t size, uint32_t* actual_size);
```

Fetch the value from the iterator and return the iterator's status. It aborts the transaction if
`itr` wasn't returned by `kv_it_create`, was destroyed, or has status `iterator_erased`.

This sets `*actual_size` to the size of the value and copies up to `size` bytes into `dest`.

If `itr`'s status is `iterator_end`, then this function behaves as if the value is empty.

`kv_it_value` doesn't modify `dest` if `offset` is greater than the iterator's value size.
It still aborts the transaction if `[dest, dest + size)` is out of the WASM's linear
memory range.


## Privileged intrinsics

```c++
int64_t  get_resource_limit( name account, name resource );
void     set_resource_limit( name account, name resource, int64_t limit );
uint32_t get_kv_parameters_packed( name db, void * packed_parameters, uint32_t buffer_size );
void     set_kv_parameters_packed( name db, const void * packed_parameters, uint32_t buffer_size );
```

These intrinsics control per-account resource limits and kv specific limits.
Attempting to call them from a non-privileged contract aborts the transaction.

### set_resource_limit, get_resource_limit

```c++
void set_resource_limit( name account, name resource, int64_t limit );
int64_t get_resource_limit( name account, name resource );
```

Replacements for `get_resource_limits` and `set_resource_limits`.  Sets or gets the current
limit of the named resource.  A limit of -1 indicates that the account's use of the
resource is not constrained.  Valid resources are currently `"ram"`, `"disk"`, `"net"`, `"cpu"`.
If the account does not exist aborts the transaction.  Attempting to set the limit to a
negative value other that -1 aborts the transaction.

The default limit for `"ram"`, `"cpu"`, and `"net"` for new accounts is -1.
The default for `"disk"` is 0.

### get_kv_parameters_packed

```c++
uint32_t get_kv_parameters_packed( name db, void * parameters, uint32_t buffer_size, uint32_t max_version );
```

Gets the maximum key size, maximum value size, and maximum iterators of a kv database and returns the
size of the data.  If `buffer_size` is too small to fit the parameters, returns the size but does not
write to the buffer.  `max_version` has no effect, but should be 0.  Other values are reserved for
future extensions.

The kv parameters are encoded as 16 bytes, representing four 32-bit little-endian values.

```
+-------+---------------+---------------+---------------+---------------+
| byte  | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |10 |11 |12 |13 |14 |15 |
+-------+---------------+---------------+---------------+---------------+
| field |   version     |   key limit   |  value limit  | max iterators |
+-------+---------------+---------------+---------------+---------------+
| type  |      0        |   32-bits LE  |  32-bits LE   |  32-bits LE   |
+-------+---------------+---------------+---------------+---------------+
```

### set_kv_parameters_packed

```c++
void set_kv_parameters_packed( name db, const void * parameters, uint32_t buffer_size );
```

Sets the maximum key size, and maximum value size, and maximum iterators of a kv database.  Each database has
independent limits.  The key and value limits only apply to new items.  They do not apply to
items written before they were applied.  If the database is invalid, if version is non-zero, or
if `buffer_size` is less than 16, aborts the transaction.

The default limits on a new chain are 0KiB for the key, 0KiB for the value, and 0 iterators.
After activating the protocol feature, the limits should be set to sensible values.
