# Choose a bounded Core cache

Read this before implementing a cache with FIFO or least-recently-used eviction. Paths below are relative to Core.

Use `el::FifoCache<Key, Value>` from `<erbsland/FifoCache.hpp>` or `el::LruCache<Key, Value>` from
`<erbsland/LruCache.hpp>`. Both belong to the `util` source domain and use flattened consumer names.

## Choose the eviction policy

| Behavior | `FifoCache` | `LruCache` |
| --- | --- | --- |
| Entry evicted when full | Oldest insertion | Least recently used entry |
| Successful `get(key)` | Preserves insertion order | Refreshes recency; requires a mutable cache |
| `set(key, value)` for an existing key | Replaces the value without moving its position | Replaces the value and refreshes recency |
| `peek(key)` or `contains(key)` | Preserves order | Preserves order |

Choose FIFO when insertion age determines eviction; choose LRU when repeated reads should keep entries resident.
Use `get` directly when retrieving an LRU value, rather than testing `contains` and then calling `peek`:
those inspection methods deliberately do not mark the entry as recently used.

## Capacity and ownership

- Construct with a finite `el::ItemCount`, such as `el::ItemCount{128}`. `capacity()` and `count()` return
  `ItemCount`; the limit counts entries, not bytes. For runtime integers, follow [unit construction](unit-types.md).
- `setCapacity(...)` immediately evicts excess entries in policy order. Zero clears the cache and disables retention
  until the capacity increases. `clear()` retains the capacity; `remove(key)` ignores absent keys.
- `get(key)` and `peek(key)` return `std::optional<Value>` by value. Check for absence before dereferencing.
  Returned values survive removal or eviction; they are copies, not references to entries.
- Keys and values must be copyable. The optional third and fourth template parameters select the default-constructible
  hash and equality types; defaults are `std::hash<Key>` and `std::equal_to<Key>`.
- Cache instances own independent storage. Copying copies keys and values; sharing one cache between threads requires
  external synchronization, including for LRU reads that update recency.

Retain the cache with the state whose computations it reuses. Creating it inside every lookup discards the retained
entries. Use it for bounded key/value reuse; regex convenience caching is already provided by
[the cached regex helpers](string-matching.md#cached-literal-expressions-for-simple-script-like-code).

## Example: reading an entry keeps it resident in an LRU cache

With `<erbsland/LruCache.hpp>`, `<erbsland/ItemCount.hpp>`, and `<erbsland/String.hpp>`, in a function body with
`using namespace el::text::literals;` supplied by the enclosing application namespace:

```cpp
auto labels = el::LruCache<int, el::String>{el::ItemCount{2}};
labels.set(1, "north"_el).set(2, "south"_el);
const auto north = labels.get(1);
labels.set(3, "west"_el);
```

The cache retains keys 1 and 3 because `get(1)` refreshed key 1's recency. `north` is an optional copy of `"north"_el`.
With `FifoCache` and its corresponding include, the same sequence evicts key 1 and retains keys 2 and 3 instead;
the returned `north` value remains valid in either case.

Sources: `src/erbsland/util/{FifoCache,LruCache}.hpp` and the bounded-caches section of
`doc/reference/util/utilities.rst`. Behavioral examples are in `test/unittest/src/util/{FifoCache,LruCache}Test.cpp`.
