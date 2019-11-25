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

## Temporary Data Buffer

Some intrinsics use a temporary data buffer. `kv_get_data` reads this buffer. The buffer is initially empty.

## Ordering

Keys are ordered lexicographically by `uint8_t`.

## Non-Iterator Intrinsics

```c++
void     kv_erase(uint64_t db, uint64_t contract, const char* key, uint32_t key_size);
void     kv_set(uint64_t db, uint64_t contract, const char* key, uint32_t key_size, const char* value, uint32_t value_size);
bool     kv_get(uint64_t db, uint64_t contract, const char* key, uint32_t key_size, uint32_t& value_size);
uint32_t kv_get_data(uint64_t db, uint32_t offset, char* data, uint32_t data_size);
```

These intrinsics modify the database and support point lookups. Point lookups may use less CPU than iterator operations.

### kv_erase

`kv_erase` erases a key. If `contract` doesn't match the executing contract then it aborts the transaction. Else the
function succeeds, even if the key didn't exist.

If any iterators are positioned at the erased key, then `kv_erase` switches their status to `iterator_erased`.

`kv_erase` clears the temporary data buffer.

### kv_set

`kv_set` sets a key-value pair. If `contract` doesn't match the executing contract then it aborts the transaction.
If the contract exceeds available resources, it will fail at the appropriate time, similar to existing database
primitives.

If any iterators are positioned at the key, then `kv_set` switches their status to `iterator_ok`. This allows
iterators to live through `kv_erase` followed by `kv_set` unharmed.

`kv_set` clears the temporary data buffer.

Consensus parameters limit the maximum key size and the maximum value size. `kv_set` aborts the transaction if
the contract exceeds these limits.

### kv_get

`kv_get` checks the existence of a key. If the key doesn't exist, it returns `false`, clears the temporary data buffer,
and sets `value_size` to 0. If the key does exist, it returns `true`, stores the value into the temporary data buffer,
and sets `value_size` to the value size. Use `kv_get_data` to retrieve the value.

If the contract doesn't exist, then this behaves as if the key doesn't exist.

WASM memory order: `kv_get` writes to `value_size` after it finishes reading from `key`.

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
it_stat  kv_it_next(uint32_t itr);
it_stat  kv_it_prev(uint32_t itr);
it_stat  kv_it_lower_bound(uint32_t itr, const char* key, uint32_t size);
it_stat  kv_it_key(uint32_t itr, uint32_t offset, char* dest, uint32_t size, uint32_t& actual_size);
it_stat  kv_it_value(uint32_t itr, uint32_t offset, char* dest, uint32_t size, uint32_t& actual_size);
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

A consensus parameter limits the number of available iterators. `kv_it_create` aborts the
transaction if the contract exceeds this.

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
it_stat  kv_it_next(uint32_t itr);
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

`kv_it_next` never returns `iterator_erased`.

### kv_it_prev

```c++
it_stat  kv_it_prev(uint32_t itr);
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

`kv_it_prev` never returns `iterator_erased`.

### kv_it_lower_bound

```c++
it_stat  kv_it_lower_bound(uint32_t itr, const char* key, uint32_t size);
```

Find the first non-deleted key >= the provided key and return the new iterator status.
It aborts the transaction if `itr` wasn't returned by `kv_it_create` or was destroyed.

If a key is found, the new status is `iterator_ok`. If not found, the new
status is `iterator_end`. `kv_it_lower_bound` never returns `iterator_erased`.

### kv_it_key

```c++
it_stat  kv_it_key(uint32_t itr, uint32_t offset, char* dest, uint32_t size, uint32_t& actual_size);
```

Fetch the key from the iterator and return the iterator's status. It aborts the transaction if
`itr` wasn't returned by `kv_it_create`, was destroyed, or has status `iterator_erased`.

This sets `actual_size` to the size of the key and copies up to `size` bytes into `dest`.
If `itr`'s status is `iterator_end`, then this function behaves as if the key is empty.

`kv_it_key` doesn't modify `dest` if `offset` is greater than the iterator's key size.
It still aborts the transaction if `[dest, dest + size)` is out of the WASM's linear
memory range.

WASM memory order: `kv_it_key` writes to `actual_size` after it finishes writing to `dest`.

### kv_it_value

```c++
it_stat  kv_it_value(uint32_t itr, uint32_t offset, char* dest, uint32_t size, uint32_t& actual_size);
```

Fetch the value from the iterator and return the iterator's status. It aborts the transaction if
`itr` wasn't returned by `kv_it_create`, was destroyed, or has status `iterator_erased`.

This sets `actual_size` to the size of the value and copies up to `size` bytes into `dest`.

If `itr`'s status is `iterator_end`, then this function behaves as if the value is empty.

`kv_it_value` doesn't modify `dest` if `offset` is greater than the iterator's value size.
It still aborts the transaction if `[dest, dest + size)` is out of the WASM's linear
memory range.

WASM memory order: `kv_it_value` writes to `actual_size` after it finishes writing to `dest`.
