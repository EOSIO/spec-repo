# Key-Value Store

## Simple Summary

This ESR defines a potential set of intrinsics to implement part of [Key-Value Database](esr_key_value_database.md).
It doesn't cover CDT enhancements.

## Database ID

The `db` argument indicates which database to operate on. There are initially 2 values for this:

* `0`: This database has low latency and consumes `RAM` resources.
* `1`: This database consumes a new `DISK` resource. It may have a higher capacity, but may also consume higher
       CPU to access than database `0`.

All intrinsics which have a `db` argument assert if it's out of range.

## Temporary Data Buffer

Some intrinsics use a temporary data buffer. `kv_get_data` reads this buffer. The buffer is initially empty.

## Non-Iterator Intrinsics

```c++
void     kv_erase(uint64_t db, uint64_t contract, const char* key, uint32_t key_size);
void     kv_set(uint64_t db, uint64_t contract, const char* key, uint32_t key_size, const char* value, uint32_t value_size);
bool     kv_get(uint64_t db, uint64_t contract, const char* key, uint32_t key_size, uint32_t& value_size);
uint32_t kv_get_data(uint32_t offset, char* value, uint32_t value_size);
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

## Iterator Intrinsics

```c++
uint32_t kv_it_create(uint64_t db, uint64_t contract, const char* prefix, uint32_t size);
void     kv_it_destroy(uint32_t itr);
it_stat  kv_it_status(uint32_t itr);
int      kv_it_compare(uint32_t itr_a, uint32_t itr_b);
int      kv_it_key_compare(uint32_t itr, const char* key, uint32_t size);
it_stat  kv_it_move_to_begin(uint32_t itr);
it_stat  kv_it_move_to_end(uint32_t itr);
it_stat  kv_it_lower_bound(uint32_t itr, const char* key, uint32_t size);
it_stat  kv_it_upper_bound(uint32_t itr, const char* key, uint32_t size);
it_stat  kv_it_increment(uint32_t itr);
it_stat  kv_it_decrement(uint32_t itr);
it_stat  kv_it_key(uint32_t itr, uint32_t offset, char* dest, uint32_t size, uint32_t& copied);
it_stat  kv_it_value(uint32_t itr, uint32_t offset, char* dest, uint32_t size, uint32_t& copied);
```

These intrinsics support iterating over ranges of key-value pairs. An iterator, during its lifetime:

* Stays within a single database ID
* Stays within a single contract's data
* Stays within the subset of keys defined by the prefix passed to `kv_it_create`

An iterator has a status (`it_stat`), which is one of the following:

| Name           | Value | Description |
|----------------|-------|-------------|
|iterator_ok     | 0     | Iterator is positioned at a key-value pair |
|iterator_erased | -1    | The key-value pair that the iterator used to be positioned at was erased |
|iterator_oob    | -2    | Iterator is out-of-bounds |

### kv_it_create

```c++
uint32_t kv_it_create(uint64_t db, uint64_t contract, const char* prefix, uint32_t size);
```

`kv_it_create` creates an iterator. The returned handle:

* Starts at 1.
* Counts up by 1 at each call, assuming no destroyed iterators are available.
* If destroyed iterators are available, the lowest-numbered one is reinitialized and returned.

The prefix limits the range of keys that the iterator covers. If the prefix is empty, the iterator
covers the entire range of keys belonging to a contract within the database ID.

The newly-created iterator has `iterator_oob` status.

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

### kv_it_compare

```c++
int      kv_it_compare(uint32_t itr_a, uint32_t itr_b);
```

Compare the position of two iterators. It aborts the transaction if either iterator wasn't
returned by `kv_it_create` or was destroyed. It also aborts if the iterators are from different
databases or from different contracts.

| Return Value | Description |
|--------------|-------------|
| -1           | `itr_a`'s key is less than `itr_b`'s key |
| 0            | `itr_a`'s key is the same as `itr_b`'s key |
| 1            | `itr_a`'s key is greater than `itr_b`'s key |

If an iterator has status `iterator_erased`, then `kv_it_compare` uses the value of the erased key
during comparison.

If an iterator has status `iterator_oob`, then it compares greater than iterators with status
`iterator_ok` or `iterator_erased`. Two iterators with status `iterator_oob` compare equal
(0 return value).

### kv_it_key_compare

```c++
int      kv_it_key_compare(uint32_t itr, const char* key, uint32_t size);
```

Compare the position of an iterator to a provided key. It aborts the transaction if `itr` wasn't
returned by `kv_it_create` or was destroyed.

| Return Value | Description |
|--------------|-------------|
| -1           | `itr`'s key is less than `key` |
| 0            | `itr`'s key is the same as `key` |
| 1            | `itr`'s key is greater than `key` |

If `itr` `iterator_erased`, then `kv_it_key_compare` uses the value of the erased key
during comparison. If `itr` has status `iterator_oob`, then it compares greater than 
`key`, no matter what value `key` has.
